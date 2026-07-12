# Diagrama de arquitectura — redSegura

> Diagramas en [Mermaid](https://mermaid.js.org/) (se renderizan en GitHub/GitLab).
> Contexto y decisiones: [`memoria_tecnica_global.md`](memoria_tecnica_global.md).

**Última actualización:** 2026-07-11 · **Estado:** en desarrollo (Fundación)

---

## 1. Vista de capas / componentes

```mermaid
flowchart TB
    subgraph client["Cliente"]
        UI["Angular SPA — Dashboard<br/>(tiempo real: WebSocket/SSE)"]
    end

    GW["API Gateway (Spring Cloud Gateway)<br/>AuthN/AuthZ · Rate limit · Agregacion"]

    subgraph infra["Infraestructura compartida"]
        KC["Keycloak<br/>(OAuth2/OIDC)"]
        SD["Service Discovery<br/>(Eureka/Consul)"]
        CS["Config Server<br/>(Spring Cloud Config)"]
    end

    subgraph faseA["Microservicios — Fase A"]
        AIS["asset-inventory (Java)"]
        CBS["config-backup (Python)"]
        CAS["compliance-audit (Python)"]
        ALS["alerting (Java)"]
        NOS["notification (Java)"]
    end

    subgraph faseB["Microservicios — Fase B"]
        SOS["scan-orchestrator (Python)"]
        VUS["vulnerability (Java)"]
        RTS["remediation-tracking (Java)"]
        RES["reporting (Python)"]
        TCS["telemetry-collector (Python)"]
    end

    BROKER{{"RabbitMQ<br/>exchange: redsegura.events"}}
    DBS[("PostgreSQL x10<br/>database-per-service<br/>(+ TimescaleDB para telemetria)")]
    OBS["Observabilidad<br/>Prometheus/Grafana · Loki · Jaeger"]

    UI -->|HTTPS / WSS| GW
    GW --> KC
    GW --> SD
    GW --> faseA
    GW --> faseB
    faseA <--> BROKER
    faseB <--> BROKER
    faseA --> DBS
    faseB --> DBS
    CS -.config.-> faseA
    CS -.config.-> faseB
    faseA -.metricas/logs/trazas.-> OBS
    faseB -.metricas/logs/trazas.-> OBS
```

---

## 2. Flujo de eventos (RabbitMQ pub/sub — Fase A)

```mermaid
flowchart LR
    AIS["asset-inventory"] -->|asset.created / updated / decommissioned| X{{"topic exchange<br/>redsegura.events"}}
    CBS["config-backup"] -->|config.backup_completed / failed| X
    CBS -->|config.drift_detected| X
    CBS -->|config.unsaved_changes_detected| X
    CAS["compliance-audit"] -->|compliance.finding_created| X
    ALS["alerting"] -->|alert.created| X

    X -->|asset.*| CBS
    X -->|asset.*| CAS
    X -->|config.* + compliance.finding_created| ALS
    X -->|alert.created| NOS["notification"]
    X -.no procesable.-> DLQ[("dead-letter queue")]
```

---

## 3. Flujo clave: golden path de Fase A

Alta de un dispositivo → respaldo → auditoría → alerta → notificación/dashboard.

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant GW as API Gateway
    participant AIS as asset-inventory
    participant CBS as config-backup
    participant CAS as compliance-audit
    participant MQ as RabbitMQ
    participant ALS as alerting
    participant NOS as notification
    participant UI as Dashboard

    Admin->>GW: POST /devices (alta dispositivo)
    GW->>AIS: crear dispositivo (RBAC: Administrador)
    AIS-->>MQ: asset.created

    Admin->>GW: POST /devices/{id}/backups
    GW->>CBS: respaldar via SSH (running + startup)
    CBS-->>MQ: config.backup_completed
    Note over CBS: si running != startup<br/>publica config.unsaved_changes_detected

    Admin->>GW: POST /audits
    GW->>CAS: auditar (Playbooks Ansible)
    CAS-->>MQ: compliance.finding_created

    MQ-->>ALS: compliance.finding_created
    ALS->>ALS: evalua regla configurable
    ALS-->>MQ: alert.created
    MQ-->>NOS: alert.created
    NOS-->>Admin: notificacion (email/Telegram)
    ALS-->>UI: alerta en tiempo real (WebSocket)
```

---

## 4. Estructura interna de un microservicio (patrón por capa)

```mermaid
flowchart TB
    subgraph java["Servicio Java / Spring Boot"]
        JC["Controller (REST)"] --> JSEC["Filtro JWT + autorizacion por rol"]
        JSEC --> JS["Service (@Transactional)"]
        JS --> JR["Repository (JPA)"]
        JS --> JP["Event Publisher"]
    end
    subgraph py["Servicio Python / FastAPI"]
        PC["Router"] --> PSEC["Dependency Auth (JWT)"]
        PSEC --> PS["Service"]
        PS --> PR["Repository"]
        PS --> PP["Event Publisher"]
    end
    JR --> DBJ[("PostgreSQL propia")]
    PR --> DBP[("PostgreSQL propia")]
    JP --> MQ{{"RabbitMQ"}}
    PP --> MQ
```

> Regla transversal: DTO ↔ entidad (nunca exponer entidades), excepciones tipadas de negocio,
> manejo global de errores sin filtrar internos, e inyección por constructor / DI nativa.

---

## 5. Máquina de estados: ticket de remediación (Fase B)

```mermaid
stateDiagram-v2
    [*] --> Abierto
    Abierto --> EnProgreso: asignar / iniciar
    EnProgreso --> Resuelto: marcar resuelto
    Resuelto --> Verificado: verificar OK
    Resuelto --> EnProgreso: verificacion rechazada
    Verificado --> [*]
    note right of Verificado
        Transiciones invalidas rechazadas (RF-27)
        Historial auditable por ticket (RF-28)
    end note
```

---

## 6. Vista de despliegue (evolución Docker Compose → Kubernetes)

```mermaid
flowchart LR
    D1["Docker Compose<br/>(local, MacBook)"] --> D2["k3d / minikube<br/>(Kubernetes local)"]
    D2 --> D3["k3s en EC2<br/>(AWS, sin costo de control plane)"]
    D3 --> D4["Amazon EKS<br/>(ventana corta, demo)"]
```

> Mismo código en todos los entornos; solo cambia la configuración (12-factor, RNF-22).
> Infraestructura definida como código con Terraform (RNF-23); entorno AWS "apagado por
> defecto" por presupuesto (RNF-24).
