# Proyecto: redSegura — Plataforma de Automatización, Monitoreo y Gestión de Vulnerabilidades de Red

**Estado:** aprobado (opción elegida) · **Versión:** 1.2.0 · **Fecha:** 2026-07-10
**Autor:** David Reyna Pineda

> Este documento es la **especificación definitiva, específica y
> autocontenida** del proyecto. Incluye toda la información de arquitectura,
> alcance, requerimientos y organización necesaria para entenderlo sin
> depender de otros documentos. El `anteproyecto_microservicios.md` y
> `especificaciones_diseño.md` (ver Sección 14) documentan el proceso de
> exploración y las prácticas base que dieron origen a este proyecto, pero
> no son lectura requerida para trabajar sobre él.

---

## 1. Resumen ejecutivo

**redSegura** es una plataforma de microservicios que centraliza el inventario,
respaldo de configuraciones, auditoría de cumplimiento, gestión de
vulnerabilidades y monitoreo de dispositivos de red — combinando dos
funciones que normalmente se venden como productos separados en el mercado
(automatización de red + gestión de vulnerabilidades), unificadas sobre un
mismo inventario de activos. Es un proyecto de portafolio con exigencia de
calidad de industria: arquitectura de microservicios completa,
documentación pedagógica exhaustiva, y un despliegue de prueba real en AWS
con ruta a Kubernetes.

---

## 2. Problema y oportunidad

- **Problema que resuelve:** en redes de tamaño medio, el respaldo de
  configuraciones, la verificación de cumplimiento y la detección de
  vulnerabilidades se hacen manualmente, dispositivo por dispositivo, o con
  herramientas desconectadas entre sí (un inventario en una hoja de cálculo,
  escaneos de vulnerabilidades en otra herramienta, sin correlación entre
  "qué tengo" y "qué tan seguro/cumplido está").
- **Oportunidad:** unificar inventario + postura de configuración + postura
  de vulnerabilidades en una sola fuente de verdad reduce el tiempo de
  auditoría y mejora la capacidad de respuesta ante incidentes.
- **Por qué ahora (para el autor):** es el proyecto de portafolio que
  demuestra, en un solo sistema, la combinación de networking (CCNA),
  automatización (Python/Netmiko/Ansible) y desarrollo de software
  (Spring Boot/Angular/PostgreSQL) — la combinación diferenciadora
  identificada en el plan de desarrollo profesional.

---

## 3. Objetivos

**General:** construir y desplegar una plataforma de microservicios,
funcional de extremo a extremo, que automatice el respaldo, la auditoría de
cumplimiento y la gestión de vulnerabilidades de una red de dispositivos,
demostrando competencias de nivel industria en arquitectura de
microservicios, Python, Java, Kubernetes y AWS.

**Específicos (medibles):**
- O1: Entregar la **Fase A (núcleo redSegura)** funcional en Docker Compose local, con los 5 microservicios del golden path, documentados y con tests en verde.
- O2: Entregar la **Fase B (gestión de vulnerabilidades)** como extensión funcional sobre la Fase A, con los 5 microservicios adicionales.
- O3: Migrar el sistema de Docker Compose → Kubernetes local (k3d/minikube) → Kubernetes en AWS (k3s en EC2), con al menos una demostración documentada en Amazon EKS.
- O4: Certificar el sistema con el Protocolo de verificación en 4 fases (mínimo cobertura de pruebas ≥70% por microservicio).
- O5: Publicar los 3 repositorios en GitHub con documentación completa (README, CHANGELOG, CLAUDE.md, docs indexados) como pieza de portafolio.

---

## 4. Alcance

### 4.1 Incluye — Fase A (núcleo redSegura, MVP prioritario)
- `asset-inventory-service`, `config-backup-service`, `compliance-audit-service`, `alerting-service`, `notification-service`.
- Dashboard Angular con vista de inventario, estado de respaldo y alertas.
- Despliegue en Docker Compose (local) y despliegue de prueba en AWS (EC2 + RDS).

### 4.2 Incluye — Fase B (extensión de gestión de vulnerabilidades)
- `scan-orchestrator-service`, `vulnerability-service`, `remediation-tracking-service`, `reporting-service`, `telemetry-collector-service`.
- Dashboard ampliado con vista de vulnerabilidades, severidad CVSS, y reportes ejecutivos en PDF.
- Migración del despliegue de prueba a Kubernetes (k3s en EC2, con ventana corta opcional en EKS).

### 4.3 Capacidad de producción vs. entorno usado durante el proyecto

Es importante distinguir dos cosas que no deben confundirse:

- **Capacidad funcional del sistema:** redSegura se diseña y construye para
  poder operar sobre redes en **producción real** — esa es su finalidad de
  negocio (RF-15 a RF-19, Sección 7.4). El alcance de escaneo/automatización
  es **configurable por entorno**, de modo que un despliegue para un cliente
  real se apunte a su red, con la autorización explícita correspondiente.
- **Entorno usado durante el desarrollo, pruebas y despliegue de ESTE
  proyecto:** mientras se construye y valida redSegura como proyecto de
  portafolio, el sistema **nunca** se apunta contra una red de producción
  real ni de terceros. Se utiliza exclusivamente una **red simulada en
  GNS3 o Cisco Packet Tracer**, dimensionada para ejercitar todos los
  servicios de forma realista (ver Sección 6.6 para la topología
  recomendada).

Esta distinción es tanto un requisito ético/legal (nunca escanear
infraestructura sin autorización explícita) como una decisión de diseño
verificable: el alcance autorizado de escaneo es un control técnico
(lista blanca configurable), no solo una política de buenas intenciones.

### 4.4 Fuera de alcance (explícito)
- Service mesh (Istio/Linkerd), arquitectura multi-región, despliegues blue-green/canary avanzados, operadores personalizados de Kubernetes, mTLS entre servicios (documentado como mejora futura, no como entregable del MVP).
- Soporte multi-tenant (el sistema asume un solo cliente/organización por despliegue).
- Cualquier ejecución de escaneo o automatización contra redes de terceros sin autorización explícita — control técnico y de proceso, no solo una restricción declarativa.

