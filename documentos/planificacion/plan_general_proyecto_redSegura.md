# Plan General del Proyecto — redSegura

**Documento:** Plan de trabajo general (operativiza la especificación del proyecto)
**Proyecto:** redSegura — Plataforma de Automatización, Monitoreo y Gestión de Vulnerabilidades de Red
**Versión del documento:** 1.2.0 · **Fecha:** 2026-07-11 · **Última actualización:** 2026-07-14
**Autor:** David Reyna Pineda · **Estado del proyecto:** Fundación **completa**; Fase A en curso —
`asset-inventory-service` implementado (110 tests, CI activo), pendiente su certificación QA de 4 fases

---

## 0. Propósito y uso de este documento

Este documento es el **plan rector** del proyecto redSegura. Traduce la especificación
funcional (`../proyecto_microservicios_redsegura.md`, en adelante "la Especificación") en un
**plan de ejecución concreto, secuenciado y verificable**: qué se construye, en qué orden,
con qué dependencias, y bajo qué criterios se considera terminado cada bloque de trabajo.

- **Fuente de verdad de *requerimientos*** (qué debe hacer el sistema): la Especificación
  (RF/RNF, RBAC, arquitectura). Este plan **no redefine** requerimientos; los referencia.
- **Fuente de verdad de *ejecución*** (cómo y cuándo se construye): este documento.
- **Audiencia:** cualquier ingeniero que se incorpore al proyecto debe poder leer este
  documento de forma autónoma y entender el estado, la secuencia y los criterios de calidad,
  sin conocimiento previo. Por eso se incluye un glosario (§1) y descripciones explícitas.
- **Mantenimiento:** es un **documento vivo**. Se actualiza al cerrar cada tarea o hito
  (ver §11 Bitácora) y cuando cambie el alcance o la secuencia. La versión del documento
  (encabezado) sube con cada cambio sustantivo.

---

## 1. Glosario de términos clave

Definiciones operativas de los términos usados en este plan y en el `CLAUDE.md` del
repositorio, para que su significado sea inequívoco.

| Término | Significado en este proyecto |
|---|---|
| **Microservicio** | Servicio autónomo, desplegable de forma independiente, dueño exclusivo de su propia base de datos (*database-per-service*), que representa un subdominio de negocio. |
| **Contrato (API)** | Definición formal, en OpenAPI 3.1, de los endpoints que un servicio expone: rutas, métodos, parámetros, cuerpos de petición/respuesta, códigos de estado y roles autorizados. Es un acuerdo verificable entre el productor (el servicio) y sus consumidores (frontend u otros servicios). |
| **Contrato (evento)** | Esquema JSON versionado de un mensaje que un servicio publica en el message broker (RabbitMQ), incluyendo su *routing key*, campos y semántica. |
| **Contract-first / API-first** | Práctica de **definir y acordar el contrato antes de escribir el código**, de modo que frontend y backend puedan avanzar en paralelo y el contrato sirva como documentación viva y como base para *contract testing*. |
| **Contract testing (Pact)** | Prueba automatizada que verifica que un servicio productor no rompió el contrato que un servicio consumidor espera, **antes** de publicar una nueva versión. |
| **Gatekeeper** | Conjunto de verificaciones obligatorias que todo cambio debe pasar en verde antes de integrarse: **build + tests + lint/análisis estático + cobertura ≥ 70 %**. Se ejecuta por microservicio (no sobre todo el monorepo) y lo corre el CI de forma automática. |
| **Definición de "Done" (DoD)** | Lista cerrada de condiciones **verificables** que deben cumplirse para declarar una unidad de trabajo (endpoint, microservicio, fase) como terminada. Mientras una sola condición no se cumpla, el trabajo **no** está *done*, aunque "parezca" funcionar. Se detalla en §8. |
| **Criterio de éxito / de salida** | Condición objetiva que cierra una fase o hito. A diferencia del DoD (aplica a una unidad de trabajo), el criterio de salida aplica a una **etapa completa** de la hoja de ruta (§9). |
| **Golden path** | El flujo principal de extremo a extremo que demuestra el valor del sistema: *alta de un dispositivo → respaldo de su configuración → auditoría de cumplimiento → alerta visible en el dashboard*. Es el hilo conductor de la Fase A. |
| **Blast radius** | Alcance del impacto de un cambio. **Local**: solo afecta la lógica interna de un microservicio → se reprueba ese servicio. **Global**: afecta un contrato de API/evento compartido, el API Gateway o la seguridad → se reprueban **todos** los servicios consumidores. |
| **RBAC** | *Role-Based Access Control*. Control de acceso por rol. Roles del sistema: **Administrador**, **Operador**, **Auditor/Solo lectura** (matriz de acceso en la Especificación §5). La autorización se valida **en cada microservicio**, no solo en el API Gateway. |
| **RF / RNF** | Requerimiento Funcional / Requerimiento No Funcional, numerados en la Especificación §7 y §8 (p. ej. RF-01, RNF-07). |
| **Soft delete** | Baja lógica: el registro se marca como dado de baja (conserva historial y trazabilidad) en lugar de eliminarse físicamente. |
| **Protocolo de verificación en 4 fases** | Metodología de QA del proyecto: (1) Inventario de bugs sobre código *congelado*, (2) Corrección + gatekeeper, (3) Re-ejecución completa desde cero, (4) Certificación. Una ronda solo es válida si se ejecuta íntegra sobre una versión congelada del código. |

---

## 2. Objetivo de la etapa de Fundación

La **Fundación** es la etapa previa a la construcción de cualquier microservicio. Su
objetivo es dejar el proyecto en condiciones de **empezar a codificar con contexto fuerte,
contratos acordados y control de calidad operativo desde el primer commit**. Se apoya en el
orden que impone el `CLAUDE.md`: *documentación y contratos antes que código*.

