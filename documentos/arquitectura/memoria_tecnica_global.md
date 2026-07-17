# Memoria técnica global — redSegura

> Documento **maestro** del sistema: fuente primaria de contexto arquitectónico. Leer al
> iniciar trabajo en cualquier microservicio; actualizar al cerrar cada módulo si hay
> decisiones transversales nuevas. Conciso: **decisiones y contratos, no tutoriales**.
> Requerimientos (RF/RNF) y matriz RBAC completa: `../proyecto_microservicios_redsegura.md`.

**Versión:** 0.6.2 · **Última actualización:** 2026-07-14 · **Estado:** en desarrollo (Fase A —
`asset-inventory-service` implementado y **QA R1 certificada**, incl. RF-05a dual-stack IPv4/IPv6;
CI activo)

---

## 1. Visión y alcance

- **Problema que resuelve:** en redes medianas, el respaldo de configuraciones, la
  verificación de cumplimiento y la detección de vulnerabilidades se hacen manualmente o con
  herramientas desconectadas, sin correlación entre "qué tengo" (inventario) y "qué tan
  seguro/cumplido está". redSegura unifica **inventario + postura de configuración + postura
  de vulnerabilidades** sobre una sola fuente de verdad.
- **Usuarios / actores:** **Administrador** (gestión total), **Operador** (operación diaria),
  **Auditor / Solo lectura** (revisión sin modificar). Autenticación federada vía Keycloak.
- **Alcance — dentro:** inventario de activos, respaldo y versionado de configuraciones,
  auditoría de cumplimiento, orquestación de escaneos, gestión de vulnerabilidades (CVE/CVSS),
  seguimiento de remediación, alertas, notificaciones, reportes ejecutivos, telemetría, y un
  dashboard Angular. Despliegue en Docker Compose → Kubernetes (k3s/EKS) sobre AWS.
- **Alcance — fuera:** service mesh, multi-región, blue-green/canary avanzado, operadores K8s
  propios, mTLS entre servicios (mejora futura), multi-tenant, y **cualquier escaneo contra
  redes de terceros sin autorización explícita** (control técnico, no solo política — RNF-07).
- **Tipo de sistema:** plataforma de **microservicios poliglota** (Java/Spring Boot + Python/
  FastAPI) con persistencia PostgreSQL *database-per-service*, mensajería RabbitMQ y SPA Angular.
- **Restricción ético/legal (transversal):** durante desarrollo, pruebas y despliegue de este
  proyecto, todo escaneo/automatización apunta **exclusivamente** a una red simulada
  (GNS3/Packet Tracer). La capacidad de operar sobre redes reales existe (RF-16), pero solo se
  usaría con autorización explícita del propietario.

---

## 2. Decisiones arquitectónicas (ADR resumidas)