### 4.5 Supuestos y restricciones
- Se dispone de una cuenta AWS de prueba con saldo limitado (ver Sección 10).
- El entorno de red objetivo durante desarrollo/pruebas es la red simulada descrita en la Sección 6.6, no infraestructura real de terceros.

---

## 5. Usuarios y roles (RBAC)

| Rol | Necesidad principal |
|---|---|
| **Administrador** | Gestionar el sistema completo: inventario, reglas, usuarios, alcance autorizado de escaneo |
| **Operador** | Operación diaria: disparar respaldos/auditorías, dar seguimiento a tickets de remediación asignados |
| **Auditor / Solo lectura** | Revisar la postura de cumplimiento y seguridad sin poder modificar nada ni disparar acciones |

**Matriz de acceso detallada por módulo/acción:**

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
| Tickets de remediación — crear / cambiar estado | ✓ | ✓ (solo los asignados a él) | — |
| Reglas de alerta — configurar | ✓ | — | — |
| Alertas — ver / reconocer | ✓ | ✓ | ✓ |
| Reportes ejecutivos — generar / descargar | ✓ | ✓ | ✓ |
| Gestión de usuarios y roles | ✓ | — | — |

**Autenticación:** OAuth2/OIDC vía Keycloak, con JWT propagado a cada
microservicio; el rol activo determina qué acciones se habilitan tanto en
la interfaz (Angular) como — de forma obligatoria e independiente — en cada
endpoint del backend (nunca se confía solo en ocultar botones en el
cliente).

---

## 6. Arquitectura

### 6.1 Principios de diseño

- **Domain-Driven Design (DDD) — bounded contexts:** cada microservicio
  representa un subdominio de negocio con responsabilidad única, evitando
  microservicios "anémicos" que solo hacen operaciones básicas sin lógica
  de dominio real.
- **Database-per-service:** cada microservicio es dueño exclusivo de su
  propia base de datos (PostgreSQL). Ningún otro servicio accede
  directamente a ella — solo a través de su API o de eventos.
- **12-Factor App:** configuración externa vía variables de entorno, logs
  como stream de eventos, procesos sin estado (stateless), dependencias
  declaradas explícitamente, builds reproducibles.
- **API-first / contract-first:** cada servicio expone un contrato OpenAPI
  definido antes de implementarse, permitiendo que frontend y backend
  avancen en paralelo y sirviendo como documentación viva.
- **Arquitectura orientada a eventos:** la comunicación entre servicios que
  no requiere respuesta síncrona inmediata se hace vía eventos publicados
  en un message broker, evitando cadenas frágiles de llamadas directas
  entre microservicios.
- **Resiliencia por diseño:** cada llamada entre servicios asume que el
  otro puede fallar, estar lento o no responder — con timeouts, reintentos
  y circuit breaker.

### 6.2 Qué hace cada componente (explicado para el usuario final)

Descripción de cada microservicio en lenguaje llano, sin jerga técnica,
pensada para que un usuario o cliente sin conocimientos de programación
entienda su propósito:

- **Inventario de activos:** es la libreta de direcciones de la red — aquí
  se registra qué dispositivos existen, dónde están y qué tan importantes
  son para el negocio. Todos los demás componentes consultan esta libreta
  antes de hacer cualquier cosa, para asegurarse de que siempre hablan del
  mismo dispositivo.
- **Respaldo de configuraciones:** es una copia de seguridad automática de
  la configuración de cada dispositivo de red, como un respaldo de fotos en
  la nube pero para la "memoria" de un router o switch. Si alguien borra o
  daña una configuración por accidente, se puede recuperar la última
  versión guardada y ver exactamente qué cambió y cuándo.
- **Auditoría de cumplimiento:** es un inspector automático que revisa,
  dispositivo por dispositivo, si se están siguiendo las reglas básicas de
  buena configuración (por ejemplo, que la hora esté sincronizada o que las
  contraseñas no viajen sin cifrar). Si encuentra algo mal, lo reporta para
  que se corrija.
- **Orquestador de escaneos:** es quien organiza y lanza las revisiones de
  seguridad sobre la red — parecido a un chequeo médico periódico, pero
  para detectar puertas abiertas o puntos débiles que alguien mal
  intencionado podría aprovechar.
- **Gestión de vulnerabilidades:** es quien recibe los resultados de esas
  revisiones de seguridad, los organiza, y les pone una calificación de qué
  tan grave es cada problema encontrado (usando un estándar internacional),
  para saber qué atender primero.
- **Seguimiento de remediación:** es el tablero de pendientes del sistema —
  convierte cada problema detectado (de configuración o de seguridad) en
  una tarea con responsable, prioridad y estado, dándole seguimiento hasta
  que quede resuelto y verificado, como un sistema de tickets de soporte.
- **Alertas:** es quien vigila todo lo que pasa en el sistema y decide,
  según reglas definidas, cuándo algo merece un aviso — por ejemplo, si
  aparece un cambio de configuración inesperado o un hallazgo de seguridad
  grave.
- **Notificaciones:** es quien realmente envía el aviso a la persona
  correspondiente, por correo o por Telegram, cuando se genera una alerta —
  para que nadie tenga que estar revisando el sistema constantemente.
- **Reportes:** arma reportes ejecutivos en PDF, listos para presentar a un
  jefe o cliente, resumiendo el estado de cumplimiento y seguridad de la
  red sin necesidad de entrar al sistema.
- **Telemetría:** recolecta datos de "signos vitales" de los dispositivos
  (si están encendidos, cuánto recurso están usando) para tener un
  historial de su comportamiento a lo largo del tiempo.
- **Panel de control (dashboard):** es la pantalla principal donde una
  persona ve, de un vistazo, cómo está la red — cuántos dispositivos hay,
  si hay alertas activas, qué tan cumplida está la configuración, y qué
  vulnerabilidades existen — sin necesidad de saber programar ni usar la
  línea de comandos.

