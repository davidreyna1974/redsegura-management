# Reporte de Aseguramiento de Calidad (QA) — redSegura

Reporte consolidado de las campañas de QA por microservicio, bajo el
[Protocolo de verificación en 4 fases](protocolo_verificacion_4_fases.md).

**Última actualización:** 2026-07-25
**Resultado global:** ✅ **2 módulos certificados** (`asset-inventory-service`,
`config-backup-service`) + **1.er golden path event-driven validado** (`asset-inventory` →
`config-backup`, 6/6 pasos, [`golden_path_asset_config-backup.md`](../../../backend/documentos/integracion/golden_path_asset_config-backup.md)),
0 bugs funcionales sin resolver, 0 regresiones. Resto de microservicios: sin iniciar.

---

## `asset-inventory-service` — ✅ CERTIFICADO · R1 (07-14) + R1.1 (07-15) + R1.2 (07-15) + re-certificación R1.3 (2026-07-17, endurecimiento)

**Build certificado:** rama `develop` del repo `backend` (Java 21, Spring Boot 3.5.16).

> **R1.3 — re-certificación por endurecimiento a producción (4 fases, 2026-07-17):** tras R1.2 se
> cerraron los 4 RNF de etapa DEV pendientes (RNF-08/27/29/30, ADR-14..17): validación de JWT
> issuer+audience, entrega de eventos con publisher confirms + relay `SKIP LOCKED`, gate de SCA/imagen
> (Trivy) en CI, y OpenAPI en runtime (springdoc). **El gate de SCA detectó 5 CVE CRÍTICOS** en las
> dependencias de Spring Boot 3.3.5 → **upgrade a 3.5.16** (springdoc 2.8.17). Se re-ejecutó el
> gatekeeper completo sobre `develop` congelado (**112 tests, 0 fallos, cobertura ≥ 70 %, 0 lint, CI
> verde incluyendo Trivy**) + **verificación en vivo** (10/10 endpoints + JWT 401 + RBAC 403 +
> redacción + Swagger 200). **0 regresiones.** Se añadieron **9 tests** (`AudienceValidatorTest`,
> `ApiDocsIT`, y los de **camino negativo** `JwtIssuerAudienceValidationTest` —issuer/audience
> equivocado rechazado— y `OutboxRelayConfirmFailureTest` —sin ACK del broker, el evento no se marca
> publicado—; casos SEC-05..08, EVT-04/05/06, APIDOC-01/02). Matriz de RNF DEV completa: `matriz_rnf.md`.

> **R1.2 — re-certificación estricta completa (4 fases, 2026-07-15):** tras R1.1 se incorporó la
> **verificación en vivo de endpoints** (curl/Postman sobre `docker-compose.dev.yml` con JWT reales
> de Keycloak), que detectó y corrigió `HALLAZGO-LIVE-01` (PUT no cumplía reemplazo completo RFC 9110;
> +2 tests de regresión `CRUD-04b`/`CRUD-04c`). Se re-ejecutó el **protocolo de 4 fases íntegro** sobre
> el `develop` congelado: **Fase 1** inventario (gatekeeper verde, sin bugs) → **Fase 2** sin bugs
> nuevos → **Fase 3** re-ejecución estricta (**100 tests, 0 fallos, cobertura ≥ 70 %, 0 lint**) +
> **verificación en vivo 13/13** (10 endpoints + PUT-nulifica-omitido + 401 + 403) → **Fase 4**
> certificación. **0 regresiones.** Detalle de la pasada en vivo:
> [`verificacion_endpoints.md` (repo backend)](../../../backend/asset-inventory-service/documentos/verificacion_endpoints.md).

> **R1.1 — re-certificación estricta completa (4 fases, 2026-07-15):** tras R1 se añadieron cambios
> productivos (BSRCH-02 `unaccent`+V6, sobre de evento `version` 1.1.0, JSON Schema a ubicación
> compartida) y de test (VAL-04/RN-02, conformidad de eventos, BDD/Cucumber). Se re-ejecutó el
> **protocolo de 4 fases íntegro** sobre el `develop` congelado: **Fase 1** inventario (matriz
> completa: 71 PASS / 2 N/A / 0 diferido; gatekeeper verde) → **Fase 2** sin bugs → **Fase 3**
> re-ejecución limpia continua (**98 tests, 0 fallos, cobertura ≥ 70 %, 0 lint**) → **Fase 4**
> certificación. **0 regresiones.**