| # | Decisión | Alternativas consideradas | Motivo |
|---|---|---|---|
| A1 | **Microservicios** por *bounded context* (DDD) | Monolito modular | Evolución/escala independiente por dominio; evita servicios anémicos |
| A2 | **Database-per-service** (PostgreSQL; TimescaleDB para telemetría) | BD compartida | Aislamiento y ownership; ningún servicio accede a la BD de otro (RNF-14) |
| A3 | **Arquitectura orientada a eventos** (RabbitMQ, *topic exchange*) | Cadenas de llamadas síncronas | Desacoplamiento; reacción sin acoplamiento directo |
| A4 | **API Gateway** (Spring Cloud Gateway) + **Keycloak** (OAuth2/OIDC) | Auth por servicio sin gateway | AuthN/rate-limit centralizados; RBAC además **independiente en cada servicio** |
| A5 | **Poliglota justificado por dominio** | Un solo lenguaje | Python = estándar de automatización de red/seguridad (Netmiko/Ansible/Nmap); Java = lógica de negocio pesada (RBAC, máquinas de estado) |
| A6 | **Resiliencia por diseño** (Resilience4j: timeout, retry+backoff, circuit breaker) | Llamadas sin protección | Cada llamada asume que el otro puede fallar/estar lento (RNF-10) |
| A7 | **Observabilidad de 3 pilares** (Prometheus+Grafana, Loki+Promtail, OpenTelemetry+Jaeger) | Solo logs | Métricas + logs estructurados (RNF-17) + tracing con `traceId` propagado (RNF-16) |
| A8 | **Contract-first** (OpenAPI 3.1) + **contract testing** (Pact) | Código primero | Frontend/backend en paralelo; documentación viva; rupturas detectadas en CI (RNF-21) |
| A9 | **Alcance de escaneo autorizado como control técnico** (lista blanca CIDR por entorno) | Solo política declarativa | Rechazo/registro de objetivos fuera de alcance (RNF-07, RF-17) |
| A10 | **IaC con Terraform**; entorno AWS "apagado por defecto" | Provisión manual | Reproducible (`apply`/`destroy`); presupuesto acotado (RNF-23, RNF-24) |
| ADR-01 | **Respaldos: Git interno (contenido) + PostgreSQL (metadatos)** | Todo en BD / todo en Git | Git aporta versionado y diff nativos (RF-07); PostgreSQL aporta consulta/filtro/paginación. Ver §4.4 y plan §13 |
| ADR-02 | **Respaldo de `running-config` + `startup-config`** con flag `unsavedChanges` y **escalamiento por evento** | Solo running-config | Detecta cambios sin guardar; escala vía `config.unsaved_changes_detected` (regla configurable en alerting) |
| ADR-03 | **Canales de notificación enchufables** (adapter/strategy) | Canales cableados | Email + Telegram como adaptadores MVP (RF-32); nuevos canales sin refactor |
| ADR-04 | **Transactional outbox** para publicar eventos al broker | Publicación directa (dual write) | Consistencia BD↔RabbitMQ: el evento se publica solo si la transacción de BD confirmó. Ver `especificaciones/comunicacion_por_eventos.md` |
| ADR-05 | **Contract-first en Java** (openapi-generator) y **code-first + check de divergencia en Python** (FastAPI) | Code-first uniforme (springdoc/FastAPI) / contract-first uniforme | Java genera interfaces+DTOs desde el `openapi.yaml` (cumple el contrato por construcción); Python genera su OpenAPI desde el código y CI lo compara contra el `openapi.yaml` comprometido (falla si divergen). Mantiene el `openapi.yaml` como fuente de verdad en ambos stacks |
| ADR-06 | **Mapeo entidad↔DTO: MapStruct en Java, Pydantic `from_attributes` en Python** | Mapeo manual / ModelMapper (reflexión) | Java: MapStruct genera mappers en compilación (type-safe, sin reflexión), en el POM padre. Python: no requiere librería aparte; Pydantic convierte ORM↔DTO. Cumple la regla "DTO↔entidad explícito, nunca exponer entidades" |
| ADR-07 | **Columnas de auditoría estándar** (`created_at/by`, `updated_at/by`) auto-pobladas | Sin auditoría / auditoría manual | Patrón único en toda entidad persistida: cuarteto en entidades modificables por usuario; `created_*` en registros disparados por usuario; los generados por el sistema se rastrean por su timestamp + evento origen. Java: JPA Auditing; Python: SQLAlchemy + servicio, `*_by` desde el JWT. Ver `estandares_desarrollo.md §11` |
| ADR-08 | **Modelo de error `RFC 7807/9457` (Problem Details)** + `PATCH` = **JSON Merge Patch (RFC 7386)** | Error propietario `{code,message}` / PATCH ambiguo | `application/problem+json` con `type,title,status,detail,instance` (+ `traceId`); PATCH parcial con semántica merge. Estándar de la industria; reemplaza el `ApiError` propietario. Global (10 servicios) |
| ADR-09 | **Fiabilidad y concurrencia HTTP:** `Idempotency-Key` en escrituras + `ETag`/`If-Match` (RFC 7232) | Sin idempotencia (duplicados por reintento) / *lost updates* | POST de creación acepta `Idempotency-Key`, **acotado por usuario** y validado contra el **hash del cuerpo** (misma clave + cuerpo distinto → 409, no replay silencioso); GET devuelve `ETag`, y `PUT/PATCH/DELETE` exigen `If-Match` → mapea el bloqueo optimista (`@Version`) al protocolo HTTP. Global |
| ADR-10 | **Health probes diferenciados** liveness / readiness / startup | `/health` único | `GET /health/liveness` (¿el proceso vive?), `/health/readiness` (¿listo para tráfico: BD/broker OK?), startup. Spring Actuator *health groups* / equivalente FastAPI. Requisito de Kubernetes (no enrutar hasta readiness). Global |
| ADR-11 | **Seguridad de datos:** redacción de campos sensibles por rol (server-side) + **log de auditoría de seguridad** (OWASP A09) | Confiar en ocultar en cliente / sin traza de seguridad | Matriz campo×rol aplicada en el servidor (p. ej. `mgmtIp` enmascarada para Auditor); log estructurado marca `security` de accesos denegados (403), autenticaciones fallidas y mutaciones con actor. Global |
| ADR-12 | **Gobernanza de contratos con Spectral en CI** | Consistencia manual / solo revisión | `.spectral.yaml` codifica los estándares OpenAPI (ADR-08..11) como reglas verificables; un workflow los aplica a **todo** `openapi.yaml` en cada push/PR y **falla el gate** si un contrato (Fase A o B) no cumple. Enforcement automático en vez de disciplina; garantiza consistencia al crecer a 10 servicios |
| ADR-13 | **Direccionamiento de gestión dual-stack** (IPv4/IPv6) modelado tipo IPAM (RF-05a) | Un solo `mgmtIp` string IPv4 | El dispositivo tiene `managementIpv4` y/o `managementIpv6` (al estilo NetBox `primary_ip4`/`primary_ip6`), cada uno con **dirección + prefijo CIDR + gateway**. IPv6 se **canonicaliza** (RFC 5952) antes de persistir para que la unicidad sea real; validación de formato por familia autoritativa en el servidor (`java.net.InetAddress`); redacción por rol enmascara la porción de host de ambas familias (extiende ADR-11). Afecta el contrato y el payload de eventos `asset.*` (→ `version` 1.1.0). Aplica a `asset-inventory`; los consumidores que lean la IP se ajustan |
| ADR-14 | **Validación de JWT en profundidad** (issuer + audience + expiración) por servicio (RNF-29) | Validar solo firma / confiar en el Gateway | Cada Resource Server construye su `JwtDecoder` con validadores de **`issuer`** (URL del realm) y **`audience`** (claim `aud`/`azp` esperado) además de firma y expiración; un token de otro realm/cliente → `401`. Defensa en profundidad; no se asume que el Gateway ya filtró. Global (todos los servicios con API protegida) |
| ADR-15 | **Entrega garantizada de eventos: outbox endurecido** (RNF-30) | Outbox con relay best-effort, sin confirms, no seguro entre réplicas | Al transactional outbox (ADR-04) se añade: **publisher confirms** del broker (un mensaje solo se marca publicado tras el ACK), relay **seguro con múltiples réplicas** vía `SELECT … FOR UPDATE SKIP LOCKED`, reintentos y **DLQ**. Consumidores idempotentes. Garantiza at-least-once real sin duplicados descontrolados al escalar. Global (productores/consumidores de eventos) |
| ADR-16 | **Gate de cadena de suministro en CI** (RNF-08) | Solo Dependabot (avisos, no bloqueante) | El workflow de cada servicio corre **SCA de dependencias** (OWASP dependency-check / pip-audit) y **escaneo de imagen** (Trivy), **bloqueantes en severidad crítica**, además de Dependabot. El escaneo pesado con base NVD puede ir en un job programado. Global |
| ADR-17 | **OpenAPI servido en runtime (Swagger UI)** (RNF-27) | Solo el `openapi.yaml` estático en el repo | Cada servicio expone **Swagger UI** y `\/v3\/api-docs` en runtime (springdoc en Java / equivalente FastAPI), sirviendo el contrato como fuente de verdad, sin auth para la doc. Contrato explorable en vivo para integradores y demo de UAT. Global |