### 6.3 Microservicios y lenguaje asignado

| Microservicio | Fase | Lenguaje | Responsabilidad técnica |
|---|---|---|---|
| `asset-inventory-service` | A | Java (Spring Boot) | Inventario único de activos — fuente de verdad |
| `config-backup-service` | A | **Python (FastAPI)** | Respaldo de configuraciones vía Netmiko/NAPALM, versionado en Git |
| `compliance-audit-service` | A | Python (FastAPI) | Auditoría de cumplimiento vía Playbooks Ansible |
| `alerting-service` | A | Java (Spring Boot) | Evaluación de reglas y generación de alertas |
| `notification-service` | A | Java (Spring Boot) | Envío efectivo de notificaciones (email/Telegram) |
| `scan-orchestrator-service` | B | **Python (FastAPI)** | Orquestación de escaneos Nmap/OpenVAS |
| `vulnerability-service` | B | Java (Spring Boot) | Hallazgos, enriquecimiento CVE/CVSS |
| `remediation-tracking-service` | B | Java (Spring Boot) | Flujo de remediación tipo ticket |
| `reporting-service` | B | Python (FastAPI) | Reportes ejecutivos en PDF |
| `telemetry-collector-service` | B | **Python (FastAPI)** | Telemetría SNMP/streaming |

El sistema es deliberadamente **poliglota**: Python se asigna a los
servicios donde es el estándar de facto de la industria (automatización de
red y seguridad — Netmiko, Ansible, Nmap/OpenVAS), y Java/Spring Boot se
mantiene en los servicios de lógica de negocio pesada (RBAC, máquinas de
estado, relaciones de datos), reforzando ambos lenguajes de forma
justificada por el dominio de cada servicio, no de forma arbitraria.

### 6.4 Capas transversales

**a) Frontend — Angular SPA.** Angular + Angular Material. Comunicación en
tiempo real con el backend vía WebSocket/SSE para el dashboard. Estados de
carga/vacío/error explícitos, diseño responsive, accesibilidad WCAG 2.1 AA
básica.

**b) API Gateway / Backend-for-Frontend.** Punto único de entrada
(Spring Cloud Gateway): enrutamiento a los microservicios internos,
autenticación/autorización centralizada, rate limiting, agregación de
respuestas cuando el frontend necesita una vista compuesta.

**c) Service Discovery & Config Server.** Service Discovery (Eureka/Consul)
para que los microservicios se encuentren dinámicamente entre sí. Config
Server (Spring Cloud Config) para centralizar la configuración de todos los
servicios en un repositorio Git, evitando archivos de configuración
dispersos.

**d) Comunicación asíncrona — Message Broker.** RabbitMQ. Patrón
publish/subscribe: un servicio publica un evento (p. ej.
`vulnerability.critical_found`) y otros servicios reaccionan sin
acoplamiento directo.

**e) Persistencia.** PostgreSQL como motor principal, una base de datos
independiente por microservicio. TimescaleDB (extensión de PostgreSQL) para
la telemetría de alta frecuencia.

**f) Observabilidad — tres pilares.** Métricas (Prometheus + Micrometer,
visualizadas en Grafana). Logs centralizados (Loki + Promtail). Trazabilidad
distribuida (OpenTelemetry + Jaeger), con un identificador de correlación
propagado entre servicios para una misma solicitud.

**g) Seguridad.** Autenticación centralizada en el API Gateway (OAuth2/OIDC
vía Keycloak); JWT propagado y validado por cada microservicio; gestión de
secretos fuera del código fuente; Resilience4j para circuit breaker, rate
limiting y timeouts; aplicación de OWASP Top 10 / OWASP API Security Top 10
en cada servicio.

**h) CI/CD y despliegue.** GitHub Actions construye la imagen Docker de
cada servicio, corre tests, y publica la imagen en un registry (GitHub
Container Registry / ECR). Docker Compose para desarrollo local; Kubernetes
(k3s/EKS) para el despliegue de prueba en AWS (ver Sección 6.7).

### 6.5 Diagrama de capas (referencia)

```
                          ┌─────────────────────────┐
                          │   Angular SPA (UI/UX)    │
                          │  Dashboard + WebSocket    │
                          └────────────┬─────────────┘
                                       │ HTTPS / WSS
                          ┌────────────▼─────────────┐
                          │   API Gateway (BFF)       │
                          │  AuthN/AuthZ · Rate limit │
                          └──────┬─────────────┬──────┘
                    ┌────────────┘             └────────────┐
          ┌─────────▼─────────┐               ┌─────────────▼────────┐
          │ Service Discovery │               │   Config Server        │
          │   (Eureka/Consul) │               │ (Spring Cloud Config) │
          └─────────┬─────────┘               └───────────────────────┘
                    │
   ┌────────────────┼──────────────── … (10 microservicios) ───────────┐
   │                │                                                    │
┌──▼──────┐   ┌──────▼─────┐                                    ┌────────▼────────┐
│ asset-  │   │  config-   │   ...                              │  telemetry-      │
│inventory│   │  backup    │                                    │  collector       │
│ +su DB  │   │  +su DB    │                                    │  +su DB          │
└──┬──────┘   └──────┬─────┘                                    └────────┬────────┘
   │                 │                                                    │
   └─────────────────┴──────────────── Message Broker (RabbitMQ) ────────┘

     Observabilidad transversal: Prometheus + Grafana (métricas),
     Loki (logs), Jaeger/OpenTelemetry (tracing) — cada servicio expone
     métricas/logs/trazas hacia este stack.
```

### 6.6 Entorno de red simulada para desarrollo y pruebas

Durante el desarrollo, las pruebas y el despliegue de prueba de este
proyecto, todos los servicios que interactúan con dispositivos de red
(`config-backup-service`, `compliance-audit-service`, `scan-orchestrator-
service`, `telemetry-collector-service`) operan exclusivamente contra una
**red simulada** en GNS3 o Cisco Packet Tracer — nunca contra una red de
producción real ni de terceros (ver Sección 4.3).

