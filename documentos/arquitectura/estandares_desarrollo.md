# Estándares de desarrollo — redSegura

> Convenciones que se asumen en **todo** el código. Reglas accionables, no ensayos.
> Contexto y decisiones: [`memoria_tecnica_global.md`](memoria_tecnica_global.md).

**Última actualización:** 2026-07-11 · **Estado:** en desarrollo (Fundación)

---

## 1. Principios generales

- **Legibilidad sobre astucia.** El código nuevo se parece al que lo rodea (nombres, idioma,
  densidad de comentarios).
- **Una fuente de verdad.** No duplicar lógica/constantes; configuración centralizada (Config
  Server / variables de entorno).
- **Fallar pronto y claro.** Validar entradas en el borde (Controller/Router); errores con
  mensaje útil y sin filtrar internos.
- **Seguro por defecto.** Lo no autorizado se niega; RBAC validado en **cada** microservicio;
  los secretos nunca se hardcodean ni se loguean.
- **Pequeño y testeable.** Una responsabilidad por unidad; dependencias inyectables.
- **Contract-first.** El `openapi.yaml` del servicio se acuerda antes de codificar; el código
  cumple el contrato, no al revés.
- **Resiliencia por diseño.** Toda llamada saliente asume fallo posible: timeout, reintento con
  backoff y circuit breaker (Resilience4j en Java; equivalente en Python).

---

## 2. Nomenclatura y estructura

| Elemento | Convención | Ejemplo |
|---|---|---|
| Carpetas / archivos de servicio | kebab-case | `asset-inventory-service/` |
| Ramas git | `feature/` · `fix/` · `chore/` | `feature/asset-inventory-crud` |
| Commits | Conventional Commits `tipo(scope): msg` | `feat(asset-inventory): alta de dispositivo` |
| Eventos (routing key) | `dominio.evento` (snake para el evento) | `config.drift_detected` |
| Endpoints REST | `/api/v1/recurso` en kebab/plural | `/api/v1/alert-rules` |

> El `scope` del commit = nombre del microservicio cuando aplique. El "por qué" va en el cuerpo.
> Nunca commit directo a `main`/`develop` (git-hook `pre-commit` + branch protection).

Estructura de cada microservicio (Especificación §9):
```
codigo/backend/<servicio>/
├── src/                 código fuente
├── documentos/          propuesta_modulo.md · casos_de_prueba.md · memoria_tecnica.md
├── Dockerfile
└── openapi.yaml         contrato de API del servicio
```

---

## 3. Estilo y herramientas por stack

### 3.1 Java 21 / Spring Boot 3.x (servicios de lógica de negocio)
- **Build:** Maven, heredando del POM padre del monorepo.
- **Formato:** **Spotless** (`google-java-format`); el formato no se discute en review.
- **Lint / estilo:** **Checkstyle**; 0 errores en el gate.
- **Cobertura:** **JaCoCo** ≥ 70 % statements (falla el build si no se alcanza).
- **Diseño:** inyección **por constructor** (nunca `@Autowired` en campos); DTO ↔ entidad
  explícito (nunca exponer entidades JPA); `@Transactional` en la capa de servicio; validación
  con Bean Validation (`@Valid`, `@NotNull`…).
- **Contrato (contract-first, ADR-05):** el `openapi.yaml` es la **entrada**; se generan las
  interfaces de API + DTOs con `openapi-generator-maven-plugin` y el controlador **implementa**
  la interfaz generada. El código cumple el contrato por construcción.
- **Mapeo entidad ↔ DTO (ADR-06):** **MapStruct** (generación en tiempo de compilación,
  type-safe, sin reflexión), configurado en el POM padre (`dependencyManagement` +
  `annotationProcessorPaths` del compiler plugin). Nunca mapear a mano ni exponer entidades.

### 3.2 Python 3.12 / FastAPI (servicios de automatización de red y seguridad)
- **Gestión de paquetes:** `pip`/`uv` con dependencias fijadas.
- **Lint + formato:** **ruff** (`ruff check` + `ruff format`); 0 errores en el gate.
- **Tipos:** **mypy** en modo estricto; sin `# type: ignore` sin justificación.
- **Diseño:** DTOs con **Pydantic v2**; inyección de dependencias con `Depends`; capa
  `router → service → repository`; nunca exponer modelos ORM directamente.
- **Contrato (code-first + check de divergencia, ADR-05):** FastAPI genera su OpenAPI **desde el
  código**; el `openapi.yaml` comprometido es el contrato de diseño. En CI se compara el OpenAPI
  que produce el servicio (`app.openapi()`) contra el `openapi.yaml`; si divergen, el gate
  **falla**. Los modelos Pydantic pueden sembrarse desde el YAML con `datamodel-code-generator`.
