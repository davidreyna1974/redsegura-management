# Reporte de Aseguramiento de Calidad (QA) — redSegura

Reporte consolidado de las campañas de QA por microservicio, bajo el
[Protocolo de verificación en 4 fases](protocolo_verificacion_4_fases.md).

**Última actualización:** 2026-07-15
**Resultado global:** ✅ **1 módulo certificado** (`asset-inventory-service`), 0 bugs funcionales sin
resolver, 0 regresiones. Resto de microservicios: sin iniciar.

---

## `asset-inventory-service` — ✅ CERTIFICADO · R1 (2026-07-14) + R1.1 (2026-07-15) + re-certificación R1.2 (2026-07-15)

**Build certificado:** rama `develop` del repo `backend` (Java 21, Spring Boot 3.5.16).

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
| Tests automatizados | **100** · 0 fallos · cobertura ≥ 70 % statements (R1: 82 → R1.1: 98 → R1.2: 100) |
| Verificación en vivo de endpoints | **10/10 ✅** (curl/Postman sobre Docker Compose, R1.2) |
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