**Topología simulada mínima recomendada**, dimensionada para ejercitar
todos los servicios de forma realista:

- **Al menos 2 routers**, para ejercitar rutas de Capa 3 y dar diversidad de
  tipo de dispositivo al inventario (no solo switches).
- **Al menos 5 switches con enlaces redundantes**, reutilizando/extendiendo
  la topología STP ya construida durante el estudio de CCNA/ENCOR (SW1 a
  SW5, con root bridge, alternate y blocking ports), para dar variedad de
  roles y estados a las políticas de `compliance-audit-service`.
- **Al menos 3 VLANs**, para ejercitar segmentación en las políticas de
  auditoría.
- **2–3 hosts finales** (VMs Linux en VMware Fusion) conectados a la
  topología, para que `scan-orchestrator-service` también tenga objetivos
  de tipo servidor, no solo equipo de red — acercando la prueba a un
  escenario real de gestión de vulnerabilidades.
- **Al menos un dispositivo deliberadamente mal configurado** (contraseña
  en texto plano, SNMP con community `public`, versión insegura de SSH si
  el equipo lo permite), de forma controlada y documentada, para poder
  demostrar que `compliance-audit-service` y `vulnerability-service`
  detectan hallazgos reales — no solo un estado "todo correcto" que no
  demuestra nada.

El **alcance autorizado de escaneo** (Sección 7.4, RF-15 a RF-19) se
configura, durante esta etapa, exactamente con los rangos IP de esta red
simulada. En un uso posterior del sistema para un cliente real, ese mismo
mecanismo de configuración se usaría con los rangos que el cliente autorice
explícitamente sobre su propia red — la capacidad funcional del sistema no
cambia, solo el alcance configurado (ver Sección 4.3).

### 6.7 Estrategia de despliegue en AWS (entorno de prueba con presupuesto limitado)

**Presupuesto y restricciones.** Saldo disponible: MXN $4,958 (≈ USD
$250–270, según tipo de cambio). Ventana de tiempo: 80 días. Esto alcanza
para un despliegue de demostración/prueba, no para mantener el sistema
corriendo 24/7. El diseño clave es poder **encender y apagar el entorno bajo
demanda** en lugar de dejarlo corriendo de forma continua.

**Componentes AWS recomendados vs. a evitar:**

| Servicio AWS | Uso recomendado | Costo aproximado | Nota |
|---|---|---|---|
| EC2 (t3.small/t3.medium) | Host de contenedores (Docker Compose o k3s) | Elegible Free Tier si la cuenta tiene menos de 12 meses | Apagar (`stop`, no `terminate`) cuando no se use |
| RDS PostgreSQL (db.t3.micro) | Bases de datos de los microservicios | Elegible Free Tier (750 h/mes, 12 meses) | Alternativa de costo cero: PostgreSQL en contenedor dentro del mismo EC2 |
| S3 | Hosting del build de Angular + backups | Prácticamente gratuito a esta escala | — |
| ALB | Enrutamiento HTTPS hacia el API Gateway | ~USD $16–20/mes si corre continuo | Usarlo solo durante ventanas de demo, o sustituir por Elastic IP + Nginx mientras se desarrolla |
| ACM | Certificados SSL/HTTPS | Gratuito | — |
| ECR | Registro de imágenes Docker generadas por CI/CD | Costo mínimo por almacenamiento | — |
| CloudWatch (básico) | Logs/métricas mínimas de AWS | Nivel gratuito cubre uso bajo | Preferir Prometheus/Grafana propios |
| **EKS (control plane administrado)** | — | **USD $0.10/hora ≈ $73/mes si corre continuo** | **Evitar dejarlo corriendo de forma permanente** — ver Sección 6.8 |
| **NAT Gateway** | — | ~USD $0.045/hora + tráfico | **Evitar** — usar subredes públicas con Security Groups bien definidos |
| **MSK (Kafka administrado)** | — | Costoso a esta escala | Usar RabbitMQ autoalojado en un contenedor |

**Plan de uso del entorno:**
1. Configurar **AWS Budgets** con alertas en USD $50 / $100 / $180 desde el primer día.
2. Definir todo el entorno como código con **Terraform**, para crearlo y destruirlo en minutos.
3. Mantener el entorno **apagado por defecto**; encenderlo solo para demos, entrevistas técnicas o validar un cambio importante.
4. Reservar los últimos 10–15 días de la ventana de 80 días para el despliegue final completo.

### 6.8 Ruta de adopción de Kubernetes (Docker Compose → Kubernetes)

**Etapa 1 — Docker Compose (local, MacBook Air).** MVP funcional de los
microservicios, entorno de desarrollo diario.

**Etapa 2 — Kubernetes local (k3d o minikube, misma MacBook Air).** Una vez
estable Docker Compose, se migran los manifiestos a Kubernetes (Deployments,
Services, ConfigMaps, Secrets, Ingress) y se practica el ciclo completo
localmente, sin costo de nube.

**Etapa 3 — Kubernetes en AWS, autoadministrado (k3s en EC2).** Para el
despliegue de prueba en AWS, se instala **k3s** (Kubernetes ligero, sin
cargo de control plane) en 1–2 instancias EC2, evitando el costo fijo de
EKS mientras se demuestran las mismas competencias (manifiestos, HPA,
Ingress, Secrets).

**Etapa 4 (opcional, ventana corta) — Amazon EKS real.** Para demostrar
específicamente experiencia con el servicio administrado más usado de la
industria, se crea el clúster solo durante una ventana corta (2–3 días,
p. ej. antes de una entrevista técnica) con Terraform, y se destruye
inmediatamente después. Costo aproximado: USD $5–10.

### 6.9 Stack de tecnologías/estándares de la industria (nivel básico-medio)