**Entregables de la etapa de Fundación:**

1. **Reorganización documental** — consolidar la documentación general bajo el repositorio
   *umbrella* `management` (§3).
2. **Arquitectura global** — memoria técnica global, diagrama de arquitectura, estándares de
   desarrollo y especificación de comunicación por eventos.
3. **Contratos de Fase A** — los 5 contratos OpenAPI de los microservicios de Fase A más el
   catálogo de eventos del message broker (§7).
4. **Estructura del monorepo backend** — POM padre de Maven con la configuración común
   (versiones, plugins de calidad) más la documentación del repositorio (README, CHANGELOG,
   SECURITY, plantilla de PR, Dependabot, esqueleto de CI, índice de documentación).
5. **Control de versiones operativo** — `git init` en los repositorios `backend` y
   `management`, con un *git-hook* que impide commits directos a `main`/`develop`, y la
   creación de los repositorios remotos en GitHub con *branch protection* y CI, al cierre de
   la etapa.

**Fuera del alcance de la Fundación** (se abordan en fases posteriores): código de
microservicios, pruebas, `Dockerfile` reales, contratos de Fase B, infraestructura de
despliegue (Docker Compose, Terraform, Kubernetes) y el frontend Angular.

---

## 3. Reorganización documental

Decisión estructural acordada: la documentación **general del sistema** vive en el
repositorio *umbrella* `management`, y cada repositorio de código aloja su documentación
**propia** en un directorio `documentos/` (en español, de forma consistente en todo el
proyecto).

| Antes | Después | Motivo |
|---|---|---|
| `Proyecto microservicios/documentos/` | `codigo/management/documentos/` | `management` es el repositorio umbrella; centraliza la documentación general del sistema (arquitectura, planificación, QA, despliegue). |
| — | `codigo/backend/documentos/` | Documentación propia del repositorio backend (índice, notas transversales del monorepo). |
| — | `codigo/frontend/documentos/` (a futuro) | Mismo criterio cuando el repositorio frontend tenga contenido. |
| `<servicio>/docs/` (§9 original, en inglés) | `<servicio>/documentos/` | Nomenclatura en español, coherente con el resto del proyecto. Cada microservicio conserva su tríada de documentos (propuesta de módulo, casos de prueba, memoria técnica). |

> **Impacto en `CLAUDE.md`:** al mover `documentos/` dentro de `management`, se actualizan
> en `codigo/backend/CLAUDE.md` las rutas relativas `../../documentos/...` →
> `../management/documentos/...`.

---

## 4. Secuencia de tareas — etapa de Fundación

Las tareas se ejecutan respetando sus pre-requisitos. La columna **Estado** se actualiza en
cada avance (`pendiente` → `en curso` → `done`).

| # | Tarea | Pre-req | Entregable | Estado |
|---|---|---|---|---|
| 0 | Mover `documentos/` a `codigo/management/documentos/` y actualizar rutas en `CLAUDE.md` | — | Reorganización documental (§3) | ✅ done |
| 1 | `git init` en `management`; activar git-hook (`core.hooksPath=.githooks`); crear rama `develop` | 0 | Repositorio `management` versionado | ✅ done |
| 2 | **Plan general del proyecto** (este documento), desde `plan_trabajo_TEMPLATE.md` | 1 | `management/documentos/planificacion/plan_general_proyecto_redSegura.md` | ✅ done |
| 3 | Memoria técnica global (visión, decisiones de arquitectura, RBAC, catálogo de contratos) | 2 | `arquitectura/memoria_tecnica_global.md` | ✅ done |
| 4 | Diagrama de arquitectura (Mermaid: capas, despliegue, flujo de eventos) | 2 | `arquitectura/diagrama_arquitectura.md` | ✅ done |
| 5 | Estándares de desarrollo (Java/Spring, Python/FastAPI, Angular; incluye logging estructurado RNF-17) | 2 | `arquitectura/estandares_desarrollo.md` | ✅ done |
| 6 | Especificación de comunicación por eventos (topología RabbitMQ + esquemas JSON) | 3 | `arquitectura/especificaciones/comunicacion_por_eventos.md` | ✅ done |
| 7 | Protocolo de verificación en 4 fases (metodología de QA global) | 2 | `qa/protocolo_verificacion_4_fases.md` | ✅ done |
| 8 | `git init` en `backend`; git-hook; rama `develop`; documentación del repositorio | 0 | Repositorio `backend` versionado | ✅ done |
| 9 | POM padre del monorepo (Java 21, Spring Boot BOM, Checkstyle, Spotless, JaCoCo, Surefire) | 8 | `codigo/backend/pom.xml` | ✅ done |
| 10 | Contratos OpenAPI de Fase A (5 servicios) | 6 | `codigo/backend/<servicio>/openapi.yaml` (×5) | ✅ done |
| 11 | Documentación del repositorio backend (README, CHANGELOG, SECURITY, PR template, Dependabot, esqueleto CI, índice de docs) | 8 | Archivos base del repositorio | ✅ done |
| 12 | Documentación del repositorio management (README raíz del sistema, CHANGELOG, SECURITY, PR template) | 1 | Archivos base del repositorio | ✅ done |
| 13 | Integrar ramas `feature/*` a `develop` con merge `--no-ff` (ambos repositorios) | 3–12 | Historia git limpia y trazable | ✅ done |
| 14 | Crear repositorios remotos en GitHub, configurar *branch protection* y el gatekeeper de CI | 13 | Repositorios remotos protegidos, con CI | ✅ done |

**Repositorios remotos** (públicos): `github.com/davidreyna1974/redsegura-management` ·
`github.com/davidreyna1974/redsegura-backend`. Branch protection: `main` requiere Pull Request
(aplicado a admins, sin force-push/borrado); `develop` con historial protegido y push permitido.

