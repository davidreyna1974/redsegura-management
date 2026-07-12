# Anteproyecto de Arquitectura de Microservicios
**Autor:** David Reyna Pineda
**Contexto:** Proyecto de portafolio — Network Automation Engineer / Desarrollo de software
**Fecha:** 2026-07-08

---

## 1. Objetivo del documento

Definir una arquitectura de referencia de microservicios, basada en estándares
de la industria (software engineering, ciberseguridad, UI/UX), y proponer tres
dominios de aplicación concretos donde esa arquitectura pueda implementarse
como proyecto de portafolio, dentro del alcance de los recursos tecnológicos
disponibles (MacBook Air 2024 Apple Silicon, Docker, VMware Fusion, GNS3,
Cisco Packet Tracer, stack Spring Boot + Angular + PostgreSQL, Python).

---

## 2. Arquitectura de referencia (aplicable a las 3 opciones)

### 2.1 Principios de diseño

- **Domain-Driven Design (DDD) — bounded contexts:** cada microservicio
  representa un subdominio de negocio con responsabilidad única (Single
  Responsibility a nivel de servicio), evitando microservicios "anémicos"
  que solo hacen CRUD sin lógica de dominio.
- **Database-per-service:** cada microservicio es dueño exclusivo de su
  propio esquema/base de datos (PostgreSQL). Ningún otro servicio accede
  directamente a esa base — solo a través de la API o de eventos. Esto
  evita el acoplamiento típico de un monolito con una sola base de datos
  compartida.
- **12-Factor App:** configuración externa vía variables de entorno,
  logs como stream de eventos (stdout), procesos stateless, dependencias
  declaradas explícitamente (pom.xml/package.json), builds reproducibles.
- **API-first / contract-first:** cada servicio expone un contrato OpenAPI
  (Swagger) definido antes de implementar, permitiendo que frontend y
  backend avancen en paralelo y sirviendo como documentación viva.
- **Event-driven architecture:** la comunicación entre servicios que no
  requiere respuesta síncrona inmediata se hace vía eventos publicados en
  un message broker, no vía llamadas REST encadenadas (evita el
  anti-patrón de "cadena de microservicios" frágil y lenta).
- **Resiliencia por diseño:** cada llamada entre servicios asume que el
  otro servicio puede fallar, estar lento o no responder — timeouts,
  reintentos con backoff, circuit breaker.

### 2.2 Capas de la arquitectura

**a) Frontend — Angular SPA**
- Angular + Angular Material (design system consistente, componentes
  accesibles por defecto — cumple lineamientos WCAG 2.1 AA básicos:
  contraste, navegación por teclado, aria-labels).
- Comunicación en tiempo real con el backend vía WebSocket o Server-Sent
  Events (SSE) para dashboards con datos en vivo (alertas, estado de
  dispositivos, hallazgos de escaneo).
- Principios de UI/UX aplicados: jerarquía visual clara (dashboard →
  detalle → acción), estados de carga/vacío/error explícitos para cada
  vista, feedback inmediato ante acciones del usuario, diseño responsive
  (desktop primero, pero usable en tablet).

**b) API Gateway / Backend-for-Frontend (BFF)**
- Punto único de entrada para el frontend. Responsabilidades: enrutamiento
  a los microservicios internos, autenticación/autorización centralizada,
  rate limiting, agregación de respuestas de varios servicios cuando el
  frontend necesita una vista compuesta.
- Opciones de implementación: Spring Cloud Gateway (se integra
  naturalmente con tu stack Spring Boot) o un gateway dedicado como Kong/
  Traefik.

**c) Service Discovery & Config Server**
- Service Discovery (Eureka o Consul): permite que los microservicios se
  encuentren entre sí dinámicamente sin hardcodear IPs/puertos —
  fundamental cuando los contenedores se reinician o escalan.
- Config Server (Spring Cloud Config): centraliza la configuración de
  todos los servicios en un solo repositorio Git, evitando archivos de
  configuración dispersos y permitiendo cambios sin reconstruir imágenes.

**d) Microservicios de dominio**
- Cada uno con su propio proceso Spring Boot, su propia base de datos
  PostgreSQL, y su propio ciclo de despliegue independiente. El desglose
  específico depende de la opción de dominio elegida (ver Sección 3).

**e) Comunicación asíncrona — Message Broker**
- RabbitMQ (más simple, ideal para empezar) o Kafka (más robusto para
  volúmenes altos de eventos/telemetría — más relevante si el dominio
  elegido genera streams continuos de datos, como logs o métricas de red).
- Patrón publish/subscribe: un servicio publica un evento ("dispositivo
  detectado", "vulnerabilidad crítica encontrada", "configuración
  respaldada") y otros servicios reaccionan sin acoplamiento directo.

**f) Persistencia**
- PostgreSQL como motor principal (ya lo dominas por el proyecto WMS).
- Opcionalmente, un almacén de series de tiempo (TimescaleDB, extensión
  de PostgreSQL) para métricas/telemetría de alta frecuencia, si el
  dominio lo requiere.

**g) Observabilidad — los tres pilares**
- **Métricas:** Prometheus recolectando métricas de cada servicio
  (expuestas vía Spring Boot Actuator + Micrometer), visualizadas en
  Grafana.
- **Logs centralizados:** stack ligero Loki + Promtail (más liviano que
  ELK completo, adecuado para una MacBook Air) o Elastic/Kibana si se
  quiere mostrar experiencia con el stack más usado en la industria.
- **Tracing distribuido:** OpenTelemetry + Jaeger, para seguir una
  solicitud a través de múltiples microservicios y diagnosticar cuellos
  de botella o fallos en cascada.

**h) Seguridad**
- Autenticación centralizada en el API Gateway con OAuth2/OIDC (Keycloak
  como identity provider — gratuito, self-hosted, estándar de facto).
- Propagación de JWT firmado entre servicios; cada microservicio valida
  el token sin necesidad de volver a autenticar contra el usuario.