- **Mapeo ORM ↔ DTO (ADR-06):** **no** se usa una librería tipo MapStruct; se usa **Pydantic**:
  entidad→DTO con `model_config = ConfigDict(from_attributes=True)` (`DTO.model_validate(orm)`),
  y DTO→entidad construyendo el objeto ORM con `dto.model_dump()`.

### 3.3 Angular (dashboard — repo `frontend`)
- **Lint + formato:** **ESLint + Prettier**; TypeScript en modo **strict**.
- **Diseño:** componentes **contenedor** (lógica) vs **presentacionales** (reutilizables);
  **formularios reactivos** con validación centralizada; limpieza de suscripciones RxJS
  (`takeUntilDestroyed`); los servicios de datos retornan observables (no se suscriben dentro).

> Detalle de convenciones de nomenclatura por lenguaje: Java (`PascalCase` tipos, `camelCase`
> miembros, `UPPER_SNAKE` constantes, paquetes `com.redsegura.<servicio>`); Python (`snake_case`
> módulos/funciones, `PascalCase` clases, PEP 8); Angular (archivos kebab-case, componentes
> `PascalCase`).

---

## 4. Manejo de errores

- **Modelo uniforme (RFC 7807/9457 Problem Details, ADR-08):** respuestas de error con
  `Content-Type: application/problem+json` y campos `type`, `title`, `status`, `detail`,
  `instance` (+ `traceId`). **Nunca** filtrar stack traces ni nombres de tablas/clases al cliente.
- **Códigos:** 400 (malformada) · 401 (no autenticado) · 403 (autenticado sin permiso) ·
  404 (no existe) · 409 (conflicto de estado) · 422 (validación de negocio).
- **Centralización:** Java → `@RestControllerAdvice` global; Python → *exception handlers* de
  FastAPI. Excepciones **tipadas** de negocio (nunca genéricas); nunca tragar excepciones en
  silencio.
- **Distinción 401 vs 403** coherente en todos los servicios.

---

## 5. Configuración y secretos

- Toda config sensible vía **variables de entorno** (12-factor); nunca en el código ni en el repo.
- Cada servicio provee un **`.env.example`** con las claves (sin valores) y la descripción de cada una.
- Secretos reales fuera del control de versiones (RNF-06); rotación inmediata si se exponen.
- El **alcance autorizado de escaneo** (`SCAN_AUTHORIZED_CIDRS`) es configuración obligatoria y
  control técnico (RNF-07): durante el proyecto apunta solo a la red simulada.

---

## 6. Logging y observabilidad (RNF-15 a RNF-17)

**Logging estructurado — obligatorio desde el primer commit de cada microservicio (RNF-17):**
- **Formato JSON estructurado** (una línea = un evento), no texto libre.
- Campos mínimos: `timestamp`, `level`, `service`, `traceId`, `message` (+ `spanId`, `userId`/
  `role` cuando aplique). El **`traceId` se propaga entre servicios** para correlacionar una
  misma solicitud (RNF-16).
- Niveles consistentes (`debug/info/warn/error`). **Nunca** PII, secretos, credenciales de
  dispositivos ni configuraciones sensibles en los logs.
- **Centralización:** logs enviados a **Loki** (vía Promtail); no se escriben archivos locales
  como fuente primaria. **Rotación + retención** definidas en producción (no llenar disco).
- **Implementación:** Java → Logback con encoder JSON (`logstash-logback-encoder`) + Micrometer
  Tracing (OpenTelemetry). Python → logging con formateador JSON (p. ej. `structlog`) +
  instrumentación OpenTelemetry.

**Métricas y trazas:**
- Métricas expuestas para **Prometheus** (Micrometer en Java; cliente Prometheus/OTel en
  Python), visualizadas en Grafana (RNF-15).
- Trazabilidad distribuida con **OpenTelemetry → Jaeger** (RNF-16).
- `GET /health` en cada servicio (RNF-12).
- **Graceful shutdown** en servicios de larga vida (drenar peticiones en vuelo; el orquestador
  espera más que el timeout de apagado antes del SIGKILL).

---

## 7. Estándar de búsqueda de texto

- Búsquedas insensibles a **mayúsculas y acentos** (normalizar / `unaccent` en PostgreSQL), no
  solo a mayúsculas. *(Lección: `LOWER()` quita mayúsculas pero NO acentos.)*