> **Decisiones añadidas durante la Fundación** (ver §13 y la memoria técnica global): ADR-05
> (contract-first Java / code-first + check Python), ADR-06 (MapStruct / Pydantic para
> entidad↔DTO), ADR-07 (columnas de auditoría estándar).

---

## 5. Modelo de repositorios y ubicación de artefactos

| Repositorio | Ruta | Contenido | Versionado en esta etapa |
|---|---|---|---|
| `management` | `codigo/management/` | Documentación general del sistema (`documentos/`), y a futuro orquestación (docker-compose, Terraform, manifiestos K8s), README raíz | Sí (tarea 1) |
| `backend` | `codigo/backend/` | Monorepo de los 10 microservicios (Java + Python), POM padre, contratos OpenAPI por servicio | Sí (tarea 8) |
| `frontend` | `codigo/frontend/` | SPA Angular (dashboard) | No (fase posterior) |

---

## 6. Convenciones de control de versiones

- **Ramas protegidas:** `main` y `develop`. **Nunca** se commitea directamente en ellas.
- **Flujo de trabajo:** todo cambio se realiza en una rama `feature/…`, `fix/…` o `chore/…`
  y se integra a `develop` mediante `git merge --no-ff` (conserva la traza del merge).
- **Doble capa de protección de ramas:** (1) *git-hook* local `pre-commit` versionado que
  rechaza commits en `main`/`develop`; (2) *branch protection* en GitHub (capa servidor,
  que el hook local no puede sustituir). El remoto se crea al cierre de la Fundación
  precisamente para activar esta segunda capa **antes** de construir microservicios.
- **Mensajes de commit:** *Conventional Commits* (`tipo(scope): mensaje`), con `scope` =
  nombre del microservicio cuando aplique, y el "por qué" en el cuerpo.
- **Versionado:** SemVer por microservicio, con tags prefijados por servicio
  (p. ej. `asset-inventory-service-v0.1.0`). CHANGELOG por repositorio (Keep a Changelog).

---

## 7. Contratos (reforzado)

### 7.1 Qué es un contrato y por qué se define primero

Un **contrato** es el acuerdo formal y verificable de cómo se comunica un servicio con sus
consumidores. En redSegura hay dos tipos:

- **Contrato de API (síncrono):** un archivo **OpenAPI 3.1** (`openapi.yaml`) por servicio
  que describe cada endpoint (ruta, método, parámetros, esquemas de petición/respuesta,
  códigos de estado, seguridad por rol).
- **Contrato de evento (asíncrono):** el **esquema JSON versionado** de cada mensaje que un
  servicio publica en RabbitMQ, con su *routing key* y semántica.

Se definen **antes** del código (*contract-first*) por tres razones: (1) permiten que el
frontend y los servicios consumidores avancen en paralelo; (2) sirven como documentación
viva y precisa; (3) habilitan *contract testing* (Pact) para detectar rupturas antes de
publicar. **Regla del proyecto:** antes de escribir un cliente que consuma otro servicio o un
evento, se verifica el contrato real — nunca se asumen nombres de campos ni estructura.

### 7.2 Reglas transversales de todos los contratos de API

- **Versionado de ruta:** prefijo `/api/v1`. Un cambio incompatible implica `v2`.
- **Autenticación:** *Bearer* JWT (OAuth2/OIDC vía Keycloak). Cada endpoint declara sus
  roles permitidos (`security`), y la autorización se valida **en el propio servicio**.
- **Modelo de error uniforme (RFC 7807/9457, ADR-08):** `application/problem+json` con
  `type`, `title`, `status`, `detail`, `instance` (+ `traceId`), **sin** filtrar detalles
  internos. Códigos: `400`, `401`, `403`, `404`, `409`, `412` (If-Match no coincide),
  `422`, `428` (falta If-Match). `PATCH` = JSON Merge Patch (RFC 7386).
- **Fiabilidad y concurrencia (ADR-09):** `Idempotency-Key` en creaciones; `ETag`/`If-Match`
  (RFC 7232) en mutaciones.
- **Paginación estándar** en toda colección: parámetros `page`, `size`, `sort`; respuesta
  con `content`, `page`, `size`, `totalElements`, `totalPages`.
- **Filtrado en colecciones (criterio general):** todo endpoint que devuelva una lista
  **debe** ofrecer filtro(s) por las **propiedades más características** del objeto listado
  (p. ej. `hostname`, IP de gestión `mgmtIp`, criticidad, severidad, estado, rango de
  fechas), además de la paginación y el ordenamiento. Los filtros se combinan (AND) y se
  omiten los vacíos. La búsqueda de texto es insensible a mayúsculas/acentos.
- **Health probes (ADR-10):** cada servicio expone `GET /health/liveness` y
  `GET /health/readiness` (RNF-12) sin autenticación.
- **Trazabilidad:** se propaga un identificador de correlación (`traceId`) entre servicios (RNF-16).
- **Gobernanza (ADR-12):** estas reglas se verifican en CI con **Spectral** sobre todo
  `openapi.yaml`; un contrato que no cumpla hace fallar el gate.

> **Nota:** los contratos OpenAPI (fuente de verdad, `<servicio>/openapi.yaml`) fueron elevados a
> estándares de industria (ADR-08..12) después de esta sección; `asset-inventory` incorpora además
> identidad estable (`serialNumber`), `deviceType`, ubicación DCIM, bulk import y redacción de
> `mgmtIp` por rol. El detalle vigente vive en cada `openapi.yaml`.

### 7.3 Contratos de API — Fase A (detalle por servicio)

Roles: **ADM** = Administrador, **OPE** = Operador, **AUD** = Auditor. La columna "Roles"
refleja la matriz RBAC de la Especificación §5.