- Gestión de secretos (credenciales de base de datos, API keys) fuera del
  código fuente — variables de entorno en desarrollo, HashiCorp Vault o
  Docker secrets en un entorno más maduro.
- Resiliencia como control de seguridad: Resilience4j para circuit
  breaker, rate limiting y timeouts — protege contra fallos en cascada y
  abuso de la API.
- Aplicación de OWASP Top 10 / OWASP API Security Top 10 en cada
  microservicio (validación de entradas, autorización a nivel de objeto,
  manejo seguro de errores sin filtrar información interna).

**i) CI/CD y despliegue**
- GitHub Actions: pipeline que construye la imagen Docker de cada
  servicio, corre tests automatizados, y publica la imagen en un
  registry (GitHub Container Registry, gratuito).
- Orquestación: Docker Compose para el entorno de desarrollo/portafolio
  (suficiente y realista para una MacBook Air con recursos limitados);
  Kubernetes (k3d/minikube) como extensión opcional de aprendizaje una
  vez que la versión con Docker Compose esté sólida — evita empezar con
  Kubernetes directamente, que añadiría complejidad innecesaria al
  MVP del portafolio.

### 2.3 Diagrama de capas (referencia textual)

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
   ┌────────────────┼────────────────┬───────────────────┐
   │                │                │                    │
┌──▼──────┐   ┌──────▼─────┐   ┌──────▼─────┐   ┌──────────▼────────┐
│Servicio │   │  Servicio  │   │  Servicio  │   │     Servicio       │
│    A    │   │     B      │   │     C      │   │        N           │
│ +su DB  │   │  +su DB    │   │  +su DB    │   │      +su DB        │
└──┬──────┘   └──────┬─────┘   └──────┬─────┘   └──────────┬────────┘
   │                 │                │                     │
   └─────────────────┴───────┬────────┴─────────────────────┘
                              │
                  ┌───────────▼────────────┐
                  │  Message Broker         │
                  │  (RabbitMQ / Kafka)     │
                  └─────────────────────────┘

     Observabilidad transversal: Prometheus + Grafana (métricas),
     Loki/ELK (logs), Jaeger/OpenTelemetry (tracing) — cada servicio
     expone métricas/logs/trazas hacia este stack.