- En cliente: `debounce` ~300 ms; búsqueda vacía = **omitir** el filtro (no enviar cadena vacía).
- Reset del paginador al cambiar filtros, en un punto único.

---

## 8. Accesibilidad / UX (dashboard Angular)

- Contraste mínimo **WCAG 2.1 AA**; navegación por teclado; `aria-label` en íconos interactivos
  (RNF-25).
- **Estados vacíos diferenciados**: "sin dispositivos registrados" vs "sin resultados de
  búsqueda" (RNF-26); feedback de carga/éxito/error explícito.
- **RBAC en la UI:** ocultar acciones no permitidas para el rol activo — **además** de la
  validación server-side, nunca en su lugar (RNF-04).
- Confirmación para acciones destructivas; botón de guardar deshabilitado si el formulario es
  inválido, no modificado o está cargando.

---

## 9. Pruebas (RNF-18)

- **Cobertura mínima 70 % statements** por microservicio; seguridad y reglas de negocio con
  prioridad.
- **Taxonomía:** unit (JUnit 5 / pytest) · integración con **Testcontainers** (PostgreSQL/
  RabbitMQ reales, sin mocks) · **contract testing (Pact)** para contratos compartidos · E2E
  (Playwright, repo frontend).
- Tests **deterministas** (sin dependencia de orden/tiempo real); datos de prueba prefijados y
  limpiados; usuarios de prueba permanentes por rol.
- **Pruebas de seguridad server-side por rol:** verificar enforcement de RBAC y redacción de
  campos con el rol **menos** privilegiado (Auditor).
- Cada bug corregido nace con un **test de regresión** que lo reproduce.

---

## 10. Documentación

- Cada módulo, **antes** de codificar: propuesta + casos de prueba + memoria técnica (ver
  `CLAUDE.md` y §9 de la Especificación).
- Decisiones transversales → memoria técnica global; lecciones → registro de lecciones (abajo).
- OpenAPI desde la primera versión funcional (RNF-27); README, CHANGELOG y diagrama al día como
  parte de "done".

**Documentación en el código (obligatoria):**
- **Toda clase/tipo** y **todo método público significativo** llevan Javadoc (Java) / docstring
  (Python) que explica su **propósito e intención** (el *porqué*), no solo el *qué*.
- Todo **comportamiento no obvio** (algoritmos, máquinas de estado, casos borde, efectos
  secundarios, decisiones de diseño locales) se comenta en el punto donde ocurre.
- **No** se documenta lo trivial (getters/setters, `main`, DTOs autogenerados, tests de nombre
  autoexplicativo). La relajación del Javadoc-obligatorio en Checkstyle es para evitar **ruido**,
  no para omitir lo significativo.
- Los **tests referencian el ID del caso de prueba** que cubren (trazabilidad casos↔código).
- Referencias cruzadas al ADR/RN correspondiente cuando el código materializa una decisión
  (p. ej. `// RN8: ETag/If-Match` o `(ADR-04)`).

---

## 11. Datos y persistencia

**Columnas de auditoría estándar (patrón único, ADR-07).** Toda entidad persistida lleva
columnas de auditoría **auto-pobladas** (nunca a mano):

| Columna | Significado |
|---|---|
| `created_at` | Marca de tiempo de creación (UTC) |
| `created_by` | Usuario que creó el registro (del JWT) |
| `updated_at` | Marca de tiempo de la última modificación |
| `updated_by` | Usuario de la última modificación |

- **Entidades modificables por el usuario** (p. ej. `Device`, `Schedule`, `AlertRule`,
  `Template`, `ChannelConfig`, tickets de remediación) → **las cuatro** columnas.
- **Registros generados por el sistema / inmutables** (p. ej. `Alert`, `Notification`,
  `Finding`) → se rastrean por su **propia marca de tiempo** y por el **evento/regla origen**;
  no llevan `updated_*` (no se editan). Los que un usuario dispara (`Backup`, `Audit`) llevan
  `created_by` = usuario que los originó (o el planificador, si es programado).
- **Auto-población:** Java → **JPA Auditing** (`@CreatedDate`, `@LastModifiedDate`,
  `@CreatedBy`, `@LastModifiedBy` en una `@MappedSuperclass` base + `@EnableJpaAuditing` +
  `AuditorAware` que lee el usuario del JWT). Python → SQLAlchemy con `server_default=now()` /
  `onupdate=now()` para las fechas y la capa de servicio (o un event listener) para `*_by`
  desde el usuario autenticado.