### 1. Resumen ejecutivo
Campaña de QA bajo el Protocolo de 4 fases sobre una versión **congelada** del código. Se verificaron
**73 casos** en las categorías `SEC, RBAC/AUTHZ, CRUD, VAL, FLOW, RN, ERR, CYBER` (+ direccionamiento
dual-stack IP, búsqueda, observabilidad), respaldados por una suite automatizada de **98 tests**
(unit + Testcontainers PostgreSQL/RabbitMQ + MockMvc), con verificación por rol (ADM/OPE/AUD) y
pruebas de seguridad server-side.

| Métrica | Valor |
|---|---|
| Casos de prueba | **73** |
| ✅ PASS | **71** |
| N/A (justificados) | **2** (RN-05/RN-07, imposibles por construcción) |
| ⏳ Diferido | **0** (BSRCH-02 se cerró tras la certificación con `unaccent`, Flyway V6) |
| Tests automatizados | **112** · 0 fallos · cobertura ≥ 70 % statements (R1: 82 → R1.1: 98 → R1.2: 100 → R1.3: 112) |
| Verificación en vivo de endpoints | **10/10 ✅** (curl/Postman sobre Docker Compose; re-verificado en R1.3) |
| Cadena de suministro (SCA/imagen) | **Trivy en CI, verde** (RNF-08); detectó y cerró 5 CVE críticos → Boot 3.5.16 |
| RNF de etapa DEV | **16/16 ✅** (`matriz_rnf.md`); diferidos con disparador (`preparacion_produccion.md`) |
| Lint/formato | 0 Checkstyle · 0 Spotless |
| Regresión final | **0 regresiones** |

### 2. Metodología (4 fases)
1. **Inventario** (código congelado `351e41b`): gatekeeper verde (65 tests); mapeo de la matriz de
   casos → tests; identificación de gaps sin cobertura.
2. **Corrección:** se añadieron los tests faltantes (PUT/CRUD-04, SEC-03/04, FLOW-04, VAL-02..08b,
   ERR-03, BSRCH-03/04, EMPTY-01, IP-08, CYBER-01/03) y se corrigió **1 bug real** (ver §5).
3. **Re-ejecución** completa sobre build congelado: `mvn verify` → 82 tests, 0 fallos (R1); re-corrida
   en R1.1 → **98 tests, 0 fallos**.
4. **Certificación:** gatekeeper + cobertura en verde; este reporte; commit `chore(qa)`.

### 3. Categorías cubiertas
`SEC` (401/403 problem+json, RBAC por endpoint) · `RBAC/AUTHZ` (redacción `mgmtIp` por rol,
autorización server-side) · `CRUD` (alta/consulta/PUT/PATCH/baja lógica) · `VAL` (Bean Validation,
enum, rango, tamaño de página) · `FLOW` (ETag/If-Match 412/428, soft-delete) · `RN` (RN1..RN11:
unicidad, inmutabilidad, idempotencia, eventos, dual-stack RF-05a) · `ERR` (RFC 7807 uniforme, JSON
malformado) · `CYBER` (inyección parametrizada, no fuga de stack trace/internos, authz no asumida
del Gateway). `UI/VIS` → repo `frontend`.

### 4. Fixes destacados (Fase 2)
1. **`size` de página fuera de rango devolvía 500** — la validación `@Max` del parámetro (contrato)
   lanzaba `ConstraintViolationException` **no manejada** → 500 con fuga (viola RNF-09). **Solución:**
   handler dedicado → **422 `VALIDATION_ERROR`** en `problem+json`; además se alineó el contrato
   (`size` máx **100**, antes 200 vs cap de código 100). *Blast radius: local* (capa web + parámetro
   de búsqueda; sin consumidores del contrato aún). Regenerados los DTOs.

*No se hallaron bugs funcionales adicionales.*

### 5. Verificación de regresión final (2026-07-14)
- Gatekeeper: `mvn -pl asset-inventory-service -am clean verify` → **98/98 tests**, cobertura ≥ 70 %,
  0 Checkstyle, 0 Spotless, sobre JDK 21 (toolchain).