```

---

## 3. Tres opciones de sistema / área de uso / aplicación específica

Las tres opciones reutilizan **exactamente la misma arquitectura de
referencia** de la Sección 2 — lo único que cambia es el dominio de negocio
(qué hace cada microservicio). Las tres tienen amplio uso comercial actual y
son alcanzables con tus recursos.

---

### Opción 1 — NetOps: Plataforma de Automatización y Monitoreo de Red

**Descripción general:** Sistema que centraliza el inventario, respaldo de
configuraciones, auditoría de cumplimiento y monitoreo de dispositivos de
red (routers, switches), automatizando tareas que hoy se hacen manualmente
dispositivo por dispositivo.

**Microservicios principales:**
| Servicio | Responsabilidad | Base de datos |
|---|---|---|
| `device-inventory-service` | Catálogo de dispositivos, atributos, ubicación | PostgreSQL |
| `config-backup-service` | Conexión SSH (Netmiko/NAPALM) para respaldar configs, versionado en Git | PostgreSQL + Git |
| `compliance-audit-service` | Ejecuta Playbooks Ansible que validan políticas (NTP, banner, SSH v2, contraseñas cifradas) | PostgreSQL |
| `telemetry-collector-service` | Recolecta estado/métricas vía SNMP o streaming telemetry | TimescaleDB |
| `alerting-service` | Evalúa reglas y dispara notificaciones (email/Telegram) | PostgreSQL |
| `notification-service` | Envío efectivo de notificaciones, plantillas | — (stateless) |

**Por qué encaja con tu perfil:** Es la extensión directa de los Servicios
6, 7 y 8 que ya definiste en tu plan de consultoría (backup automático,
auditoría de cumplimiento, monitoreo centralizado) — este anteproyecto los
convierte en un producto de software real en vez de scripts sueltos.

**Factibilidad con tus recursos:** Alta. Puedes simular la red objetivo con
GNS3 o Cisco Packet Tracer en la misma MacBook Air, o usar dispositivos
reales de tu home lab. Los seis microservicios son ligeros y corren
cómodamente vía Docker Compose en Apple Silicon.

**Referencia de mercado:** Categoría equivalente a SolarWinds NCM, Itential,
NetBox, y parcialmente a Cisco DNA Center — mercado de Network Automation/
NetOps en crecimiento constante, exactamente el nicho que identificaste en
tu plan de desarrollo profesional.

---

### Opción 2 — Plataforma de Gestión de Activos TI y Vulnerabilidades

**Descripción general:** Sistema que mantiene un inventario de activos TI
(servidores, dispositivos de red, endpoints), orquesta escaneos de
vulnerabilidades, enriquece los hallazgos con datos de CVE/CVSS, y da
seguimiento al ciclo de remediación — el tipo de plataforma que sustenta
los Servicios 1 y 9 de tu plan de consultoría en ciberseguridad.

**Microservicios principales:**
| Servicio | Responsabilidad | Base de datos |
|---|---|---|
| `asset-inventory-service` | Catálogo de activos, criticidad de negocio | PostgreSQL |
| `scan-orchestrator-service` | Dispara escaneos (Nmap/OpenVAS) vía API, controla frecuencia | PostgreSQL |
| `vulnerability-service` | Almacena hallazgos, enriquece con CVE/CVSS (API pública NVD) | PostgreSQL |
| `remediation-tracking-service` | Flujo de trabajo tipo ticket: asignar, priorizar, cerrar | PostgreSQL |
| `reporting-service` | Genera reportes ejecutivos en PDF (hallazgos por severidad) | — (stateless) |

**Por qué encaja con tu perfil:** Aprovecha tu conocimiento declarado de
OWASP Top 10 y tu interés en AppSec/DevSecOps (Opción D de tu plan). Es un
proyecto que demuestra al mismo tiempo desarrollo de software Y
ciberseguridad — la combinación que identificaste como diferenciador.

**Factibilidad con tus recursos:** Alta. OpenVAS y Nmap corren en
contenedores Docker o en una VM Linux con VMware Fusion; la API de NVD
(National Vulnerability Database) es gratuita y no requiere infraestructura
adicional.

**Referencia de mercado:** Categoría equivalente a Tenable.io/Nessus,
Qualys VMDR, Rapid7 InsightVM — mercado de vulnerability/asset management,
uno de los más demandados en ciberseguridad corporativa y auditorías de
cumplimiento (PCI-DSS, ISO 27001).

---

### Opción 3 — Plataforma de Monitoreo Centralizado y Correlación de Eventos de Seguridad (SIEM ligero)

**Descripción general:** Sistema que ingiere logs y eventos de seguridad
(syslog de dispositivos de red, logs de servidores, eventos de
autenticación), los normaliza, aplica reglas de correlación para detectar
patrones sospechosos (intentos de fuerza bruta, escaneo de puertos,
direcciones MAC fluctuando — como viste en el Capítulo 3 de STP), y genera
alertas — equivalente al Servicio 8 de tu plan de consultoría, pero
construido como producto propio en vez de solo desplegar una herramienta
existente.

**Microservicios principales:**
| Servicio | Responsabilidad | Base de datos / infraestructura |
|---|---|---|
| `log-ingestion-service` | Receptor syslog/Filebeat, primer punto de entrada | Kafka (buffer de ingestión) |
| `normalization-service` | Parsea y normaliza formatos heterogéneos a un esquema común | PostgreSQL |
| `correlation-rule-engine-service` | Evalúa reglas sobre el stream de eventos normalizados | PostgreSQL |
| `alerting-service` | Genera y despacha alertas según severidad | PostgreSQL |
| `dashboard-service` | Agrega datos para las vistas del frontend (BFF especializado) | — (lectura) |

**Por qué encaja con tu perfil:** Se alinea directamente con CompTIA
Security+ (análisis de logs, SIEM, respuesta a incidentes — brechas que ya
identificaste como críticas en tu autodiagnóstico) y con tu experiencia
operativa en infraestructura crítica (CFE), donde la detección temprana de
anomalías es central.

**Factibilidad con tus recursos:** Media-alta. Requiere generar un volumen
sintético de logs (puedes usarlo tu propio home lab/GNS3 como fuente real,
o generadores de logs sintéticos) y es el más exigente en throughput de los
tres — Kafka como backbone de ingestión es justamente el patrón estándar de
la industria para este tipo de sistema (así lo hacen internamente
Elastic Security, Splunk y Wazuh).

**Referencia de mercado:** Categoría equivalente a Wazuh, Elastic Security,
Splunk, IBM QRadar — mercado de SIEM/detección de amenazas, con demanda
creciente por regulaciones de protección de datos y ciberseguridad
corporativa.

---

### 3.4 Decisión confirmada (2026-07-08): Opción 1 ampliada con Gestión de Vulnerabilidades

> **Nota de nomenclatura (2026-07-10):** el sistema resultante de esta
> decisión se nombró de trabajo "NetOps" durante la exploración, y fue
> renombrado definitivamente a **redSegura** — ver
> [`proyecto_microservicios_redsegura.md`](proyecto_microservicios_redsegura.md)
> para la especificación con el nombre definitivo.

**Se elige la Opción 1 (NetOps, ahora redSegura) como base, incorporando la
funcionalidad de gestión de vulnerabilidades de la Opción 2.** La fusión es
arquitectónicamente sólida por tres razones:

1. **Mismo bounded context de inventario.** `device-inventory-service`
   (Opción 1) y `asset-inventory-service` (Opción 2) son, en la práctica,
   el mismo concepto — un dispositivo de red es un activo de TI. Mantenerlos
   separados obligaría a sincronizar dos inventarios duplicados; fusionarlos
   en un único `asset-inventory-service` los convierte en la fuente única de
   verdad (single source of truth) para ambos dominios.
2. **Dominios complementarios, no redundantes.** `compliance-audit-service`
   (¿el dispositivo está bien configurado?) y `vulnerability-service` (¿el
   dispositivo tiene vulnerabilidades conocidas?) son dos formas distintas de
   responder la misma pregunta de fondo: "¿cuál es la postura de seguridad de
   este activo?". Es exactamente el patrón que en la industria se conoce
   como **Continuous Security Posture Management** — categoría más amplia y
   más comercializable que un NetOps aislado o una herramienta de
   vulnerability management aislada.
3. **Reutilización real de servicios transversales.** `alerting-service` y
   `notification-service` no necesitan duplicarse: una alerta de drift de
   configuración y una alerta de vulnerabilidad crítica son, técnicamente,
   el mismo tipo de evento con distinto origen — se procesan por el mismo
   pipeline.

**Microservicios del sistema unificado:**

| Servicio | Origen | Responsabilidad | Base de datos |
|---|---|---|---|
| `asset-inventory-service` | Fusión Opción 1 + 2 | Catálogo único de dispositivos/activos, atributos, criticidad | PostgreSQL |
| `config-backup-service` | Opción 1 | Respaldo de configuraciones (Netmiko/NAPALM), versionado en Git | PostgreSQL + Git |
| `compliance-audit-service` | Opción 1 | Playbooks Ansible que validan políticas de configuración | PostgreSQL |
| `telemetry-collector-service` | Opción 1 | Estado/métricas vía SNMP o streaming telemetry | TimescaleDB |
| `scan-orchestrator-service` | Opción 2 | Dispara escaneos (Nmap/OpenVAS) contra los activos del inventario | PostgreSQL |
| `vulnerability-service` | Opción 2 | Almacena hallazgos, enriquece con CVE/CVSS (API NVD) | PostgreSQL |
| `remediation-tracking-service` | Opción 2 (ampliado) | Flujo tipo ticket para hallazgos de compliance **y** vulnerabilidades | PostgreSQL |
| `reporting-service` | Opción 2 | Reportes ejecutivos en PDF (compliance + vulnerabilidades) | — (stateless) |
| `alerting-service` | Compartido | Evalúa reglas y dispara alertas de cualquier origen | PostgreSQL |
| `notification-service` | Compartido | Envío efectivo de notificaciones (email/Telegram) | — (stateless) |

Total: **10 microservicios** (frente a 6 de la Opción 1 aislada) — ver nota
de fase más abajo para manejar este incremento de alcance.

**Consideración operativa importante:** el `scan-orchestrator-service`
ejecuta escaneos activos (Nmap/OpenVAS). Estos escaneos **no deben
apuntarse a infraestructura real de producción o adyacente a CFE** —
equipos de red antiguos pueden colapsar ante un escaneo agresivo, y
escanear infraestructura ajena sin autorización explícita es un problema
legal/ético, no solo técnico. El entorno de prueba debe ser exclusivamente
el lab propio (GNS3, Packet Tracer, VMs en VMware Fusion, o el entorno de
AWS definido en la Sección 6).

**Manejo del incremento de alcance — entrega por fases:**

- **Fase A (núcleo NetOps, prioridad):** `asset-inventory-service`,
  `config-backup-service`, `compliance-audit-service`, `alerting-service`,
  `notification-service`. Este es el MVP mínimo demostrable y el que más
  alinea con el plan de 6 meses ya definido.
- **Fase B (extensión de vulnerability management):**
  `scan-orchestrator-service`, `vulnerability-service`,
  `remediation-tracking-service`, `reporting-service`,
  `telemetry-collector-service`. Se agrega una vez estable la Fase A, sin
  bloquear la primera demo/portafolio utilizable.

Esta fase B es también un punto natural para reutilizar la ventana de
despliegue en AWS (Sección 6): la Fase A puede demostrarse con Docker
Compose local, y la Fase B es un buen candidato para el despliegue de
prueba en AWS con Kubernetes (Sección 7), ya que para entonces el sistema
tiene más servicios y mejor justifica la orquestación.

---

## 4. Tabla comparativa

| Criterio | Opción 1 — NetOps | Opción 2 — Vulnerability Mgmt | Opción 3 — SIEM ligero |
|---|---|---|---|
| Alineación con tu plan actual | Muy alta (Servicios 6-8) | Alta (Servicios 1, 9 / Opción D) | Alta (Servicio 8 / Security+) |
| Complejidad técnica | Media | Media | Alta (throughput de eventos) |
| Uso de tu stack actual (Spring Boot/Angular/PostgreSQL) | Completo | Completo | Completo + Kafka |
| Diferenciador en el mercado | Muy alto (networking + dev) | Alto (AppSec + dev) | Alto (Security+ + dev) |
| Recursos requeridos | GNS3/Packet Tracer, Docker | Docker, VM Linux, API NVD | Docker, generador de logs, Kafka |
| Esfuerzo de datos de prueba | Bajo (lab de red propio) | Bajo (API NVD + Nmap) | Medio-alto (volumen sintético) |

---

## 5. Restricciones y decisiones adicionales confirmadas (2026-07-08)

- El entorno de desarrollo principal sigue siendo la MacBook Air.
- Se dispone de una cuenta AWS de prueba con saldo de **MXN $4,958** y
  **80 días** restantes — el proyecto debe incluir un **despliegue de
  prueba real en AWS**, no solo diseño teórico.
- El proyecto debe incorporar **Kubernetes**, comenzando con Docker Compose
  y migrando después a Kubernetes.
- El proyecto debe reflejar, a **nivel básico-medio**, las
  funcionalidades/tecnologías/estándares actualmente usados en la
  industria — no solo lo mínimo indispensable para que el sistema funcione.

---

## 6. Estrategia de despliegue en AWS (entorno de prueba con presupuesto limitado)

### 6.1 Presupuesto y restricciones

- Saldo disponible: MXN $4,958 (≈ USD $250–270, dependiendo del tipo de
  cambio vigente — conviene verificarlo al momento de planear los gastos).
- Ventana de tiempo: 80 días.
- Esto alcanza cómodamente para un **despliegue de demostración/prueba**,
  pero **no** para mantener el sistema corriendo 24/7 durante los 80 días
  completos. La clave del diseño es poder **encender y apagar el entorno
  bajo demanda** (para grabar una demo, preparar una entrevista técnica,
  validar un cambio) en lugar de dejarlo corriendo de forma continua.

### 6.2 Componentes AWS recomendados vs. a evitar

| Servicio AWS | Uso recomendado | Costo aproximado | Nota |
|---|---|---|---|
| EC2 (t3.small/t3.medium) | Host de contenedores (Docker Compose o k3s) | Elegible Free Tier si la cuenta tiene menos de 12 meses | Apagar (`stop`, no `terminate`) cuando no se use |
| RDS PostgreSQL (db.t3.micro) | Bases de datos de los microservicios | Elegible Free Tier (750 h/mes, 12 meses) | Alternativa de costo cero: PostgreSQL en contenedor dentro del mismo EC2 |
| S3 | Hosting del build de Angular (sitio estático) + backups | Prácticamente gratuito a esta escala | — |
| ALB (Application Load Balancer) | Enrutamiento HTTPS hacia el API Gateway | ~USD $16–20/mes si corre de forma continua | Usarlo solo durante ventanas de demo, o sustituir por Elastic IP + Nginx en el EC2 mientras se desarrolla |
| ACM (certificados SSL) | HTTPS | Gratuito | — |
| ECR (registro de contenedores) | Almacenar las imágenes Docker generadas por CI/CD | Costo mínimo por almacenamiento | — |
| CloudWatch (básico) | Logs/métricas mínimas de AWS | Nivel gratuito cubre uso bajo | Preferir Prometheus/Grafana propios para no depender del costo de CloudWatch a mayor escala |
| **EKS (control plane administrado)** | — | **USD $0.10/hora ≈ $73/mes si corre continuo** | **Evitar dejarlo corriendo de forma permanente** — ver Sección 7 |
| **NAT Gateway** | — | ~USD $0.045/hora + tráfico procesado | **Evitar** — usar subredes públicas con Security Groups bien definidos; es aceptable para un entorno de portafolio, no sería la práctica recomendada en producción real |
| **MSK (Kafka administrado)** | — | Costoso a esta escala | Usar Kafka/RabbitMQ autoalojado en un contenedor |

### 6.3 Plan de uso del entorno

1. Configurar **AWS Budgets** con alertas en USD $50 / $100 / $180 desde el
   primer día — control de gasto no negociable.
2. Definir todo el entorno como código con **Terraform** (ver Sección 8)
   para poder crearlo (`terraform apply`) y destruirlo (`terraform
   destroy`) en minutos, de forma repetible.
3. Mantener el entorno **apagado por defecto**; encenderlo solo para:
   grabar el video/demo del proyecto, preparar una entrevista técnica, o
   validar un cambio importante.
4. Reservar los últimos 10–15 días de la ventana de 80 días para el
   despliegue final completo (incluyendo, si el presupuesto lo permite, una
   ventana corta con EKS real — ver Sección 7, Etapa 4).

**Respuesta a la pregunta 1: sí, es factible.** El presupuesto y el tiempo
disponibles alcanzan para un despliegue de prueba serio y demostrable en
AWS, siempre que el entorno se trate como "bajo demanda" y no como un
servicio productivo corriendo de forma continua.

---

## 7. Ruta de adopción de Kubernetes (Docker Compose → Kubernetes)

Se confirma el enfoque por etapas que propusiste:

**Etapa 1 — Docker Compose (local, MacBook Air).** MVP funcional de los
microservicios, usado para el desarrollo diario. Es también el entorno que
se usará la mayor parte del tiempo, ya que Kubernetes añade complejidad
operativa que no aporta valor durante el desarrollo activo del código.

**Etapa 2 — Kubernetes local (k3d o minikube, en la misma MacBook Air).**
Una vez estable Docker Compose, se migran los manifiestos a Kubernetes
(Deployments, Services, ConfigMaps, Secrets, Ingress) y se practica el
ciclo completo localmente, sin costo de nube. Aquí es donde realmente se
aprende Kubernetes — la nube solo sirve para demostrar que también se sabe
operarlo ahí.

**Etapa 3 — Kubernetes en AWS, autoadministrado (k3s en EC2).** Para el
despliegue de prueba en AWS, la opción más eficiente en costo es instalar
**k3s** (distribución ligera de Kubernetes, sin cargo de control plane) en
1–2 instancias EC2, en vez de usar EKS directamente. Esto demuestra las
mismas competencias de Kubernetes (manifiestos, autoscaling básico con HPA,
Ingress, Secrets) sin el costo fijo de ~USD $73/mes del control plane
administrado de EKS.

**Etapa 4 (opcional, ventana corta) — Amazon EKS real.** Si se quiere
demostrar específicamente experiencia con el servicio administrado más
usado en la industria (EKS), se puede crear el clúster solo durante una
ventana corta (2–3 días, por ejemplo antes de una entrevista técnica) con
Terraform, y destruirlo inmediatamente después. Costo aproximado de esa
ventana: unos USD $5–10 de control plane más el costo de los nodos —
totalmente dentro del presupuesto si se usa de forma puntual.

**Respuesta a la pregunta 2: sí, es factible**, y la secuencia propuesta
(Docker Compose → Kubernetes) es exactamente la progresión recomendada por
la industria — evita el error común de saltar directo a Kubernetes sin
haber estabilizado primero la arquitectura de microservicios en un entorno
más simple.

---

## 8. Stack de tecnologías/estándares de la industria (nivel básico-medio)

Además de lo ya definido en la Sección 2, para reflejar el estado actual de
la industria a un nivel básico-medio (sin sobre-diseñar el proyecto), se
incorporan las siguientes piezas:

| Categoría | Tecnología/estándar | Nivel objetivo |
|---|---|---|
| Infrastructure as Code | Terraform | Básico-medio: módulos para VPC, EC2/k3s, RDS, S3, IAM |
| CI/CD | GitHub Actions | Básico-medio: build + test + push a ECR + deploy |
| Contenedores | Docker, Docker Compose | Medio (ya se domina parcialmente por el proyecto WMS) |
| Orquestación | Kubernetes (k3s / EKS) | Básico (ver Sección 7) |
| Testing | JUnit 5 + Testcontainers (levanta PostgreSQL/RabbitMQ reales durante los tests) | Básico-medio |
| Documentación de API | OpenAPI/Swagger | Básico |
| Autenticación | OAuth2/OIDC vía Keycloak, JWT | Básico-medio |
| Gestión de secretos | Variables de entorno / Kubernetes Secrets (AWS Secrets Manager opcional, tiene costo) | Básico |
| Observabilidad | Prometheus + Grafana, OpenTelemetry (tracing) | Básico-medio |
| Mensajería | RabbitMQ o Kafka | Básico-medio |

**Explícitamente fuera de alcance** (para no sobre-diseñar un proyecto de
portafolio ni comprometer el presupuesto/tiempo disponibles): service mesh
(Istio/Linkerd), arquitectura multi-región, despliegues blue-green/canary
avanzados, operadores personalizados de Kubernetes. Estos se pueden
mencionar como "extensiones futuras" en el README del proyecto, sin
necesidad de implementarlos para que el portafolio sea sólido.

**Respuesta a la pregunta 3: sí, es factible**, entendiendo "básico-medio"
como saber configurar, operar y justificar cada pieza (no ser experto en
cada una) — que es exactamente el nivel que se evalúa en una entrevista
técnica de Network Automation/DevOps junior-mid.

---

## 9. Recomendación y siguiente paso (actualizado)

Las tres opciones de dominio (Sección 3) siguen siendo válidas y ahora se
diseñan considerando el despliegue en AWS y la ruta a Kubernetes definidas
en las Secciones 6–8.

**Nota de alcance importante:** agregar AWS + Terraform + Kubernetes de
forma seria representa un incremento significativo de trabajo sobre el plan
original de 6 meses (que ya incluye CCNA, Security+ y automatización). Se
recomienda decidir explícitamente si este proyecto de portafolio se ejecuta
dentro del Mes 5, de forma reducida (Docker Compose + 1 despliegue simple
en AWS), o si se extiende como entregable de los meses 7–9, después de las
certificaciones, para no comprometer el objetivo prioritario de aprobar
CCNA y Security+.

Próximo paso sugerido:
1. Elegir una de las 3 opciones de dominio (Sección 3).
2. Definir los contratos de API (OpenAPI) y el modelo de datos por
   servicio.
3. Definir un plan de implementación por sprints: MVP en Docker Compose
   (4–6 semanas) → Kubernetes local (Etapa 2) → despliegue de prueba en
   AWS con k3s en EC2 (Etapa 3) → ventana corta opcional con EKS real
   (Etapa 4), según lo definido en la Sección 6.

---

## 10. Prácticas, documentación y organización del proyecto (2026-07-10)

Análisis de factibilidad de los 6 puntos planteados, basado en el kit de
templates (`documentos/templates/`) y en `documentos/especificaciones_diseño.md`
— ambos documentos propios, destilados del proyecto Almacenes (WMS).

### 10.1 Punto 1 — Cumplir estándares de la industria, sistema funcional, con prueba de deploy y tests

**Factible — sin trabajo adicional de diseño**, ya que las Secciones 2, 6, 7,
8 y 9 de este anteproyecto ya lo cubren (arquitectura de referencia,
despliegue de prueba en AWS, ruta a Kubernetes, stack básico-medio de
industria). Lo que se formaliza aquí es la **metodología de calidad** que ya
usaste y probaste en Almacenes, adaptada a microservicios:

- **Gatekeeper por servicio:** build + tests + lint en verde, en CI, antes de
  fusionar cualquier cambio — igual que en Almacenes, pero ejecutado de forma
  independiente por cada microservicio (ver 10.5, CI por servicio en monorepo).
- **Protocolo de verificación en 4 fases** (Inventario → Corrección →
  Re-ejecución → Certificación) como metodología única de QA para todo el
  sistema — ver 10.3 y 10.5 para su adaptación a "blast radius" entre
  servicios.
- **"Completamente funcional"** se define, para la Fase A (núcleo NetOps),
  como un **golden path** demostrable de extremo a extremo y con evidencia:
  alta de un dispositivo en `asset-inventory-service` → respaldo de su
  configuración (`config-backup-service`) → auditoría de cumplimiento
  (`compliance-audit-service`) → alerta visible en el dashboard Angular —
  ejecutado sobre el despliegue de prueba en AWS (Sección 6), no solo en
  local.
- **Tests de funcionalidad:** casos de prueba por microservicio (categorías
  `SEC/RBAC/CRUD/VAL/FLOW/RN/ERR/CYBER` del kit — `UI/VIS/BSRCH` aplican solo
  al frontend), tests de integración con **Testcontainers** (PostgreSQL/
  RabbitMQ reales, no mocks) y un smoke test E2E con Playwright contra el
  entorno desplegado en AWS.

### 10.2 Punto 2 — Reforzar Python en la arquitectura

**Factible, y arquitectónicamente justificado** (no es un lenguaje "forzado"
sin motivo): se reasignan a Python los microservicios donde es, de hecho, el
estándar de facto de la industria — automatización de red y seguridad —
manteniendo Java/Spring Boot en los servicios de lógica de negocio pesada
(donde ya tienes experiencia por el proyecto WMS). El resultado es un sistema
**poliglota** deliberado, un patrón real y valorado en la industria (empresas
de networking/seguridad casi siempre combinan Java/Go para servicios de
negocio con Python para automatización).

| Microservicio | Lenguaje | Motivo |
|---|---|---|
| `asset-inventory-service` | Java (Spring Boot) | Lógica de negocio, RBAC, relaciones de datos |
| `config-backup-service` | **Python (FastAPI)** | Netmiko/NAPALM son librerías Python — no existe equivalente maduro en Java |
| `compliance-audit-service` | Java o Python | Orquesta Ansible (Python-based); puede quedar en Python para reforzar consistencia con `config-backup-service` |
| `telemetry-collector-service` | **Python (FastAPI)** | SNMP/streaming telemetry con librerías Python maduras (pysnmp) |
| `scan-orchestrator-service` (Fase B) | **Python (FastAPI)** | python-nmap / python-gvm son el estándar para automatizar Nmap/OpenVAS |
| `vulnerability-service` (Fase B) | Java (Spring Boot) | Enriquecimiento CVE/CVSS, lógica de negocio de severidad |
| `remediation-tracking-service` (Fase B) | Java (Spring Boot) | Flujo tipo ticket, máquina de estados, RBAC |
| `reporting-service` (Fase B) | Python o Java | Generación de PDF — ambos ecosistemas son fuertes aquí; Python (WeasyPrint/ReportLab) es una opción sólida para reforzar el lenguaje |
| `alerting-service` / `notification-service` | Java (Spring Boot) | Servicios transversales simples, reutilizan patrones ya dominados |

Stack Python recomendado: **FastAPI** (async, genera OpenAPI nativo — encaja
con el principio API-first de la Sección 2.1) + **pytest** +
**Testcontainers-python** para paridad de metodología de pruebas con el lado
Java.

### 10.3 Puntos 3 y 4 — Documentación pedagógica: taxonomía sin repetición

**Factible.** Principio rector: **cada documento vive una sola vez, en el
nivel correcto** (global · por repositorio · por microservicio/feature), y
se enlaza desde un índice — nunca se duplica el mismo contenido en dos
niveles. La siguiente tabla mapea **cada documento que mencionaste** a su
plantilla de origen (`documentos/templates/`) y a su nivel correcto:

| Documento solicitado | Plantilla origen | Nivel | Ubicación |
|---|---|---|---|
| Propuesta de proyecto | `planificacion/propuesta_proyecto_TEMPLATE.md` | Global (1) | `documentos/planificacion/propuesta_proyecto.md` |
| plan_global | `planificacion/plan_trabajo_TEMPLATE.md` | Global (1) | `documentos/planificacion/plan_trabajo.md` — filas por Fase A/Fase B |
| diagrama_arquitectura | `arquitectura/diagrama_arquitectura_TEMPLATE.md` | Global (1) | `documentos/arquitectura/diagrama_arquitectura.md` |
| memoria_tecnica (global) | `arquitectura/memoria_tecnica_global_TEMPLATE.md` | Global (1) | `documentos/arquitectura/memoria_tecnica_global.md` |
| diseño (transversal) | `arquitectura/especificacion_tecnica_TEMPLATE.md` | Selectivo — solo temas complejos/transversales | `documentos/arquitectura/especificaciones/<tema>.md` (p. ej. comunicación por eventos, migración a Kubernetes) — **no** uno por microservicio |
| Estándares de desarrollo | `contexto/estandares_desarrollo_TEMPLATE.md` | Global (1), con subsecciones por stack (Java/Spring, Python/FastAPI, Angular) | `documentos/arquitectura/estandares_desarrollo.md` |
| CLAUDE.md | `contexto/CLAUDE_TEMPLATE.md` | Por repositorio (×3) | `codigo/{backend,frontend,management}/CLAUDE.md` |
| README | `proyecto/README_TEMPLATE.md` | Por repositorio (×3) + 1 README raíz que enlaza las 3 capas | `codigo/{backend,frontend,management}/README.md` |
| docs_indice | `proyecto/docs_indice_TEMPLATE.md` | Por repositorio (×3) | `codigo/<repo>/docs/README.md` |
| CHANGELOG | `proyecto/CHANGELOG_TEMPLATE.md` | Por repositorio (×3) | `codigo/<repo>/CHANGELOG.md` |
| pull_request template | `proyecto/pull_request_TEMPLATE.md` | Por repositorio (×3) | `codigo/<repo>/.github/pull_request_template.md` |
| SECURITY.md | (referenciado en CLAUDE_TEMPLATE; crear archivo breve propio) | Por repositorio (×3) | `codigo/<repo>/SECURITY.md` |
| plan_modulo | `modulos/propuesta_modulo_TEMPLATE.md` | Por microservicio + por feature de frontend | `codigo/backend/<servicio>/docs/propuesta_modulo.md` |
| casos_pruebas (unit/integración/E2E) | `qa/casos_de_prueba_TEMPLATE.md` | Por microservicio + por feature de frontend | `codigo/backend/<servicio>/docs/casos_de_prueba.md` |
| memoria_tecnica (módulo) | `modulos/memoria_tecnica_modulo_TEMPLATE.md` | Por microservicio + por feature de frontend | `codigo/backend/<servicio>/docs/memoria_tecnica.md` |
| protocolo_qa | `qa/protocolo_verificacion_4_fases_TEMPLATE.md` | Global (1) — una sola metodología | `documentos/qa/protocolo_verificacion_4_fases.md` |
| Reporte de QA | `qa/reporte_qa_TEMPLATE.md` | Global (1), consolidado con tabla de resultados por servicio | `documentos/qa/reporte_qa.md` |
| análisis de pruebas | `qa/analisis_pruebas_TEMPLATE.md` | Opcional, por microservicio, solo si hace falta diagnosticar/planear su suite | `codigo/backend/<servicio>/docs/analisis_pruebas.md` |
| plan_puesta_produccion | `despliegue/plan_salida_produccion_TEMPLATE.md` | Global (1) — un despliegue = todo el sistema | `documentos/despliegue/plan_salida_produccion.md` |
| guia_mantenimiento | `despliegue/runbook_despliegue_TEMPLATE.md` (secciones "Operación rutinaria" y "Diagnóstico de incidencias") | Global (1) — **no** se crea un documento aparte | `documentos/despliegue/runbook_despliegue.md` |
| plan_implementacion | Cubierto por `plan_trabajo` (global) + Sección 7 de `propuesta_modulo` (por servicio) | — | **No** se crea documento nuevo — evita duplicar el plan_global |
| guia_usuario | `proyecto/guia_rapida_usuario_TEMPLATE.md` | Global (1) — el usuario final solo interactúa con el dashboard Angular | `documentos/guias/guia_rapida_usuario.md` |
| Estado de sesión | `sesiones/estado_sesion_activa_TEMPLATE.md` + `contexto_sesion_siguiente_TEMPLATE.md` | Global (1), documento operativo vivo | `documentos/sesiones/` |

**Documentos que mencionaste y que se resuelven reutilizando uno ya
existente** (para cumplir "sin ser repetitivos"): *guia_mantenimiento* →
Runbook de despliegue (no un archivo nuevo); *plan_implementacion* → Plan de
trabajo global + Sección 7 de cada propuesta de módulo; *diseño* → Memoria
técnica global + especificaciones técnicas selectivas.

### 10.4 Punto 5 — Estructura de repositorios Git y ubicación de documentos

**Factible — estructura ya creada, confirmada en disco.** Precisión de
diseño: `codigo/backend` se organiza como **monorepo** que contiene **todos**
los microservicios backend (Java y Python), cada uno en su propia carpeta
(`codigo/backend/asset-inventory-service/`, `codigo/backend/config-backup-service/`,
...) — se prefiere sobre 8-10 repos individuales por ser más manejable para
un proyecto de portafolio de una sola persona, y es también un patrón
legítimo en la industria (monorepo backend + repo frontend separado).
`codigo/management` concentra lo que orquesta el sistema completo:
`docker-compose.yml`, manifiestos de Kubernetes, Terraform, y el README raíz
que enlaza las 3 capas.

Verificación en disco (2026-07-10, actualizado tras el renombrado de
`umbrella` → `management`):
```
codigo/backend      → existe, vacío, sin git inicializado
codigo/frontend     → existe, vacío, sin git inicializado
codigo/management   → existe, vacío, sin git inicializado (renombrado desde "umbrella")
```

`documentos/anteproyecto_microservicios.md` ya está en la ubicación
correcta que indicaste (se verificó idéntico al que se venía editando en
`Documents/Plan de desarrollo/`). **Queda una copia duplicada en esa
ubicación anterior** — se recomienda eliminarla para que no diverja, pero no
se modifica sin tu confirmación explícita.

**Acción pendiente (Fase 0, no ejecutada aún):** `git init` en los 3
directorios + primer commit + creación de los 3 repositorios remotos en
GitHub + `git push`. Se deja pendiente de tu confirmación explícita por
tratarse de una acción que crea recursos en un servicio externo (GitHub).

### 10.5 Punto 6 — Adaptación de `especificaciones_diseño.md` a microservicios

Ese documento fue destilado de Almacenes (monolito) y **termina señalando
explícitamente** que las prácticas propias de microservicios "se recomienda
elaborar [como] un documento complementario". **Esta sección es ese
documento complementario.**

**De sus 12 categorías, 10 aplican directamente sin cambios** (Planeación
previa · Seguridad RBAC/redacción/rate limiting/CORS/SCA · Gestión de código
y versionado · CI/CD del gatekeeper · Calidad y pruebas · Estándares de
código transversales — con la salvedad de "transacciones explícitas", ver
abajo · Datos y persistencia · UX/UI · Documentación — resuelta en 10.3 ·
Gestión y metodología de proceso).

**Lo que ya estaba cubierto en este anteproyecto** (referencia cruzada, sin
repetir el detalle): API Gateway → Sección 2.2(b); descubrimiento de
servicios y configuración distribuida → Sección 2.2(c); patrones de
resiliencia (Resilience4j) → Sección 2.2(h); observabilidad distribuida
(OpenTelemetry/Jaeger) → Sección 2.2(g) y Sección 8.

**Lo que faltaba y se agrega ahora** (los puntos que `especificaciones_diseño.md`
había dejado explícitamente fuera de alcance):

- **Contract testing entre servicios.** Cada microservicio ya publica su
  contrato OpenAPI (Sección 2.1); se agrega verificación automatizada de que
  el contrato no se rompe entre versiones, con **Pact** (agnóstico de
  lenguaje — cubre Java, Python/FastAPI y el consumo desde Angular). Corre
  en el gatekeeper de CI antes de publicar una nueva imagen del servicio.
- **Consistencia eventual (reemplaza "transacciones explícitas" del
  monolito).** Con base de datos por servicio no hay transacciones ACID
  distribuidas. Se documenta en la memoria técnica global qué flujos son
  eventualmente consistentes (p. ej., un hallazgo de `vulnerability-service`
  tarda unos segundos en reflejarse como alerta) y se adopta el patrón
  **Saga coreografiada** (basada en eventos del message broker, sin
  orquestador central) para el flujo multi-servicio de la Fase A/B:
  *escaneo/auditoría completada → hallazgo creado → alerta generada →
  notificación enviada*.
- **Seguridad entre servicios (mTLS).** Se documenta como **extensión
  reconocida pero fuera del nivel básico-medio** (consistente con la
  Sección 8): para el despliegue de prueba, la seguridad inter-servicio se
  resuelve con Security Groups de AWS (solo el API Gateway expuesto
  públicamente; los microservicios solo aceptan tráfico dentro de la VPC) +
  JWT propagado en cada llamada. mTLS queda anotado como mejora futura en el
  roadmap del README, no como entregable del MVP.
- **"Blast radius" extendido a contratos compartidos.** El concepto del
  protocolo de 4 fases se amplía: un cambio en un **contrato de API o de
  evento del message broker** (no solo un "servicio compartido" como en el
  monolito) se trata como cambio **global** → se re-prueban todos los
  servicios consumidores de ese contrato/evento, verificado con el contract
  testing del punto anterior.
- **CI por servicio dentro del monorepo backend.** Al ser `codigo/backend`
  un monorepo con varios microservicios, el pipeline de CI se activa **solo**
  para el/los servicios cuyos archivos cambiaron (path-based triggers en
  GitHub Actions, p. ej. `paths: ['codigo/backend/asset-inventory-service/**']`)
  — evita reconstruir todo el backend en cada commit; es la práctica
  estándar de CI en monorepos de la industria.

### 10.6 Resumen de factibilidad

| # | Punto | Factible | Esfuerzo agregado | Dónde vive |
|---|---|---|---|---|
| 1 | Estándares de industria + deploy + tests | Sí, sin diseño adicional | Bajo (ya cubierto) | Secciones 2, 6-9, 10.1 |
| 2 | Reforzar Python | Sí, y bien justificado | Medio (aprender FastAPI + reorganizar 3-4 servicios) | Sección 10.2 |
| 3-4 | Documentación pedagógica completa, sin repetir | Sí, con taxonomía definida | Alto (documentar cada servicio antes de codificarlo, por diseño) | Sección 10.3 |
| 5 | 3 repos Git + ubicación de documentos | Sí, estructura ya creada | Bajo (falta `git init` + push, pendiente de tu confirmación) | Sección 10.4 |
| 6 | Adaptar especificaciones_diseño.md | Sí | Medio (5 prácticas nuevas de microservicios) | Sección 10.5 |

**Nota de alcance (reitera la de la Sección 9):** los 6 puntos son
individualmente factibles, pero **en conjunto aumentan de forma importante**
el esfuerzo de documentación y configuración frente a un proyecto de
portafolio mínimo. La mitigación ya está en el diseño: documentar **antes**
de codificar cada servicio (principio del propio kit) reparte ese esfuerzo a
lo largo del tiempo en vez de acumularlo al final, y la entrega por Fase A/
Fase B (Sección 3.4) permite tener un sistema demostrable y honesto en su
documentación mucho antes de completar el alcance total.