> Las decisiones de endurecimiento (ADR-14..17) se rastrean por servicio en su `matriz_rnf.md` y su
> despliegue por etapas en [`preparacion_produccion.md`](preparacion_produccion.md) (RNF-31).

> Las ADR-01/02/03 se originaron en la revisión del plan general
> (`../planificacion/plan_general_proyecto_redSegura.md` §13) y se consolidan aquí como
> registro maestro.

---

## 3. Arquitectura del sistema

- **Componentes principales:** SPA Angular → API Gateway (BFF) → 10 microservicios (cada uno
  con su BD) ↔ RabbitMQ; infraestructura compartida: Service Discovery (Eureka/Consul),
  Config Server (Spring Cloud Config), Keycloak, y el stack de observabilidad. Diagrama:
  [`diagrama_arquitectura.md`](diagrama_arquitectura.md).
- **Microservicios (10):**

  | Servicio | Fase | Lenguaje | Responsabilidad |
  |---|---|---|---|
  | `asset-inventory-service` | A | Java | Inventario único de activos — fuente de verdad |
  | `config-backup-service` | A | Python | Respaldo (Netmiko/NAPALM), versionado Git, drift |
  | `compliance-audit-service` | A | Python | Auditoría de cumplimiento (Playbooks Ansible) |
  | `alerting-service` | A | Java | Evaluación de reglas y generación de alertas |
  | `notification-service` | A | Java | Envío de notificaciones (email/Telegram) |
  | `scan-orchestrator-service` | B | Python | Orquestación de escaneos (Nmap/OpenVAS) |
  | `vulnerability-service` | B | Java | Hallazgos, enriquecimiento CVE/CVSS (NVD) |
  | `remediation-tracking-service` | B | Java | Flujo de remediación tipo ticket (máquina de estados) |
  | `reporting-service` | B | Python | Reportes ejecutivos en PDF |
  | `telemetry-collector-service` | B | Python | Telemetría SNMP, series de tiempo |