- CI de GitHub Actions en verde (status check requerido en `main`).
- **Resultado: 0 regresiones.**

### 6. Observaciones / deuda al cierre
- **BSRCH-02 — cerrado (post-certificación):** búsqueda insensible a **acentos** implementada con la
  extensión `unaccent` (Flyway V6) + envoltura `IMMUTABLE`; verificada (`galón` = `galon`). Queda como
  optimización menor un índice `pg_trgm` para el comodín inicial de los `LIKE %term%`.
- **VAL-04 / RN-02:** se añadieron los tests 1:1 que faltaban (enum `deviceType` inválido → 400;
  `hostname` duplicado → 409). Total tras el cierre: **98 tests**.
- **Conformidad de contrato de eventos (productor-side) — añadida:** los eventos `asset.*` se validan
  contra un **JSON Schema** formal (`asset-event.schema.json`) en `AssetEventContractIT` (happy/edge/sad,
  incl. test negativo del schema y el invariante "sin evento ante fallo"). Es el "mini Pact" posible sin
  consumidor. Sobre de evento en `version` 1.1.0 (payload dual-stack).
- Deuda de producción rastreada en la memoria técnica del módulo (validación `issuer`/`audience` del
  JWT; exportadores de observabilidad por entorno; publisher confirms del outbox; **Pact consumer-driven
  al existir el primer consumidor**; conformidad de respuestas HTTP contra el `openapi.yaml`, opcional).

## `config-backup-service` — ✅ CERTIFICADO · 4 fases, 3 vueltas R-C1/R-C2/R-C3 (2026-07-24)

**Build:** rama `develop` del repo `backend` (Python 3.12/FastAPI). Detalle en
[`reporte_certificacion_qa.md`](../../../backend/config-backup-service/documentos/reporte_certificacion_qa.md)
y [`reporte_r2_revalidacion.md`](../../../backend/config-backup-service/documentos/reporte_r2_revalidacion.md).

- **R1 (2026-07-22):** primera verificación en vivo (14/14 endpoints con JWT reales) durante el
  endurecimiento. **Detectó y corrigió `HALLAZGO-LIVE-CBS-01`**: los hilos de fondo (relay del outbox,
  consumidor) morían ante una caída de conexión al broker sin reconectar → outbox sin drenar. Fix:
  `run_resilient` (reconexión con backoff) + 5 tests de regresión. Origen de la lección **L-QA-07**.
- **R2 (2026-07-24) — revalidación integral (2ª iteración):** gate limpio (76 tests, 94.97 %) +
  en vivo 20/20 + regresión de resiliencia (reinicio de RabbitMQ → el relay sobrevive). 0 hallazgos.
- **Certificación 4 fases — R-C1 (1ª vuelta, 2026-07-24):** **Fase 1** (inventario, código congelado)
  destapó **2 divergencias contrato↔implementación** que R1/R2 no marcaban como fallo (solo validaban
  lo implementado): `HALLAZGO-QA-CBS-01` (`GET /backups` ignoraba los filtros del contrato
  hostname/mgmtIp/status/from/to/sort) y `HALLAZGO-QA-CBS-02` (`Idempotency-Key` declarada, no
  honrada → reintentos duplicaban jobs/schedules). **Fase 2** los corrigió (filtros completos +
  módulo de idempotencia de escritura con reserva por actor/replay/409, migración 0004; +17 tests).
  **Fase 3** re-ejecución limpia (93 tests, 95 %) + en vivo 20/20 + 13/13 de los fixes, 0 regresiones.
  **Fase 4** ✅ **CERTIFICADO**. Lección derivada: **L-QA-08**.
- **Certificación 4 fases — R-C2 (2ª vuelta, 2026-07-24, commit `ceb48a8`):** re-certificación tras
  incorporar los **3 gates de prevención** (test de conformidad contrato↔impl, gate anti-⏳,
  reglas de templates/DoD) y la **aceptación BDD** (pytest-bdd). Ronda íntegra sobre código congelado,
  todos los elementos partiendo de "no validado": **Fase 1** 0 hallazgos (63 casos ✅, conformidad
  13/13 operaciones, 0 parámetros sin honrar) → **Fase 2** sin correcciones → **Fase 3** gate limpio
  (**99 tests, cobertura 95 %**) + en vivo 20/20 + 13/13 → **Fase 4** ✅ **CERTIFICADO, 0 regresiones**.