#### `asset-inventory-service` (Java · RF-01…05) — fuente única de verdad del inventario
| Método | Ruta | Descripción | Roles |
|---|---|---|---|
| POST | `/api/v1/devices` | Registrar dispositivo (hostname, IP de gestión, fabricante, modelo, ubicación, criticidad) | ADM |
| GET | `/api/v1/devices` | Listar dispositivos; filtro por `hostname`, `mgmtIp`, `ubicacion`, `criticidad`, `fabricante`, `modelo`, `estado` (activo/baja); paginado y ordenable | ADM, OPE, AUD |
| GET | `/api/v1/devices/{id}` | Consultar detalle de un dispositivo | ADM, OPE, AUD |
| PUT/PATCH | `/api/v1/devices/{id}` | Editar dispositivo | ADM |
| DELETE | `/api/v1/devices/{id}` | Dar de baja (soft delete) | ADM |

Publica eventos `asset.created`, `asset.updated`, `asset.decommissioned` (RF-05).

#### `config-backup-service` (Python · RF-06…10) — respaldo y versionado de configuraciones
| Método | Ruta | Descripción | Roles |
|---|---|---|---|
| POST | `/api/v1/devices/{deviceId}/backups` | Respaldo bajo demanda de **un dispositivo individual** (SSH vía Netmiko/NAPALM); captura `running-config` **y** `startup-config`; devuelve el respaldo creado con el flag `unsavedChanges` | ADM, OPE |
| POST | `/api/v1/backups` | Respaldo **por lotes** (varios/todos): cuerpo `{ "scope": "all" }` \| `{ "deviceIds": [...] }` \| filtro por `hostname`/`mgmtIp`; ejecución asíncrona → devuelve `jobId` | ADM, OPE |
| GET | `/api/v1/backups/jobs/{jobId}` | Estado y avance de un job de respaldo por lotes (encolado/en progreso/completado/fallido, con resultado por dispositivo) | ADM, OPE, AUD |
| GET | `/api/v1/backups` | Historial global de respaldos; filtro por `hostname`, `mgmtIp`, `deviceId`, `unsavedChanges`, rango de fechas y estado; paginado | ADM, OPE, AUD |
| GET | `/api/v1/backups/{id}` | Detalle de un respaldo (`{id}` = id del respaldo): `running-config`, `startup-config`, `unsavedChanges`, resultado | ADM, OPE, AUD |
| GET | `/api/v1/devices/{deviceId}/backups/diff` | Diff textual entre **dos respaldos guardados** del mismo dispositivo (`from`, `to` = referencias de versión; `configType` = running\|startup); ver §7.3.a | ADM, OPE, AUD |
| POST | `/api/v1/devices/{deviceId}/drift-check` | Drift-check de **un dispositivo**: compara `running-config` en vivo vs último respaldo, **sin crear respaldo**; emite `config.drift_detected` si difiere; ver §7.3.a | ADM, OPE |
| POST | `/api/v1/drift-checks` | Drift-check **por lotes** (flota): cuerpo `{ "scope": "all" }` \| `{ "deviceIds": [...] }` \| filtro `hostname`/`mgmtIp`; asíncrono → devuelve `jobId`; emite `config.drift_detected` por cada dispositivo con drift | ADM, OPE |
| GET | `/api/v1/drift-checks/jobs/{jobId}` | Estado y avance de un job de drift-check por lotes (resultado por dispositivo: sin cambios / drift) | ADM, OPE, AUD |
| POST | `/api/v1/schedules` | Programar tareas recurrentes (`type`: `backup` \| `drift-check`) | ADM |
| GET | `/api/v1/schedules` | Listar programaciones; filtro por `type`, `deviceId`/`hostname`/`mgmtIp`, `estado` (activa/pausada) y `frecuencia`; paginado | ADM, OPE, AUD |

Registra el resultado (éxito/fallo, con motivo) de **cada intento** de respaldo, sea
individual o parte de un lote (RF-10). Publica `config.backup_completed`,
`config.backup_failed`, `config.drift_detected` (RF-09) y
`config.unsaved_changes_detected` (cuando `running ≠ startup`; ver ADR-02 y §7.4).

**§7.3.a — running-config vs startup-config, diff y *drift* (config-backup):**

- **Qué se respalda (`running-config` + `startup-config`):** cada respaldo captura **ambas**
  configuraciones del dispositivo y las versiona en el repositorio Git interno (RF-07). El
  `running-config` (activo en RAM) es la **fuente de verdad primaria** exigida por RF-06; el
  `startup-config` (en NVRAM, se carga al arrancar) se captura para detectar divergencias. El
  servicio calcula y expone en cada respaldo el flag **`unsavedChanges = (running ≠ startup)`**,
  que indica que el dispositivo tenía **cambios sin guardar** (`copy running-config
  startup-config` pendiente). Este flag es filtrable en el historial y habilita una futura
  política de cumplimiento ("no debe haber cambios sin guardar"). Modelo del recurso respaldo:
  ```
  Backup { backupId, deviceId, commit, capturedAt,
           runningConfig, startupConfig, unsavedChanges: bool, status, failureReason? }
  ```
- **Diff entre versiones** (`GET /api/v1/devices/{deviceId}/backups/diff`): el servicio versiona cada
  respaldo de la `running-config` en un repositorio Git interno (RF-07). El endpoint recibe
  un `deviceId` y dos referencias de versión (`from`, `to` — identificadores de respaldos
  previos de ese dispositivo) y devuelve la **diferencia línea por línea** entre ambas
  configuraciones (líneas añadidas/eliminadas/modificadas). Compara **dos respaldos ya
  guardados**; sirve para auditar *qué cambió y cuándo*.