- **Patrón por servicio:**
  - **Java/Spring:** `controller → service → repository`; DTO ↔ entidad (nunca exponer
    entidades); inyección por constructor; `@Transactional` en la capa de servicio;
    *global exception handler*.
  - **Python/FastAPI:** `router → service → repository`; DI nativa de FastAPI; Pydantic para
    DTOs; manejadores de excepción centralizados.
- **Comunicación:** síncrona vía API Gateway (REST) para consultas/comandos; **asíncrona** vía
  RabbitMQ para propagación de eventos de dominio (ver §4.3).

---

## 4. Contratos de integración

> **Regla del proyecto (pre-código):** antes de escribir un cliente que consuma otro servicio
> o un evento, verificar el contrato real — no asumir nombres de campos ni estructura.

### 4.1 Contratos de API (OpenAPI 3.1)
Cada servicio publica su contrato en `codigo/backend/<servicio>/openapi.yaml` (fuente
autoritativa). El detalle por endpoint (rutas, roles, esquemas) de **Fase A** está definido en
`../planificacion/plan_general_proyecto_redSegura.md` §7.3 y se materializa en los OpenAPI en la
tarea 10 de la Fundación. Estado: **Fase A definida (contratos por generar); Fase B pendiente.**

### 4.2 Reglas transversales de API
- **Versionado de ruta:** prefijo `/api/v1`.
- **Autenticación:** *Bearer* JWT (Keycloak); autorización por rol validada **en cada servicio**.
- **Modelo de error uniforme (RFC 7807/9457, ADR-08):** `application/problem+json` con
  `type,title,status,detail,instance` (+ `traceId`); sin filtrar internos. Códigos:
  400/401/403/404/409/412/422/428. `PATCH` = JSON Merge Patch (RFC 7386).
- **Fiabilidad/concurrencia (ADR-09):** `Idempotency-Key` en creaciones; `ETag`/`If-Match`
  (RFC 7232) en mutaciones (→ 412/428).
- **Health probes (ADR-10):** `/health/liveness` y `/health/readiness` (no autenticados).
- **Gobernanza (ADR-12):** estos estándares se verifican en CI con Spectral sobre todo `openapi.yaml`.
- **Paginación estándar** en toda colección:
  ```json
  { "content": [], "page": 0, "size": 20, "totalElements": 0, "totalPages": 0, "first": true, "last": true }
  ```
- **Filtrado obligatorio en colecciones:** todo endpoint de lista ofrece filtros por las
  propiedades más características del objeto (`hostname`, `mgmtIp`, `severity`, `status`, rango
  de fechas…), combinables (AND), omitiendo vacíos; búsqueda de texto insensible a
  mayúsculas/acentos.
- **Salud:** `GET /health` sin autenticación (RNF-12). **Trazabilidad:** `traceId` propagado.

### 4.3 Catálogo de eventos (message broker)
Topología: *topic exchange* `redsegura.events`, *routing keys* `<dominio>.<evento>`, una cola
por servicio consumidor, DLQ para no procesables. Sobre de evento: `eventId`, `eventType`,
`occurredAt`, `traceId`, `version`, `payload`. Esquema JSON detallado:
[`especificaciones/comunicacion_por_eventos.md`](especificaciones/comunicacion_por_eventos.md)
(tarea 6). Eventos de **Fase A**:

| Evento (routing key) | Publica | Consume |
|---|---|---|
| `asset.created` / `asset.updated` / `asset.decommissioned` | asset-inventory | config-backup, compliance-audit |
| `config.backup_completed` / `config.backup_failed` | config-backup | alerting |
| `config.drift_detected` | config-backup | alerting |
| `config.unsaved_changes_detected` | config-backup | alerting |
| `compliance.finding_created` | compliance-audit | alerting |
| `alert.created` | alerting | notification |
| `vulnerability.critical_found` *(Fase B)* | vulnerability | alerting |

### 4.4 Almacenamiento de respaldos (ADR-01, resumen)
`config-backup-service` guarda el **contenido** de `running-config`/`startup-config` como
commits en un **repositorio Git interno** (propio del servicio; no GitHub, no repos de código)
y los **metadatos** (`backupId`, `deviceId`, `commit`, `unsavedChanges`, `status`…) en su
PostgreSQL. Diff y drift se resuelven leyendo Git por `commit`; historial/filtros, por la BD.

---

## 5. Seguridad y RBAC

- **Autenticación:** OAuth2/OIDC vía **Keycloak**; JWT propagado a cada microservicio.
  Expiración/renovación gestionadas por el Gateway. **RBAC de extremo a extremo:** se valida en
  el Gateway **y**, de forma independiente y obligatoria, **en cada microservicio** — nunca se
  confía solo en que el Gateway filtró (RNF-04).
- **Distinción 401/403:** 401 = no autenticado / token inválido; 403 = autenticado sin permiso.
- **Matriz de acceso por rol** (fuente: Especificación §5):

  | Módulo / Acción | Administrador | Operador | Auditor |
  |---|:---:|:---:|:---:|
  | Inventario — ver | ✓ | ✓ | ✓ |
  | Inventario — crear / editar / dar de baja | ✓ | — | — |
  | Respaldo de configuración — ver historial | ✓ | ✓ | ✓ |
  | Respaldo de configuración — ejecutar bajo demanda | ✓ | ✓ | — |
  | Auditoría de cumplimiento — ver hallazgos | ✓ | ✓ | ✓ |
  | Auditoría de cumplimiento — disparar auditoría | ✓ | ✓ | — |
  | Escaneo de vulnerabilidades — ver hallazgos | ✓ | ✓ | ✓ |
  | Escaneo de vulnerabilidades — disparar escaneo | ✓ | — | — |
  | Alcance autorizado de escaneo — configurar | ✓ | — | — |
  | Tickets de remediación — ver | ✓ | ✓ | ✓ |
  | Tickets de remediación — crear / cambiar estado | ✓ | ✓ (solo asignados a él) | — |
  | Reglas de alerta — configurar | ✓ | — | — |
  | Alertas — ver / reconocer | ✓ | ✓ | ✓ |
  | Reportes ejecutivos — generar / descargar | ✓ | ✓ | ✓ |
  | Gestión de usuarios y roles | ✓ | — | — |

  > **Ambigüedad a resolver (L01):** "Alertas — reconocer" figura con ✓ para Auditor, pero el
  > rol se define como solo lectura. Pendiente decidir si "reconocer" excluye a Auditor; hasta
  > entonces, los contratos siguen la matriz. Ver §9.

- **Campos sensibles (redacción server-side por rol):**

  | Campo | Sensible | Comportamiento para rol no autorizado |
  |---|---|---|
  | Credenciales SSH / SNMP de dispositivos | Siempre secretas | Nunca se exponen en API ni logs; externalizadas (gestor de secretos) |
  | Detalle interno de errores (stack trace, tablas) | Siempre | Sustituido por mensaje genérico + `traceId` |

- **Secretos:** externalizados por variable de entorno / gestor de secretos; nunca en git ni en
  el historial (RNF-06). Logs estructurados sin PII ni secretos (RNF-17).

---

## 6. Configuración y entornos

| Variable (ejemplo) | Descripción | Dev | Prod |
|---|---|---|---|
| `DB_URL` / `DB_USER` / `DB_PASSWORD` | Conexión PostgreSQL del servicio | local/compose | secreto (RDS) |
| `RABBITMQ_URL` | Broker de eventos | compose | secreto |
| `KEYCLOAK_ISSUER_URI` | Emisor OIDC | local | secreto |
| `SCAN_AUTHORIZED_CIDRS` | Lista blanca de alcance de escaneo (RNF-07) | rangos de red simulada | rangos autorizados por el cliente |