- **Certificación 4 fases — R-C3 (3ª vuelta, 2026-07-24, commit `81bd1c6`):** re-certificación tras
  cerrar la conformidad de contrato de **eventos producidos** (`HALLAZGO-EVT-CBS-01`: los `config.*`
  se publicaban sin el sobre común y con payloads que no coincidían con el catálogo — la 3.ª cara del
  contrato que ningún gate vigilaba). Se añadió `config-event.schema.json` + test de conformidad
  productor-side. Ronda íntegra desde "no validado": **Fase 1** 0 hallazgos (68 casos ✅; conformidad
  API 13/13 + eventos 4/4 tipos) → **Fase 2** sin correcciones → **Fase 3** gate limpio (**105 tests,
  cobertura 95 %**) + en vivo 20/20 + 13/13 + **envelope de eventos verificado en el broker** →
  **Fase 4** ✅ **CERTIFICADO, 0 regresiones**. Con esto las **3 caras del contrato** (API, eventos
  consumidos, eventos producidos) tienen gate ejecutable.

### 7. Lecciones de QA
- **L-QA-01 — una restricción de contrato sin handler es un 500 latente:** los `@Max/@Min` en
  parámetros lanzan `ConstraintViolationException`; hay que manejarla explícitamente (→ 422) o se
  escapa como 500 con fuga. Aplicar a todos los servicios.
- **L-QA-02 — el número de página/tamaño debe tener una sola fuente de verdad:** el máximo vive en el
  contrato; el código no debe imponer un límite distinto (evita capping silencioso confuso).
- **L-QA-03 — casos "imposibles por construcción" son N/A legítimos:** si el DTO de edición no expone
  un campo inmutable (`serialNumber`, `status`), no hace falta un test de "rechazo"; documentarlo como
  N/A con la justificación estructural.

### 8. Trazabilidad
- Matriz de casos: `codigo/backend/asset-inventory-service/documentos/casos_de_prueba.md`.
- Memoria técnica del módulo (hitos, decisiones, deuda): `.../documentos/memoria_tecnica.md`.
- Suite automatizada: `codigo/backend/asset-inventory-service/src/test/**`.

### 9. Verificación en vivo de endpoints (2026-07-15) — post-certificación

Prueba manual/en vivo de los **10 endpoints** por HTTP real (curl / colección Postman) contra el
**entorno de desarrollo Docker Compose** (servicio empaquetado + PostgreSQL + RabbitMQ + Keycloak
sembrado). Complementa la certificación automatizada con una pasada de humo sobre el artefacto
desplegado. **Resultado: 10/10 ✅.** Reporte detallado (tabla por endpoint + dimensiones de
seguridad + reproducción): [`verificacion_endpoints.md` (repo backend)](../../../backend/asset-inventory-service/documentos/verificacion_endpoints.md).

- **`HALLAZGO-LIVE-01` (corregido):** `PUT /devices/{id}` no cumplía **reemplazo completo**
  (RFC 9110) — `replace()` delegaba en la ruta de merge de PATCH y **no nulificaba los campos
  omitidos**, impidiendo conmutar un dispositivo dual-stack (IPv4+IPv6) a solo-IPv6. **Causa:**
  `applyUpdate()` compartido por PUT y PATCH aplicaba solo campos no nulos. **La suite no lo
  detectó** porque `CRUD-04` hacía PUT reenviando el mismo IPv4 (nunca ejercitó un campo omitido).
  **Fix:** flag `fullReplace` en `applyUpdate` (nulifica opcionales omitidos en PUT; mantiene la
  invariante RF-05a → 422 si quedaría sin dirección). **Regresión:** `CRUD-04b`/`CRUD-04c`
  (**98 → 100 tests**). *Blast radius: local* (sin cambio de contrato ni de eventos). Verificado en
  vivo tras el fix (PUT solo-IPv6 → `managementIpv4: null`, `vendor: null`).
- **L-QA-04 — PUT vs PATCH no comparten semántica:** PUT (RFC 9110) reemplaza el recurso entero
  (los campos omitidos se limpian); PATCH (merge, RFC 7386) solo toca lo presente. Un test de PUT
  debe **ejercitar un campo omitido** y verificar que queda en `null`, no solo reenviar los mismos
  valores. Aplicar a todos los servicios con endpoints PUT.