| Categoría | Tecnología/estándar | Nivel objetivo |
|---|---|---|
| Infrastructure as Code | Terraform | Básico-medio: módulos para VPC, EC2/k3s, RDS, S3, IAM |
| CI/CD | GitHub Actions | Básico-medio: build + test + push a ECR + deploy |
| Contenedores | Docker, Docker Compose | Medio |
| Orquestación | Kubernetes (k3s / EKS) | Básico (ver Sección 6.8) |
| Testing | JUnit 5 / pytest + Testcontainers (PostgreSQL/RabbitMQ reales) | Básico-medio |
| Documentación de API | OpenAPI/Swagger | Básico |
| Autenticación | OAuth2/OIDC vía Keycloak, JWT | Básico-medio |
| Gestión de secretos | Variables de entorno / Kubernetes Secrets | Básico |
| Observabilidad | Prometheus + Grafana, OpenTelemetry (tracing) | Básico-medio |
| Mensajería | RabbitMQ | Básico-medio |
| Contract testing | Pact | Básico |

**Explícitamente fuera de alcance** (ver Sección 4.4): service mesh,
multi-región, blue-green/canary avanzado, operadores personalizados de
Kubernetes.

---

## 7. Requerimientos Funcionales (RF)

### 7.1 `asset-inventory-service`
- **RF-01:** El sistema debe permitir registrar un dispositivo de red con: hostname, **dirección(es) de gestión (IPv4 y/o IPv6) con su información complementaria (ver RF-05a)**, fabricante, modelo, ubicación física/lógica y nivel de criticidad de negocio (alta/media/baja).
- **RF-02:** El sistema debe permitir consultar, editar y dar de baja (soft delete) dispositivos del inventario.
- **RF-03:** El sistema debe exponer el inventario vía API REST como fuente única de verdad, consumible por los demás microservicios.
- **RF-04:** El sistema debe permitir búsqueda y filtrado de dispositivos por nombre, **dirección de gestión (IPv4 o IPv6)**, ubicación y criticidad.
- **RF-05:** El sistema debe publicar un evento (`asset.created`, `asset.updated`, `asset.decommissioned`) al message broker ante cada cambio relevante del inventario.
- **RF-05a (Direccionamiento de gestión dual-stack):** El sistema debe soportar **direccionamiento dual-stack** para la gestión del dispositivo, apegado a estándares de la industria de networking/IPAM (modelo tipo NetBox `primary_ip4`/`primary_ip6`; notación **CIDR**):
  - El usuario debe poder registrar una **dirección de gestión IPv4**, una **IPv6**, o **ambas**; al menos una es obligatoria.
  - Cada dirección lleva su **información complementaria**: la propia dirección, el **prefijo de red** (`prefixLength`/máscara CIDR: IPv4 0–32, IPv6 0–128) y, opcionalmente, la **puerta de enlace** (`gateway`) de esa dirección.
  - Las direcciones IPv6 se **canonicalizan** (forma comprimida estándar, RFC 5952) antes de persistir, de modo que la **unicidad** de la dirección de gestión sea real (dos formas textuales de la misma IPv6 no se consideran distintas).
  - La **validación** de formato por familia (IPv4/IPv6) y del prefijo es autoritativa en el servidor; el contrato la declara con `format: ipv4`/`ipv6`.
  - La **redacción por rol** (RNF-09/ADR-11) aplica a ambas familias: para el rol Auditor se enmascara la **porción de host** por debajo del prefijo (p. ej. IPv4 `10.0.0.11/24` → `10.0.0.***`; IPv6 `2001:db8:acad:1::11/64` → `2001:db8:acad:1::***`).

### 7.2 `config-backup-service`
- **RF-06:** El sistema debe conectarse vía SSH (Netmiko/NAPALM) a los dispositivos del inventario y respaldar su configuración `running-config`.
- **RF-07:** El sistema debe versionar cada respaldo en un repositorio Git interno, permitiendo consultar el diff entre dos versiones de un mismo dispositivo.
- **RF-08:** El sistema debe permitir programar respaldos automáticos (recurrentes) y ejecutar respaldos bajo demanda.
- **RF-09:** El sistema debe detectar y notificar (evento `config.drift_detected`) cuando la configuración de un dispositivo cambia respecto a su último respaldo conocido.
- **RF-10:** El sistema debe registrar el resultado (éxito/fallo) de cada intento de respaldo, con motivo del fallo si aplica.

### 7.3 `compliance-audit-service`
- **RF-11:** El sistema debe ejecutar Playbooks Ansible que validen políticas de configuración predefinidas (NTP configurado, banner de advertencia presente, SNMP community string no default, SSH versión 2, contraseñas cifradas, protocolos inseguros deshabilitados).
- **RF-12:** El sistema debe generar un hallazgo (finding) por cada política incumplida, indicando dispositivo, política, severidad y evidencia.
- **RF-13:** El sistema debe permitir consultar el histórico de cumplimiento de un dispositivo a lo largo del tiempo.
- **RF-14:** El sistema debe publicar un evento (`compliance.finding_created`) por cada hallazgo nuevo.

### 7.4 `scan-orchestrator-service` (Fase B)
- **RF-15:** El sistema debe permitir disparar escaneos de descubrimiento (Nmap) y de vulnerabilidades (OpenVAS) contra un conjunto de objetivos **configurable por entorno** (rangos IP/CIDR autorizados).
- **RF-16:** El sistema debe poder operar tanto contra redes de producción reales autorizadas explícitamente por su propietario, como contra el entorno de red simulada (GNS3/Packet Tracer, Sección 6.6) usado durante el desarrollo y las pruebas de este proyecto — mismo comportamiento funcional, distinto alcance configurado.
- **RF-17:** El sistema debe rechazar y registrar (sin ejecutar) cualquier solicitud de escaneo cuyo objetivo esté fuera del alcance autorizado configurado para el entorno activo.
- **RF-18:** El sistema debe permitir programar escaneos periódicos y ejecutar escaneos bajo demanda.
- **RF-19:** El sistema debe registrar y exponer el estado de cada escaneo (en cola, en progreso, completado, fallido).