Entornos: local (Docker Compose) → Kubernetes (k3d local → k3s en EC2 → ventana EKS). Principio
12-factor: mismo código, distinta configuración (RNF-22).

### 6.1 CI/CD
- **Gatekeeper por microservicio** (no global): build + tests + lint/análisis estático +
  cobertura ≥ 70 %, en entorno limpio, activado por **ruta** (`codigo/backend/<servicio>/**`).
- **Contract testing (Pact):** workflow separado; verifica que nadie rompió un contrato
  consumido por otro antes de publicar imagen.
- **CD:** build de imagen Docker por servicio → publicación en registry (GHCR/ECR) etiquetada
  por SHA. Despliegue controlado (Continuous Delivery, no Deployment automático).
- **Compuerta:** branch protection en `main`/`develop` (require PR + status checks) — capa
  servidor; complementada por el hook `pre-commit` local (capa local). *Pendiente:* se activa al
  crear los repos remotos (tarea 14 de la Fundación).

### 6.2 Versionado y releases
SemVer **por microservicio**, con tags prefijados por servicio (`asset-inventory-service-v0.1.0`).
Tags anotados e **inmutables**. CHANGELOG (*Keep a Changelog*) por repositorio, actualizado
antes de tagear.

### 6.3 Gobernanza de seguridad de dependencias (SCA)
`SECURITY.md` por repo (reporte privado); **Dependabot** (`maven`, `pip`, `github-actions`);
escaneo rápido en el gate (`mvn dependency-check` / `pip-audit`), bloqueante en `critical`;
escaneo pesado (OWASP/NVD) en workflow programado. Sin secretos en git (RNF-06, RNF-08).

### 6.4 Operación (Day-2) — *diferido a la fase de despliegue*
Backup/restore probado (RPO/RTO), monitoreo de uptime + alertas, retención de logs, graceful
shutdown, límites de recursos, rotación de credenciales. Detalle en el runbook de despliegue.

---

## 7. Estándares transversales

Enlace: [`estandares_desarrollo.md`](estandares_desarrollo.md) (tarea 5). Reglas que aplican a
todas las capas: manejo global de errores, excepciones tipadas de negocio, paginación y filtrado
estándar, logging estructurado (RNF-17), inyección por constructor / DI nativa, transacciones en
la capa de servicio, y contratos OpenAPI desde la primera versión (RNF-27).

---

## 8. Estado del proyecto y métricas

- **Fase:** Fundación **completa**; Fase A **en construcción**. `asset-inventory-service` en curso
  (dominio + CRUD + búsqueda/filtros + seguridad RBAC + ETag/If-Match + idempotencia endurecida +
  redacción `mgmtIp` por rol + log de seguridad + auditoría de capacidad productiva); resto de
  servicios sin iniciar.
- **Repositorios:** `management` y `backend` versionados y **publicados en GitHub**
  (`redsegura-management`, `redsegura-backend`, públicos) con branch protection (`main` requiere
  PR; `develop` con historial protegido). `frontend` sin inicializar.
- **CI/CD:** CI de `asset-inventory-service` **activo** (push/PR con filtro por rutas); el gatekeeper
  (`mvn verify`: build + tests Testcontainers + cobertura + lint) corre en GitHub Actions (JDK 21,
  actions `@v5`) y es **status check requerido** en `main` (bloquea merges rojos). Nota monorepo:
  al sumar servicios, añadir un check "paraguas" para PRs que no toquen `asset-inventory`.
- **Documentación de arquitectura:** ✅ completa — plan general, memoria técnica global (este
  doc, **ADR-01..13**), diagrama, estándares, especificación de eventos y protocolo de QA.
- **Contratos:** ✅ 5 contratos OpenAPI de Fase A definidos, validados y **sintonizados con
  estándares de industria** (RFC 7807, probes, Idempotency-Key/ETag, ADR-08..11); **gobernados
  en CI con Spectral** (ADR-12). Catálogo de eventos definido.
- **Tests / cobertura:** `asset-inventory-service` — **105 tests** en verde (unit + Testcontainers
  PostgreSQL/RabbitMQ + MockMvc), cobertura ≥ 70 % statements, 0 Checkstyle. Resto de servicios: N/A.
  Verificación en vivo de los 10 endpoints (curl/Postman sobre Docker Compose): 10/10 ✅
  (`backend/asset-inventory-service/documentos/verificacion_endpoints.md`).
  Umbral: ≥ 70 %.