- **Exposición:** `created_at`/`updated_at` como `readOnly` en las respuestas; `created_by`/
  `updated_by` visibles para roles autorizados.

**Otras reglas de persistencia** (prácticas heredadas):
- **Campos de solo lectura protegidos:** no editables directamente vía API (p. ej. `status` de
  un dispositivo cambia por baja lógica, no por edición directa).
- **Migraciones versionadas:** Java → **Flyway**; Python → **Alembic**. El esquema evoluciona
  solo por migraciones versionadas (nunca a mano).
- **Integridad referencial y reglas de negocio en la capa de servicio**, no solo en la BD.
- **Datos de referencia sembrados de forma idempotente** por la aplicación, no por scripts manuales.

---

## 12. Contrato HTTP y seguridad de datos (estándares de industria)

**Errores y actualización (ADR-08):**
- Errores en formato **RFC 7807/9457** (`application/problem+json`) — ver §4.
- **`PATCH` = JSON Merge Patch (RFC 7386)** (`application/merge-patch+json`) para edición parcial.

**Fiabilidad y concurrencia (ADR-09):**
- **Idempotencia:** las creaciones (`POST`) aceptan la cabecera **`Idempotency-Key`**; el servicio
  deduplica reintentos (misma clave → misma respuesta, no crea duplicados).
- **Concurrencia optimista sobre HTTP:** las respuestas de lectura devuelven **`ETag`**; las
  mutaciones (`PUT`/`PATCH`/`DELETE`) exigen **`If-Match`** (RFC 7232) → si el ETag no coincide,
  **412 Precondition Failed**. Mapea el bloqueo optimista (`@Version`) al protocolo HTTP y evita
  *lost updates*.

**Health probes (ADR-10):** cada servicio expone **`/health/liveness`** (¿el proceso vive?),
**`/health/readiness`** (¿dependencias BD/broker OK, listo para tráfico?) y *startup*. Java →
Spring Actuator *health groups*; Python → endpoints equivalentes. Kubernetes no enruta tráfico
hasta *readiness*.

**Seguridad de datos (ADR-11):**
- **Redacción de campos sensibles por rol (server-side):** matriz campo×rol aplicada en el
  servidor, **nunca** solo ocultando en el cliente. El servicio devuelve el valor enmascarado a
  los roles no autorizados (no el valor real "oculto por CSS").
- **Log de auditoría de seguridad (OWASP A09):** registrar en log estructurado (marca `security`)
  los accesos **denegados** (403), autenticaciones fallidas (401) y las mutaciones con su actor.
  Distinto del log de aplicación y de las columnas de auditoría de datos (ADR-07).

**Gobernanza de contratos (ADR-12):** los estándares OpenAPI anteriores están codificados como
reglas en `codigo/backend/.spectral.yaml` (Spectral) y se **verifican en CI** sobre todo
`openapi.yaml` (workflow `contract-governance`). Un contrato — de Fase A o Fase B — que no cumpla
(p. ej. sin `Problem`/RFC 7807, sin probes, con `ApiError`) **hace fallar el gate**. La consistencia
entre los 10 microservicios se garantiza por enforcement automático, no por disciplina.

---

## 📒 Registro de lecciones (vivo)

| ID | Lección (regla a futuro) | Origen | Alcance |
|---|---|---|---|
| L01 | Definir si "reconocer alerta" excluye al rol Auditor antes de implementar `alerting-service` | Revisión de contratos Fase A | global |
| L02 | Capturar `running-config` **y** `startup-config` y escalar su divergencia (`unsavedChanges`) | ADR-02 | global |
| L03 | Búsqueda de texto insensible a acentos, no solo a mayúsculas (`unaccent`, no solo `LOWER()`) | Práctica heredada (Almacenes) | global |
| L04 | Toda entidad persistida lleva columnas de auditoría auto-pobladas (`created_at/by`, `updated_at/by`); definir el patrón antes de crear la primera entidad | Revisión de diseño (ADR-07) | global |
| L05 | Cada microservicio incluye en su propuesta una "Revisión contra estándares de industria" (§8 de la plantilla): confirmar cumplimiento de los estándares **globales** (heredados/gobernados) + analizar los **específicos de su dominio**; promover a ADR global cualquier hallazgo transversal | Revisión de `asset-inventory` que originó ADR-08..12 | global |
| L06 | Documentar en el código el propósito de cada clase y método significativo (Javadoc/docstring) + el comportamiento no obvio; los tests referencian su ID de caso. Relajar Javadoc en Checkstyle NO exime de documentar lo importante | Revisión de documentación del scaffold de `asset-inventory` | global |