### 7.5 `vulnerability-service` (Fase B)
- **RF-20:** El sistema debe almacenar los hallazgos de vulnerabilidades reportados por `scan-orchestrator-service`.
- **RF-21:** El sistema debe enriquecer cada hallazgo con datos CVE/CVSS obtenidos de la API pública NVD (National Vulnerability Database).
- **RF-22:** El sistema debe calcular y exponer la severidad (crítica/alta/media/baja) de cada hallazgo según su score CVSS.
- **RF-23:** El sistema debe permitir consultar vulnerabilidades filtradas por activo, severidad y estado (abierta/remediada).
- **RF-24:** El sistema debe publicar un evento (`vulnerability.critical_found`) cuando se detecta un hallazgo de severidad crítica.

### 7.6 `remediation-tracking-service` (Fase B)
- **RF-25:** El sistema debe permitir crear un ticket de remediación a partir de un hallazgo de `compliance-audit-service` o de `vulnerability-service`.
- **RF-26:** El sistema debe permitir asignar un ticket a un usuario, priorizarlo y transicionarlo por los estados: `abierto → en progreso → resuelto → verificado`.
- **RF-27:** El sistema debe rechazar transiciones de estado inválidas (máquina de estados explícita) y registrar el motivo del rechazo.
- **RF-28:** El sistema debe mantener un historial auditable de cambios de estado por ticket (quién, cuándo, de qué estado a qué estado).

### 7.7 `alerting-service`
- **RF-29:** El sistema debe evaluar reglas de alerta configurables sobre los eventos entrantes del message broker (`config.drift_detected`, `compliance.finding_created`, `vulnerability.critical_found`, fallos de escaneo).
- **RF-30:** El sistema debe generar una alerta con severidad y contexto (dispositivo, tipo de evento, detalle) cuando una regla se cumple.
- **RF-31:** El sistema debe permitir consultar el historial de alertas generadas, con filtro por severidad y estado (activa/reconocida/cerrada).

### 7.8 `notification-service`
- **RF-32:** El sistema debe enviar notificaciones por correo electrónico y/o Telegram cuando `alerting-service` genera una alerta, según la severidad configurada.
- **RF-33:** El sistema debe usar plantillas de mensaje diferenciadas por tipo de evento.

### 7.9 `reporting-service` (Fase B)
- **RF-34:** El sistema debe generar reportes ejecutivos en PDF con hallazgos de cumplimiento y vulnerabilidades, agrupados por severidad y por dispositivo.
- **RF-35:** El sistema debe permitir generar el reporte bajo demanda y para un rango de fechas específico.

### 7.10 `telemetry-collector-service` (Fase B)
- **RF-36:** El sistema debe recolectar estado/métricas básicas de los dispositivos (disponibilidad, uso de CPU/memoria si el dispositivo lo expone) vía SNMP.
- **RF-37:** El sistema debe almacenar la telemetría con marca de tiempo para consulta histórica.

### 7.11 Frontend (Angular — dashboard)
- **RF-38:** El sistema debe presentar un dashboard con el estado general: total de dispositivos, alertas activas, porcentaje de cumplimiento, vulnerabilidades abiertas por severidad.
- **RF-39:** El sistema debe permitir ver el detalle de un dispositivo: datos del inventario, historial de respaldos, hallazgos de cumplimiento y vulnerabilidades asociadas.
- **RF-40:** El sistema debe actualizar el dashboard en tiempo real (WebSocket/SSE) ante nuevas alertas, sin requerir recarga manual.
- **RF-41:** El sistema debe aplicar control de acceso basado en roles (RBAC) en la interfaz — ocultar acciones no permitidas para el rol activo, además de la validación server-side.
- **RF-42:** El sistema debe permitir descargar el reporte ejecutivo en PDF desde la interfaz.

---

## 8. Requerimientos No Funcionales (RNF)

### 8.1 Rendimiento
- **RNF-01:** Los endpoints de consulta (GET) deben responder en menos de 300 ms (p95) bajo condiciones normales de carga en el entorno de prueba.
- **RNF-02:** Un respaldo de configuración de un solo dispositivo no debe exceder 30 segundos bajo condiciones normales de red.

### 8.2 Seguridad
- **RNF-03:** Autenticación centralizada vía OAuth2/OIDC (Keycloak), con JWT propagado a todos los microservicios; expiración y renovación de tokens documentada.
- **RNF-04:** RBAC de extremo a extremo — enforcement en el API Gateway, en la UI (visibilidad) y en los datos (server-side); nunca ocultar solo en el cliente.
- **RNF-05:** Comunicación cifrada (HTTPS/TLS) entre el cliente y el API Gateway. El tráfico interno entre microservicios se restringe por Security Groups dentro de la VPC de AWS; mTLS queda documentado como mejora futura, no como requisito del MVP.
- **RNF-06:** Los secretos (credenciales SSH de dispositivos, claves de API, credenciales de base de datos) deben externalizarse vía variables de entorno o gestor de secretos; nunca en código ni en el repositorio.
- **RNF-07:** El alcance autorizado de escaneo (rangos IP/CIDR) debe ser **configurable por entorno** y aplicarse como **control técnico obligatorio**, no solo como política: durante desarrollo/pruebas apunta exclusivamente a la red simulada (Sección 6.6); en un despliegue para un cliente real, apunta únicamente a los rangos que ese cliente autorice explícitamente.
- **RNF-08:** Escaneo automatizado de dependencias (SCA) vía Dependabot y análisis en CI (bloqueante en severidad crítica).
- **RNF-09:** Los errores de la API no deben filtrar detalles internos (stack traces, nombres de tablas/clases) al cliente.

### 8.3 Disponibilidad y resiliencia
- **RNF-10:** Cada microservicio debe implementar timeouts, reintentos con backoff y circuit breaker (Resilience4j) en sus llamadas a otros servicios o a dispositivos de red externos.
- **RNF-11:** El sistema debe degradarse con gracia ante la caída de un microservicio no crítico (p. ej., el dashboard debe mostrar "datos no disponibles" en la sección afectada, no fallar por completo).
- **RNF-12:** Cada microservicio debe exponer un endpoint de salud (`/health`) para verificación de disponibilidad.