- **Detección de *drift*** (evento `config.drift_detected`, RF-09): el servicio obtiene la
  `running-config` **en vivo** del dispositivo y la compara contra su **último respaldo
  conocido**. Si difieren, hubo un cambio fuera del proceso controlado (no autorizado o
  inesperado): **publica** `config.drift_detected` (con `deviceId` y referencia al diff), que
  `alerting-service` consume para, según sus reglas, generar una alerta. El evento se
  **genera automáticamente al detectar** la divergencia (no lo "solicita" ningún consumidor).
  Diferencia con el diff: el drift compara **el dispositivo en vivo contra el último
  respaldo**, no dos respaldos guardados entre sí.
  - **Disparadores de la detección:** (1) **implícito** — durante cada respaldo (programado o
    bajo demanda), al traer el `running-config` en vivo se compara con el respaldo anterior;
    (2) **drift-check dedicado** — compara vivo vs último respaldo **sin crear respaldo**,
    disponible **individual** (`POST /api/v1/devices/{deviceId}/drift-check`), **por lotes de
    flota** (`POST /api/v1/drift-checks`, asíncrono con `jobId`) y **programado**
    (`/schedules` con `type=drift-check`).
  - **Matiz de la línea base:** el drift-check dedicado **no** sobrescribe el último respaldo,
    por lo que permite vigilar cambios no autorizados entre respaldos sin alterar la línea
    base; el respaldo implícito sí guarda una nueva versión (que pasa a ser la nueva línea
    base).

> **Criterio de diseño de rutas (REST):** las operaciones **por-dispositivo** (crear un
> respaldo, comparar versiones) se anclan al recurso padre `devices`
> (`/api/v1/devices/{deviceId}/backups`, `/api/v1/devices/{deviceId}/backups/diff`), porque un
> respaldo pertenece siempre a un dispositivo. Las operaciones **globales / cross-dispositivo**
> (lote de respaldos, historial global, detalle por id global de respaldo) viven en la
> colección de nivel superior `/api/v1/backups`. Así cada `{parámetro}` de ruta tiene un único
> significado y no hay ambigüedad entre `deviceId` e id de respaldo.

#### `compliance-audit-service` (Python · RF-11…14) — auditoría de cumplimiento
| Método | Ruta | Descripción | Roles |
|---|---|---|---|
| POST | `/api/v1/audits` | Disparar auditoría (Playbooks Ansible: NTP, banner, SNMP, SSHv2, cifrado…) | ADM, OPE |
| GET | `/api/v1/findings` | Listar hallazgos; filtro por `hostname`/`mgmtIp`/`deviceId`, `policy`, `severity`, `status` (abierto/resuelto) y rango de fechas; paginado | ADM, OPE, AUD |
| GET | `/api/v1/devices/{deviceId}/compliance-history` | Histórico de cumplimiento del dispositivo; filtro por `policy`, `severity` y rango de fechas; paginado | ADM, OPE, AUD |

Publica `compliance.finding_created` por cada hallazgo nuevo (RF-14).

#### `alerting-service` (Java · RF-29…31) — evaluación de reglas y alertas
| Método | Ruta | Descripción | Roles |
|---|---|---|---|
| POST/PUT/DELETE | `/api/v1/alert-rules` | Gestionar reglas de alerta configurables | ADM |
| GET | `/api/v1/alert-rules` | Listar reglas; filtro por `estado` (activa/inactiva), `severity` y `eventType`; paginado | ADM, OPE, AUD |
| GET | `/api/v1/alerts` | Historial de alertas; filtro por `severity`, `estado` (activa/reconocida/cerrada), `hostname`/`mgmtIp`/`deviceId`, `eventType` y rango de fechas; paginado | ADM, OPE, AUD |
| POST | `/api/v1/alerts/{id}/acknowledge` | Reconocer una alerta | ADM, OPE, AUD *(ver nota)* |

Consume del broker `config.drift_detected`, `compliance.finding_created`,
`vulnerability.critical_found` (Fase B) y fallos de escaneo; genera alertas y publica
`alert.created`.