---

## 9. Registro de lecciones globales

| ID | Lección | Origen | Aplicación |
|---|---|---|---|
| L01 | Resolver explícitamente si "reconocer alerta" excluye al rol Auditor (la matriz §5 lo permite, pero Auditor es solo lectura) | Revisión de contratos de Fase A | Decisión a fijar antes de implementar `alerting-service`; ajustar contrato y matriz si aplica |
| L02 | En dispositivos de red, `running-config` ≠ `startup-config` implica cambios sin guardar: capturar ambas y escalar la divergencia | ADR-02 | `config-backup-service` y política futura de `compliance-audit-service` |
| L03 | **Capacidad productiva real** (no solo portafolio): (a) un parámetro del contrato sin cablear falla en silencio → test por cada filtro; (b) `readiness` debe verificar dependencias reales (BD), no devolver `UP` fijo; (c) validar entrada libre (`sort`, `size`) **antes** de la capa de datos, para no filtrar internos (RNF-09) ni exponer DoS | Auditoría de `asset-inventory-service` | Checklist de revisión aplicable a **todos** los microservicios antes de cerrar cada módulo |
| L04 | Un gate de calidad (spotless/checkstyle) que solo está en `pluginManagement` **no se ejecuta** en `mvn verify`; hay que **ligarlo a una fase**. Se detectó al activar el CI (que sí corría el goal explícito) → violaciones de formato acumuladas | Activación del CI de `asset-inventory` | `spotless:check` + `checkstyle:check` ligados a `verify` en el POM padre; `mvn verify` es el gatekeeper único; el CI corre solo `verify` |
| L05 | Una restricción de contrato en un **parámetro** (`@Max`/`@Min` de query/path) lanza `ConstraintViolationException`, que **sin handler se escapa como 500 con fuga** (viola RNF-09). El límite de paginación debe tener **una sola fuente de verdad** (el contrato), no un cap distinto en código | QA R1 de `asset-inventory` | Manejar `ConstraintViolationException` → 422 `problem+json` en cada servicio; alinear límites contrato↔código. Detalle en `qa/reporte_qa.md` (L-QA-01/02) |

---

## 10. Roadmap / pendientes

**Fundación — completada:**
- [x] Arquitectura global (memoria técnica, diagrama, estándares, eventos, protocolo de QA).
- [x] POM padre (Java 21, calidad, MapStruct) + 5 contratos OpenAPI de Fase A (validados).
- [x] Documentación de ambos repositorios; `git init` + git-hook; merges `--no-ff` a `develop`.
- [x] Repos remotos en GitHub + branch protection + esqueleto de CI.

**Fase A — en curso:**
- [~] `asset-inventory-service`: dominio + CRUD + búsqueda/filtros + RBAC + ETag/If-Match +
  idempotencia endurecida + **direccionamiento dual-stack IPv4/IPv6** (RF-05a) + redacción de
  direcciones por rol + 401/403 `problem+json` + log de seguridad (OWASP A09) + observabilidad +
  eventos `asset.*` vía transactional outbox (RN11) + importación masiva asíncrona (RF-04) + **CI
  activo/gateando + **QA R1 ✅ certificada** (105 tests, reporte en `qa/reporte_qa.md`) + verificación
  en vivo de 10/10 endpoints ✅; **pendiente**:
  Pact (al existir el primer consumidor).
- [ ] `config-backup-service`, `compliance-audit-service`, `alerting-service`, `notification-service`.
- [ ] Golden path E2E en Docker Compose.
- [ ] Trasladar decisiones y lecciones a este documento al cerrar cada módulo.

**Deuda transversal de producción (aplicable a todos los servicios):**
- [x] **Observabilidad (RNF-15/16/17) — Java:** `micrometer-registry-prometheus` (métricas),
  `micrometer-tracing-bridge-otel` (traceId/spanId) y `logstash-logback-encoder` (logs JSON) en el
  **POM padre** + `logback-spring.xml` de referencia. Pendiente: exportadores por entorno (OTLP→Jaeger,
  scrape Prometheus), extraer `logback-spring.xml` a un módulo commons, y el equivalente en Python.
- [ ] **Seguridad JWT:** validación de `issuer`/`audience` (hoy solo firma) vía `OAuth2TokenValidator`,
  al fijar el realm de Keycloak.
