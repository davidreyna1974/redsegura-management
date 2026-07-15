# Reporte de Aseguramiento de Calidad (QA) — redSegura

Reporte consolidado de las campañas de QA por microservicio, bajo el
[Protocolo de verificación en 4 fases](protocolo_verificacion_4_fases.md).

**Última actualización:** 2026-07-14
**Resultado global:** ✅ **1 módulo certificado** (`asset-inventory-service`), 0 bugs funcionales sin
resolver, 0 regresiones. Resto de microservicios: sin iniciar.

---

## `asset-inventory-service` — Ronda R1 · ✅ CERTIFICADO (2026-07-14)

**Build certificado:** rama `develop` del repo `backend` (Java 21, Spring Boot 3.3.5).

### 1. Resumen ejecutivo
Campaña de QA bajo el Protocolo de 4 fases sobre una versión **congelada** del código. Se verificaron
**73 casos** en las categorías `SEC, RBAC/AUTHZ, CRUD, VAL, FLOW, RN, ERR, CYBER` (+ direccionamiento
dual-stack IP, búsqueda, observabilidad), respaldados por una suite automatizada de **82 tests**
(unit + Testcontainers PostgreSQL/RabbitMQ + MockMvc), con verificación por rol (ADM/OPE/AUD) y
pruebas de seguridad server-side.

| Métrica | Valor |
|---|---|
| Casos de prueba | **73** |
| ✅ PASS | **70** |
| N/A (justificados) | **2** (RN-05/RN-07, imposibles por construcción) |
| ⏳ Diferido | **1** (BSRCH-02 búsqueda insensible a acentos — requiere `unaccent`, deuda documentada) |
| Tests automatizados | **82** · 0 fallos · cobertura ≥ 70 % statements |
| Lint/formato | 0 Checkstyle · 0 Spotless |
| Regresión final | **0 regresiones** |

### 2. Metodología (4 fases)
1. **Inventario** (código congelado `351e41b`): gatekeeper verde (65 tests); mapeo de la matriz de
   casos → tests; identificación de gaps sin cobertura.
2. **Corrección:** se añadieron los tests faltantes (PUT/CRUD-04, SEC-03/04, FLOW-04, VAL-02..08b,
   ERR-03, BSRCH-03/04, EMPTY-01, IP-08, CYBER-01/03) y se corrigió **1 bug real** (ver §5).
3. **Re-ejecución** completa sobre build congelado: `mvn verify` → 82 tests, 0 fallos.
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
- Gatekeeper: `mvn -pl asset-inventory-service -am clean verify` → **82/82 tests**, cobertura ≥ 70 %,
  0 Checkstyle, 0 Spotless, sobre JDK 21 (toolchain).
- CI de GitHub Actions en verde (status check requerido en `main`).
- **Resultado: 0 regresiones.**

### 6. Observaciones / deuda al cierre
- **BSRCH-02 (diferido):** búsqueda insensible a **acentos** — requiere `unaccent` en PostgreSQL; hoy
  la búsqueda es insensible a mayúsculas. Registrada como deuda funcional menor.
- Deuda de producción rastreada en la memoria técnica del módulo (validación `issuer`/`audience` del
  JWT; exportadores de observabilidad por entorno; publisher confirms del outbox; Pact al existir el
  primer consumidor).

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