### 8.4 Escalabilidad
- **RNF-13:** Cada microservicio debe ser stateless y escalable horizontalmente de forma independiente (demostrado con autoscaling básico — HPA — en el despliegue de Kubernetes).
- **RNF-14:** Cada microservicio debe ser dueño exclusivo de su base de datos (database-per-service); ningún otro servicio accede directamente a ella.

### 8.5 Observabilidad
- **RNF-15:** Cada microservicio debe exponer métricas (Prometheus/Micrometer) visualizables en Grafana.
- **RNF-16:** El sistema debe implementar trazabilidad distribuida (OpenTelemetry/Jaeger) con un identificador de correlación propagado entre servicios para una misma solicitud.
- **RNF-17:** Los logs deben ser estructurados y centralizados, sin datos sensibles (PII, secretos) en texto plano.

### 8.6 Mantenibilidad y calidad
- **RNF-18:** Cobertura mínima de pruebas del 70% (statements) por microservicio.
- **RNF-19:** Cada microservicio debe pasar el gatekeeper (build + tests + lint) en CI antes de fusionar cualquier cambio.
- **RNF-20:** Cada microservicio debe contar con documentación obligatoria previa a su codificación (propuesta de módulo, casos de prueba, memoria técnica — taxonomía completa en la Sección 9).
- **RNF-21:** Los contratos de API/eventos entre servicios deben verificarse con contract testing (Pact) en CI antes de publicar una nueva versión.

### 8.7 Portabilidad y despliegue
- **RNF-22:** El sistema debe ejecutarse de forma equivalente en Docker Compose (local) y en Kubernetes (k3s/EKS), sin cambios de código — solo de configuración (principio 12-factor).
- **RNF-23:** La infraestructura de AWS debe definirse como código (Terraform), permitiendo crearla y destruirla de forma reproducible (`terraform apply` / `terraform destroy`).

### 8.8 Costos
- **RNF-24:** El consumo del entorno de prueba en AWS no debe exceder el presupuesto disponible (~MXN $4,958 en 80 días); el entorno se opera "bajo demanda", con AWS Budgets configurado con alertas.

### 8.9 Usabilidad y accesibilidad
- **RNF-25:** El dashboard debe cumplir un contraste mínimo WCAG 2.1 AA, navegación por teclado y `aria-label` en íconos interactivos.
- **RNF-26:** El dashboard debe diferenciar estados vacíos ("sin dispositivos registrados" vs. "sin resultados de búsqueda") y mostrar feedback de carga/error explícito.

### 8.10 Compatibilidad y estándares
- **RNF-27:** Cada microservicio debe documentar su API con OpenAPI/Swagger desde su primera versión funcional.
- **RNF-28:** El versionado del sistema debe seguir SemVer, con CHANGELOG (formato Keep a Changelog) por repositorio.

---

## 9. Organización y taxonomía de documentación del proyecto

Principio rector: **cada documento vive una sola vez, en el nivel
correcto** (global · por repositorio · por microservicio/feature) — nunca
se duplica el mismo contenido en dos niveles.

| Documento | Nivel | Ubicación |
|---|---|---|
| Propuesta de proyecto | Global (1) | `documentos/planificacion/propuesta_proyecto.md` |
| Plan de trabajo global | Global (1) | `documentos/planificacion/plan_trabajo.md` — filas por Fase A/Fase B |
| Diagrama de arquitectura | Global (1) | `documentos/arquitectura/diagrama_arquitectura.md` |
| Memoria técnica global | Global (1) | `documentos/arquitectura/memoria_tecnica_global.md` |
| Especificación técnica (diseño de un tema complejo) | Selectivo — solo temas transversales complejos (p. ej. comunicación por eventos, migración a Kubernetes) | `documentos/arquitectura/especificaciones/<tema>.md` — **no** uno por microservicio |
| Estándares de desarrollo | Global (1), con subsecciones por stack (Java/Spring, Python/FastAPI, Angular) | `documentos/arquitectura/estandares_desarrollo.md` |
| CLAUDE.md | Por repositorio (×3) | `codigo/{backend,frontend,management}/CLAUDE.md` |
| README | Por repositorio (×3) + 1 README raíz que enlaza las 3 capas | `codigo/{backend,frontend,management}/README.md` |
| Índice de documentación | Por repositorio (×3) | `codigo/<repo>/docs/README.md` |
| CHANGELOG | Por repositorio (×3) | `codigo/<repo>/CHANGELOG.md` |
| Plantilla de pull request | Por repositorio (×3) | `codigo/<repo>/.github/pull_request_template.md` |
| SECURITY.md | Por repositorio (×3) | `codigo/<repo>/SECURITY.md` |
| Propuesta de módulo | Por microservicio + por feature de frontend | `codigo/backend/<servicio>/docs/propuesta_modulo.md` |
| Casos de prueba (unit/integración/E2E) | Por microservicio + por feature de frontend | `codigo/backend/<servicio>/docs/casos_de_prueba.md` |
| Memoria técnica de módulo | Por microservicio + por feature de frontend | `codigo/backend/<servicio>/docs/memoria_tecnica.md` |
| Protocolo de verificación en 4 fases | Global (1) — una sola metodología para todo el sistema | `documentos/qa/protocolo_verificacion_4_fases.md` |
| Reporte de QA | Global (1), consolidado con tabla de resultados por servicio | `documentos/qa/reporte_qa.md` |
| Análisis de pruebas | Opcional, por microservicio, solo si hace falta diagnosticar/planear su suite | `codigo/backend/<servicio>/docs/analisis_pruebas.md` |
| Plan de salida a producción | Global (1) — un despliegue = todo el sistema | `documentos/despliegue/plan_salida_produccion.md` |
| Runbook de despliegue y operación/mantenimiento | Global (1) — cubre despliegue, operación rutinaria y diagnóstico de incidencias en un solo documento | `documentos/despliegue/runbook_despliegue.md` |
| Guía rápida de usuario | Global (1) — el usuario final solo interactúa con el dashboard Angular | `documentos/guias/guia_rapida_usuario.md` |
| Estado de sesión / contexto entre sesiones | Global (1), documento operativo vivo | `documentos/sesiones/` |