- **L-QA-05 — la verificación en vivo cubre lo que el harness no ve:** los tests automatizados corren
  dentro del proceso de test; no prueban la **imagen/Dockerfile**, el arranque real, el wiring de
  config/secretos por entorno, los **JWT reales** del IdP, la red entre contenedores ni la
  serialización HTTP de extremo a extremo. Por eso la **verificación en vivo de endpoints** (curl/
  Postman sobre el artefacto desplegado) es **obligatoria por servicio** e integrada en el protocolo
  de 4 fases (Fase 3/4). Es la red que cazó `HALLAZGO-LIVE-01`.
- **L-QA-06 — un gate con dientes encuentra problemas reales el primer día:** al implementar RNF-08
  (Trivy en CI), el escaneo detectó **5 CVE CRÍTICOS** con parche disponible en dependencias
  transitivas de Spring Boot 3.3.5 (Tomcat RCE/bypass de auth, Spring Security bypass) que llevaban
  meses latentes sin que nada avisara. Forzó subir a Spring Boot 3.5.16. Confirma la tesis del
  proceso: *lo que no se gatea, deriva* — y en cuanto se gatea, aflora la deuda oculta. Un RNF sin
  gate ejecutable es aspiracional.
- **L-QA-09 — Pact valida el contrato, no el sistema ensamblado:** el contract testing (Pact/JSON
  Schema, RNF-21) verifica cada lado de una interacción **en aislamiento** (contra un esquema
  compartido, con dobles). **No** prueba que el evento que produce un servicio, publicado por **su**
  relay a un broker **real**, sea enrutado por el exchange correcto y **consumido y proyectado** por
  el **otro** servicio real, con sus dos BD independientes y su seguridad. Esa clase de fallo (nombres
  de exchange/routing distintos, sobre divergente, campos que no casan, colas mal ligadas, consistencia
  que no converge) **solo aparece al integrar en vivo**. Por eso la **integración cross-service (golden
  path) es obligatoria por vector** (RNF-32): cada interacción microservicio↔microservicio se valida
  end-to-end sobre el artefacto desplegado, y un gatekeeper (`check_golden_paths.py`) lo obliga en el
  release. Primera evidencia: `asset-inventory`→`config-backup` (6/6 pasos, 0 hallazgos).
- **L-QA-08 — revalidar lo construido ≠ certificar contra el contrato:** en `config-backup-service`
  las rondas R1/R2 (revalidación) pasaron limpias porque validaban el **comportamiento implementado**;
  la **Fase 1 de la certificación formal** (inventario caso-por-caso contra el `openapi.yaml`) destapó
  **2 divergencias contrato↔implementación** que ninguna revalidación había marcado: filtros de
  listado declarados pero ignorados (resultados silenciosamente incorrectos) y `Idempotency-Key`
  declarada pero no honrada (reintentos duplicaban efectos). **Regla:** el inventario de la Fase 1 se
  hace contra el **contrato** (cada parámetro/cabecera/respuesta del OpenAPI = un caso), no contra lo
  que el código ya hace; un caso sin ✅ PASS bloquea la certificación aunque los tests estén verdes.
- **L-QA-07 — la resiliencia del *plumbing* asíncrono solo se ve rompiendo la conexión:** en
  `config-backup-service` la verificación en vivo destapó `HALLAZGO-LIVE-CBS-01`: el hilo del relay
  del outbox (y consumidor/scheduler) **moría** ante un reset de conexión de RabbitMQ y no reconectaba
  → el outbox dejaba de drenarse permanentemente. Los tests con Testcontainers verifican la *lógica*
  de entrega pero **no** la supervivencia del bucle ante una caída de conexión (no la simulan). **Fix:**
  `run_resilient` (reconexión con backoff exponencial ante errores recuperables) envolviendo cada hilo
  de fondo; **5 tests de regresión** + re-verificación en vivo reiniciando el broker. **Regla:** todo
  bucle de fondo sobre una conexión externa (broker/BD) se prueba contra su caída y recuperación; si
  el proceso lanza hilos daemon de larga vida, cada uno debe auto-recuperarse, no morir.