> **Nota de diseño a resolver:** la matriz §5 marca "Alertas — ver / reconocer" con ✓ para
> los tres roles, pero el rol Auditor se define como solo lectura ("sin poder modificar nada
> ni disparar acciones"). Se registrará como decisión explícita en la memoria técnica global
> si "reconocer" debe excluir a Auditor. Hasta resolverlo, el contrato sigue la matriz.

#### `notification-service` (Java · RF-32…33) — envío efectivo de notificaciones
| Método | Ruta | Descripción | Roles |
|---|---|---|---|
| GET/PUT | `/api/v1/notification-channels` | Configurar canales (email, Telegram) y su asociación por severidad; canales como adaptadores enchufables (ADR-03) | ADM |
| GET/PUT | `/api/v1/notification-templates` | Gestionar plantillas; filtro por `eventType` y `channel` | ADM |
| GET | `/api/v1/notifications` | Historial de envíos; filtro por `channel` (email/Telegram), `status` (enviado/fallido), `severity`, `hostname`/`mgmtIp`/`deviceId`, `alertId` y rango de fechas; paginado | ADM, OPE, AUD |

Servicio principalmente dirigido por eventos: consume `alert.created` y envía la
notificación según la severidad configurada, usando plantillas diferenciadas.

### 7.4 Catálogo de eventos — Fase A

Topología RabbitMQ: un *topic exchange* (`redsegura.events`), *routing keys* jerárquicas
(`<dominio>.<evento>`), una cola por servicio consumidor, y *dead-letter queue* (DLQ) para
mensajes no procesables. Cada evento lleva `eventId`, `eventType`, `occurredAt`,
`traceId`, `version` (SemVer del esquema) y un `payload` tipado.

| Evento (routing key) | Publica | Consume | Payload (campos núcleo) |
|---|---|---|---|
| `asset.created` / `asset.updated` / `asset.decommissioned` | asset-inventory | config-backup, compliance-audit, (Fase B) | `deviceId`, `hostname`, `mgmtIp`, `criticality` |
| `config.backup_completed` | config-backup | alerting | `deviceId`, `backupId`, `commit`, `status` |
| `config.backup_failed` | config-backup | alerting | `deviceId`, `reason` |
| `config.drift_detected` | config-backup | alerting | `deviceId`, `diffRef` |
| `config.unsaved_changes_detected` | config-backup | alerting | `deviceId`, `backupId`, `runningVsStartupDiffRef` |
| `compliance.finding_created` | compliance-audit | alerting, (Fase B: remediation) | `deviceId`, `policy`, `severity`, `evidence` |
| `alert.created` | alerting | notification | `alertId`, `severity`, `deviceId`, `sourceEvent`, `detail` |
| `vulnerability.critical_found` *(Fase B, referencia futura)* | vulnerability | alerting | `deviceId`, `cveId`, `cvssScore` |

### 7.5 Criterios de validez de los contratos (esta etapa)

Un contrato de Fase A se considera **completo y válido** cuando:
1. Pasa el *lint* de OpenAPI sin errores (`@redocly/cli lint` u equivalente).
2. Cubre íntegramente los RF del servicio (trazabilidad RF → operación).
3. Declara roles por operación coherentes con la matriz RBAC §5.
4. Define el modelo de error uniforme, la paginación y `/health` de §7.2.
5. Los eventos que publica/consume están en el catálogo §7.4 con su esquema.

---

## 8. Definición de "Done" y criterios de éxito (reforzado)

### 8.1 Qué significa "Done"

"**Done**" (terminado) **no** significa "el código compila" ni "funciona en mi máquina".
Significa que se cumplen **todas** las condiciones verificables de la lista siguiente. Si una
sola condición falta, el trabajo **no** está *done*, con independencia de que aparente
funcionar. Esta disciplina evita acumular deuda de verificación y "sorpresas" al integrar.

### 8.2 Definición de "Done" por microservicio (Propuesta D del `CLAUDE.md`)

Aplica cuando se construya cada microservicio (Fase A y B). Las cuatro condiciones:

```
[ ] 1. TODOS los casos de prueba del servicio en ✅ PASS.
       → Categorías obligatorias: SEC, RBAC/AUTHZ, CRUD, VAL, FLOW, RN, ERR, CYBER.
       → Ningún endpoint está terminado hasta que todos sus casos pasan.

[ ] 2. Gatekeeper en verde + cobertura ≥ 70 % (statements).
       → build + tests + lint/análisis estático, ejecutado por el CI en entorno limpio.

[ ] 3. Contratos verificados con Pact (si el servicio expone o consume un contrato).
       → Un cambio en un contrato compartido tiene blast radius GLOBAL: se reprueban
         todos los consumidores de ese contrato, no solo el servicio local.

[ ] 4. Documentación del microservicio actualizada.
       → Tríada: propuesta de módulo + casos de prueba + memoria técnica.
```

Complementos obligatorios (del `CLAUDE.md`):
- **Gate de seguridad por endpoint nuevo:** autorización explícita por rol validada en el
  propio servicio; probado el acceso con el rol menos privilegiado (Auditor) que **no** debe
  tener acceso; la autorización **no** se delega al API Gateway.
- **Datos sensibles** (credenciales de dispositivos, secretos) ausentes en logs y en
  respuestas para roles no autorizados.

### 8.3 Definición de "Done" de la etapa de Fundación (esta etapa, sin código)

Como en la Fundación no hay código de negocio, el "Done" aplicable es:

```
[ ] Documentación de arquitectura completa, formal y sin placeholders ni notas de guía.
[ ] Los 5 contratos OpenAPI de Fase A válidos (lint sin errores) y trazables a sus RF (§7.5).
[ ] Catálogo de eventos definido con esquema JSON por evento (§7.4).
[ ] POM padre parsea correctamente (`mvn -N validate` → BUILD SUCCESS).
[ ] git-hook operativo (rechaza commit en main/develop; permite en feature/*).
[ ] Historia git con integración vía merge --no-ff en ambos repositorios.
[ ] Repositorios remotos creados con branch protection y gatekeeper de CI activos.
```

### 8.4 Criterios de éxito del proyecto (Especificación §12)

- **Fase A (mínimo demostrable):** los 5 microservicios de Fase A operando en Docker
  Compose; **golden path** ejecutado de extremo a extremo con evidencia; casos de prueba en
  ✅ PASS y cobertura ≥ 70 %; documentación completa por microservicio; despliegue de prueba
  funcional en AWS (EC2 + RDS) con *smoke test*; repositorios publicados en GitHub con CI en
  verde.
- **Fase B (extensión):** los 5 microservicios de Fase B integrados y funcionando; migración
  a Kubernetes (k3d local + k3s en EC2) documentada; ventana de demostración en Amazon EKS
  ejecutada al menos una vez; reporte de QA consolidado y sistema certificado bajo el
  protocolo de 4 fases.

---

## 9. Hoja de ruta global (reforzado)

Cada etapa se describe con su **objetivo**, sus **entregables** y su **criterio de salida**
(condición objetiva que la cierra). El orden es incremental: cada etapa se apoya en la
anterior.

### Etapa 0 — Fundación *(en curso)*
- **Objetivo:** dejar la base lista para construir microservicios (documentación,
  contratos, monorepo, control de versiones).
- **Entregables:** los de §2.
- **Criterio de salida:** el "Done" de la Fundación (§8.3) cumplido.

### Etapa 1 — Fase A: núcleo redSegura (MVP)
- **Objetivo:** construir los 5 microservicios de Fase A y el dashboard básico, entregando el
  *golden path* funcional en local.
- **Entregables:** `asset-inventory-service` (primero — fuente de verdad; los demás dependen
  de él), luego `config-backup-service`, `compliance-audit-service`, `alerting-service`,
  `notification-service`; dashboard Angular con inventario, estado de respaldo y alertas;
  `docker-compose.yml` en `management`.
- **Criterio de salida:** *golden path* (alta de dispositivo → respaldo → auditoría → alerta
  en dashboard) demostrado de extremo a extremo con evidencia; DoD por servicio cumplido en
  los 5; cobertura ≥ 70 %.

### Etapa 2 — Despliegue de prueba de Fase A en AWS
- **Objetivo:** validar el sistema en un entorno real acotado y de bajo costo.
- **Entregables:** despliegue en EC2 + RDS (o PostgreSQL en contenedor), definido con
  Terraform, operado "bajo demanda"; AWS Budgets con alertas.
- **Criterio de salida:** *smoke test* post-despliegue en verde; entorno reproducible
  (`terraform apply`/`destroy`).

### Etapa 3 — Fase B: gestión de vulnerabilidades
- **Objetivo:** extender el sistema con las capacidades de escaneo, gestión de
  vulnerabilidades, remediación, reportes y telemetría.
- **Entregables:** `scan-orchestrator-service`, `vulnerability-service`,
  `remediation-tracking-service`, `reporting-service`, `telemetry-collector-service`;
  contratos OpenAPI de Fase B; dashboard ampliado (severidad CVSS, reportes PDF).
- **Criterio de salida:** los 5 microservicios integrados y funcionando; enriquecimiento
  CVE/CVSS operativo; reportes ejecutivos PDF generados; DoD por servicio cumplido.

### Etapa 4 — Migración a Kubernetes
- **Objetivo:** demostrar portabilidad y orquestación sin cambios de código (12-factor).
- **Entregables:** manifiestos K8s; progresión Docker Compose → k3d local → k3s en EC2 →
  ventana corta en Amazon EKS; autoscaling básico (HPA), Ingress, Secrets.
- **Criterio de salida:** el sistema corre de forma equivalente en Compose y Kubernetes;
  ventana EKS ejecutada y documentada al menos una vez.

### Etapa 5 — Certificación y cierre
- **Objetivo:** certificar la calidad del sistema completo.
- **Entregables:** reporte de QA consolidado; ejecución íntegra del protocolo de 4 fases;
  documentación de portafolio (READMEs, CHANGELOGs, guías).
- **Criterio de salida:** sistema certificado bajo el protocolo de 4 fases; los 3
  repositorios publicados con documentación completa y CI en verde.

> **Restricción de alcance ético/legal (transversal a todas las etapas):** durante el
> desarrollo, pruebas y despliegue de este proyecto, todo escaneo/automatización apunta
> **exclusivamente** a la red simulada (GNS3/Packet Tracer, Especificación §6.6). El alcance
> autorizado de escaneo es un **control técnico obligatorio** (RNF-07), no solo una política.

---

## 10. Hitos

| Hito | Criterio de cumplimiento | Fecha objetivo |
|---|---|---|
| **M0 — Fundación lista** | "Done" de la Fundación (§8.3): contratos válidos, arquitectura documentada, monorepo y git operativos, remotos con branch protection y CI | por definir |
| **M1 — asset-inventory-service *done*** | DoD por microservicio (§8.2) cumplido: casos de prueba ✅ PASS, gatekeeper verde, cobertura ≥ 70 % | por definir |
| **M2 — Fase A completa (golden path)** | Golden path demostrado de extremo a extremo con evidencia en Docker Compose | por definir |
| **M3 — Despliegue de prueba AWS (Fase A)** | Smoke test post-despliegue en verde (EC2 + RDS) | por definir |
| **M4 — Fase B integrada** | Los 5 microservicios de Fase B funcionando e integrados | por definir |
| **M5 — Kubernetes + demo EKS** | Migración documentada y ventana EKS ejecutada | por definir |
| **M6 — Certificación** | Reporte de QA consolidado; sistema certificado (protocolo de 4 fases) | por definir |

---

## 11. Riesgos y bloqueos actuales

| Riesgo / bloqueo | Prob. | Impacto | Mitigación |
|---|---|---|---|
| El alcance total (10 microservicios + AWS + Kubernetes) compite con el tiempo del plan CCNA/Security+ | Alta | Alto | Entrega por fases; Fase A es el mínimo demostrable y suficiente para portafolio |
| Sobrecosto en AWS por dejar recursos corriendo | Media | Medio | AWS Budgets con alertas + entorno "apagado por defecto" |
| Escaneo apuntado por error fuera del alcance autorizado | Baja | Alto | Control técnico de alcance (RNF-07); durante el proyecto, exclusivamente la red simulada |
| Curva de aprendizaje de Kubernetes retrasa la Fase B | Media | Medio | Progresión gradual: Compose → k3d → k3s → EKS |
| Ambigüedad RBAC en "reconocer alerta" para Auditor | Baja | Bajo | Resolver como decisión explícita en la memoria técnica global (§7.3) |

---

## 12. Bitácora de avance

| Fecha | Avance | Próximo paso |
|---|---|---|
| 2026-07-11 | Análisis de la Especificación, `especificaciones_diseño.md`, `anteproyecto`, `CLAUDE.md` y kit de plantillas. Decisiones de Fundación acordadas (reorganización documental, git en backend + management, contratos de Fase A, remotos al cierre). Creado y reforzado este plan general. Registradas ADR-01 y ADR-02 (§13). | Revisión y autorización del usuario → ejecutar la reorganización documental y la arquitectura global (tareas 0–1, 3–7). |
| 2026-07-12 | Ejecutadas las tareas **0–13**: reorganización documental; `git init` en `management` y `backend` con git-hook anti-commit-directo; arquitectura global (memoria técnica, diagrama, estándares, eventos, protocolo QA); documentación de ambos repos; POM padre (validado); **5 contratos OpenAPI de Fase A** (validados con redocly, 0 errores); merges `--no-ff` a `develop`. Registradas ADR-03..07. Contratos revisados por el usuario. | **Tarea 14 (en pausa):** crear repos remotos en GitHub + branch protection + CI. Luego → construir `asset-inventory-service` (primer microservicio de Fase A). |
| 2026-07-12 | **Tarea 14 completada → FUNDACIÓN CERRADA.** Repos remotos públicos creados (`redsegura-management`, `redsegura-backend`), `main`+`develop` publicados, branch protection configurada, workflow de contract-tests en verde. | **Construir `asset-inventory-service`** (primer microservicio de Fase A): propuesta de módulo + casos de prueba (pre-código) → scaffolding → implementación → gatekeeper ≥70%. |

---

## 13. Registro de decisiones de arquitectura (ADR)

> Decisiones de diseño tomadas durante la planificación/revisión. Se **trasladarán a
> `arquitectura/memoria_tecnica_global.md`** cuando se construya (tarea 3); aquí quedan
> registradas para no perderlas.

### ADR-01 — Almacenamiento de respaldos de configuración: Git (contenido) + PostgreSQL (metadatos)
- **Contexto:** `config-backup-service` debe versionar cada respaldo y permitir consultar el
  diff entre versiones (RF-07), además de listar/filtrar/paginar el historial de forma
  eficiente.
- **Decisión:** esquema **híbrido**. El **contenido** de las configuraciones (`running-config`
  y `startup-config`) se guarda como archivos versionados en un **repositorio Git interno**
  del servicio (un commit por respaldo) — Git aporta versionado, historial y `diff` nativos.
  Los **metadatos** de cada respaldo (`backupId`, `deviceId`, `capturedAt`, `status`,
  `failureReason`, `unsavedChanges`, y el `commit` que apunta al contenido en Git) se guardan
  en la **base de datos PostgreSQL** del servicio, que habilita consulta/filtro/paginación.
- **"Repositorio interno":** repositorio Git propiedad exclusiva del `config-backup-service`
  (principio *database-per-service* aplicado también a su almacén Git), interno al despliegue
  (volumen persistente o servidor Git ligero) — **no** es GitHub ni los repos de código fuente
  del proyecto. Las configuraciones son datos operativos, no código fuente.
- **Alternativas descartadas:** (a) todo en base de datos → reimplementar a mano el
  versionado/diff que Git ya resuelve, y contradice RF-07; (b) todo en Git → Git es
  inadecuado para consultas estructuradas (filtros, paginación, joins con el inventario).
- **Consecuencias:** el flujo de respaldo es: leer configs por SSH → `commit` en Git →
  insertar metadatos en PostgreSQL con el `commit`. Un `GET /backups` consulta PostgreSQL; un
  `diff` lee el contenido desde Git por los `commit` de ambas versiones. Referencia de
  industria: Oxidized/RANCID.

### ADR-02 — Respaldo de `running-config` y `startup-config` con flag `unsavedChanges`
- **Contexto:** en dispositivos de red, `running-config` (RAM, activo) y `startup-config`
  (NVRAM, arranque) pueden diferir; esa divergencia indica **cambios sin guardar**.
- **Decisión:** cada respaldo captura **ambas** configuraciones (versionadas en Git, ADR-01).
  El `running-config` es la fuente de verdad primaria (RF-06). El servicio calcula
  `unsavedChanges = (running ≠ startup)` y lo expone en el recurso `Backup`, **filtrable** en
  el historial (`GET /api/v1/backups?unsavedChanges=true`).
- **Escalamiento proactivo (no solo consulta):** al detectarse `running ≠ startup` durante un
  respaldo, `config-backup-service` **publica el evento** `config.unsaved_changes_detected`
  (§7.4). `alerting-service` lo evalúa con una **regla configurable** (RF-29) y genera una
  alerta; `notification-service` avisa. El servicio de respaldo publica el *hecho*; la
  **severidad y la política de notificación las decide la regla en `alerting-service`** (no se
  cablean en `config-backup`), por consistencia y para evitar fatiga de alertas (la regla puede
  exigir persistencia en respaldos consecutivos, ajustar severidad, etc.). Regla por defecto
  sugerida: **severidad alta**.
- **Consecuencias:** habilita una futura política de cumplimiento en `compliance-audit-service`
  ("no debe haber cambios sin guardar"). El endpoint de diff acepta `configType=running|startup`.

### ADR-03 — Arquitectura de canales de notificación enchufable (patrón adapter/strategy)
- **Contexto:** RF-32 exige notificar por **email y/o Telegram**; es previsible la necesidad
  futura de otros canales (Slack, Teams, webhook, PagerDuty, SMS). Email es el estándar base;
  Telegram es una elección pragmática (no el estándar empresarial, que suele ser
  ChatOps + on-call).
- **Decisión:** `notification-service` define una interfaz **`NotificationChannel`**; cada
  canal es un **adaptador** independiente. Canales MVP: **EmailAdapter** y **TelegramAdapter**
  (cumplen RF-32). La elección de canal según severidad es **configuración**
  (`notification-channels` + `notification-templates`), no lógica cableada. Añadir un canal
  futuro = escribir un adaptador nuevo, **sin modificar el núcleo**.
- **Alternativas descartadas:** cablear los canales con condicionales en el servicio (rígido,
  no extensible); adoptar canales empresariales (Slack/Teams/PagerDuty) desde ya (excede
  RF-32 y añade superficie de integración/credenciales innecesaria para el MVP).
- **Consecuencias:** extensibilidad inmediata (un **adaptador Webhook genérico** habilitaría
  Slack/Teams/PagerDuty vía *incoming webhooks* con bajo costo cuando se requiera), sin deuda
  de refactor. Cumple RF-32 con el mínimo alcance.