**Documentos que se resuelven reutilizando uno ya existente** (para no
duplicar contenido): el plan de implementación se cubre con el Plan de
trabajo global (secuencia y fases) más la Sección 7 de cada propuesta de
módulo (pasos concretos del servicio) — no se crea un documento aparte. La
guía de mantenimiento se cubre con las secciones "Operación rutinaria" y
"Diagnóstico de incidencias" del Runbook de despliegue — no se crea un
documento aparte. El diseño transversal se cubre con la Memoria técnica
global más las especificaciones técnicas selectivas — no se crea un
documento "diseño.md" genérico.

**Estructura de carpetas de código dentro de cada microservicio** (ejemplo
para `asset-inventory-service`):
```
codigo/backend/asset-inventory-service/
├── src/                          código fuente
├── docs/
│   ├── propuesta_modulo.md
│   ├── casos_de_prueba.md
│   └── memoria_tecnica.md
├── Dockerfile
└── openapi.yaml                  contrato de API del servicio
```

---

## 10. Restricciones

- **Presupuesto AWS:** MXN $4,958 disponibles, ventana de 80 días (detalle completo en Sección 6.7).
- **Entorno de desarrollo principal:** MacBook Air 2024 (Apple Silicon).
- **Tiempo:** el proyecto compite por horas con el plan de certificación CCNA/Security+ del autor — se prioriza la Fase A como entregable mínimo demostrable; si el tiempo lo exige, la Fase B puede ejecutarse en una etapa posterior sin comprometer las certificaciones.
- **Alcance ético/legal:** durante el desarrollo, pruebas y despliegue de este proyecto, todo escaneo activo se ejecuta exclusivamente contra la red simulada descrita en la Sección 6.6. El sistema tiene la capacidad funcional de operar sobre redes de producción reales (RF-16), pero eso solo ocurriría en un uso posterior, con autorización explícita del propietario de esa red.

---

## 11. Estructura de repositorios

| Repositorio | Ruta local | Contenido |
|---|---|---|
| `backend` | `codigo/backend/` | Monorepo con los 10 microservicios (Java + Python), cada uno en su carpeta |
| `frontend` | `codigo/frontend/` | SPA Angular (dashboard) |
| `management` | `codigo/management/` | Orquestación: `docker-compose.yml`, manifiestos Kubernetes, Terraform, README raíz del sistema |

Los 3 directorios ya existen en disco y están vacíos; `git init` y la
creación de los repositorios remotos en GitHub quedan pendientes de
confirmación explícita antes de ejecutarse.

---

## 12. Criterios de éxito / Definition of Done

**Fase A (mínimo demostrable):**
```
[ ] Los 5 microservicios de Fase A funcionando en Docker Compose.
[ ] Golden path ejecutado de extremo a extremo con evidencia: alta de un
    dispositivo → respaldo de su configuración → auditoría de cumplimiento
    → alerta visible en el dashboard.
[ ] Casos de prueba de cada microservicio en ✅ PASS; cobertura ≥70%.
[ ] Documentación completa por microservicio (propuesta + casos + memoria técnica).
[ ] Despliegue de prueba funcional en AWS (EC2 + RDS), con smoke test post-despliegue.
[ ] Repositorios publicados en GitHub con README, CHANGELOG y CI en verde.
```

**Fase B (extensión):**
```
[ ] Los 5 microservicios de Fase B integrados y funcionando.
[ ] Migración a Kubernetes (k3d local + k3s en EC2) documentada y demostrable.
[ ] Ventana de demostración en Amazon EKS ejecutada y documentada (al menos una vez).
[ ] Reporte de QA consolidado, sistema certificado bajo el Protocolo de 4 fases.
```

---

## 13. Riesgos

| Riesgo | Prob. | Impacto | Mitigación |
|---|---|---|---|
| El alcance total (10 microservicios + AWS + Kubernetes) compite con el tiempo del plan CCNA/Security+ | Alta | Alto | Entrega por Fase A/Fase B; Fase A es el mínimo demostrable y suficiente para portafolio |
| Sobrecosto en AWS por dejar recursos corriendo | Media | Medio | AWS Budgets con alertas + entorno "apagado por defecto" (Sección 6.7) |
| Escaneo de vulnerabilidades apuntado por error fuera del alcance autorizado | Baja | Alto | Control técnico de alcance autorizado configurable, obligatorio por entorno (RNF-07); durante el proyecto, ese alcance es exclusivamente la red simulada (Sección 6.6) |
| Curva de aprendizaje de Kubernetes retrasa la Fase B | Media | Medio | Progresión gradual: Docker Compose → k3d local → k3s en EC2 → EKS (Sección 6.8) |
| Sobrecarga de documentación retrasa la codificación | Media | Medio | Documentación exigida es proporcional al servicio (propuesta breve para servicios simples; especificación técnica solo para lo complejo/transversal — Sección 9) |

---

## 14. Documentos relacionados

Los siguientes documentos son contexto histórico del proceso de diseño —
**no son lectura requerida** para trabajar sobre este proyecto, ya que su
contenido relevante fue incorporado en las secciones anteriores:

- `anteproyecto_microservicios.md` — registro de la exploración y análisis de las 3 opciones evaluadas antes de elegir la Opción 1 ampliada, posteriormente renombrada **redSegura**.
- `especificaciones_diseño.md` — prácticas base heredadas del proyecto Almacenes, origen del kit de plantillas.
- `templates/` — kit de plantillas reutilizables referenciado en la Sección 9.
