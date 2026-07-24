# Casos de prueba — módulo <NOMBRE DEL MÓDULO>

<!-- GUÍA: se crea ANTES de escribir código. Es el CRITERIO DE ACEPTACIÓN del módulo.
     Debe existir al menos un caso por cada categoría aplicable, por cada pantalla/unidad.
     Un módulo NO está "done" mientras haya casos sin estado ✅ PASS. -->

> **⚠️ Cobertura de contrato OBLIGATORIA (lección L-QA-08).** Para servicios con `openapi.yaml`, el
> criterio de aceptación es el **contrato**, no lo que el código termine haciendo. Antes de codificar,
> **cada operación, cada parámetro de query, cada cabecera y cada código de respuesta declarados en el
> `openapi.yaml` deben tener su(s) caso(s) aquí.** Un parámetro declarado que no se implemente es un
> bug silencioso (resultados incorrectos sin error). Se refuerza con **dos gates ejecutables**:
> 1. **Test de conformidad contrato↔implementación** (`tests/test_contract_conformance.py`, patrón de
>    `config-backup-service`): falla en CI si la impl no honra un parámetro/cabecera/operación del
>    contrato. **Obligatorio en todo servicio con contrato.**
> 2. **Gate anti-⏳** (`scripts/check_casos_completos.py`, exigido al liberar a `main`): ningún caso
>    puede quedar en ⏳ al declarar el servicio "done".

**Módulo:** <...> · **Ronda:** <R1> · **Fecha:** <YYYY-MM-DD> · **Versión de código:** <commit/tag>

## Formato

| ID | Pantalla/Unidad | Categoría | Descripción | Rol(es) | Resultado esperado | Estado | Notas |
|---|---|---|---|---|---|---|---|

**Estados:** `✅ PASS` (verificado y correcto) · `❌ FAIL` (bug — documentar en memoria §8) ·
`⏳ PENDIENTE` (no ejecutado) · `⚠️ ABIERTO` (bug detectado en Fase 1, sin corregir) · `N/A`.

## Categorías obligatorias (crear casos para TODAS las que apliquen)

| Sigla | Cubre |
|---|---|
| `SEC` | Acceso directo por ruta/endpoint con rol no autorizado → debe negar/redirigir |
| `RBAC` | Elementos/datos que aparecen/ocultan según rol |
| `CRUD` | Crear, leer, editar, eliminar — flujos completos incluyendo recarga |
| `VAL` | Validaciones: vacío, mínimo, máximo, formato, tipo |
| `BSRCH` | Búsquedas: parcial, case-insensitive, accent-insensitive, sin resultados |
| `UI` | Botones, íconos y acciones verificados uno a uno |
| `FLOW` | Flujos de estado/negocio: transiciones, bloqueos |
| `RN` | Reglas de negocio: qué rechaza, con qué código y mensaje |
| `ERR` | Mensajes de error: validación, 4xx/5xx, red caída |
| `EMPTY` | Estados vacíos: sin datos iniciales vs sin resultados de búsqueda |
| `VIS` | Visual: colores, espaciado, truncado, tooltips, responsive |
| `CYBER` | Ciberseguridad básica (OWASP ASVS L1): inyección, autorización server-side, redacción de campos |
| `CONF` | **Conformidad de contrato**: cada operación/parámetro/cabecera/código de respuesta del `openapi.yaml` está implementado y se comporta como declara (filtro que filtra, cabecera que se honra). Respaldado por `test_contract_conformance.py`. |

<!-- GUÍA: para proyectos sin UI (API/CLI/datos), UI/VIS suelen ser N/A; refuerza SEC/RBAC/RN/ERR/CYBER/CONF.
     Para servicios con contrato, CONF es OBLIGATORIA: un caso por cada param/cabecera/respuesta declarados. -->

## Casos

### <Pantalla / Unidad 1>

| ID | Pantalla/Unidad | Cat. | Descripción | Rol(es) | Resultado esperado | Estado | Notas |
|---|---|---|---|---|---|---|---|
| SEC-01 | <...> | SEC | Acceso por ruta directa con rol sin permiso | <ROL_min> | Niega/redirige | ⏳ | |
| RBAC-01 | <...> | RBAC | <elemento> visible sólo para <rol> | <...> | Oculto para no autorizados | ⏳ | |
| CRUD-01 | <...> | CRUD | Crear <entidad> y recargar lista | <...> | Aparece tras recarga | ⏳ | |
| VAL-01 | <...> | VAL | Campo obligatorio vacío | <...> | Error inline, no envía | ⏳ | |
| CYBER-01 | <...> | CYBER | Parámetro malformado | <...> | 400 sin filtrar tipos internos | ⏳ | |

### Conformidad de contrato (obligatorio si el servicio expone API o publica eventos)
<!-- GUÍA: ver estrategia_de_pruebas.md §1-§2. Los esquemas de eventos viven en la ubicación
     compartida codigo/backend/contracts/events/. -->

| ID | Cat. | Descripción | Resultado esperado | Estado |
|---|---|---|---|---|
| EVT-01 | happy | cada evento emitido valida contra su JSON Schema | 0 errores de schema | ⏳ |
| EVT-02 | edge | variantes válidas (campos opcionales, formas alternas) | validan | ⏳ |
| EVT-03 | sad | (1) operación fallida no emite evento; (2) payload malformado rechazado | invariante + schema con dientes | ⏳ |

### Aceptación (BDD) — base de la UAT
<!-- GUÍA: escenarios Gherkin en <servicio>/src/test/resources/features/, en lenguaje de negocio;
     automatizados (Cucumber/behave) y aprobados por el cliente. Ver plan_uat.md. -->

| ID | Descripción (escenario de negocio) | Estado |
|---|---|---|
| UAT-01 | <escenario del RF principal en lenguaje de negocio> | ⏳ |

## Patrones que han causado bugs reales (revisar siempre)

**Frontend/UI:**
- Botón de guardar en edición habilitado sin cambios → exigir formulario "dirty".
- Acciones en filas clicables sin detener la propagación del evento → click burbujea y navega.
- Botón "Consultar" con rango de fechas inválido (desde > hasta) no deshabilitado.
- Campo de sólo-lectura "bloqueado" visualmente pero editable (no deshabilitado de verdad).
- Dato sensible oculto con CSS pero presente en el DOM/respuesta.

**Backend/API (cazados por la verificación EN VIVO, no por los tests automatizados):**
- **PUT que no reemplaza de verdad** (`HALLAZGO-LIVE-01`): `PUT` que delega en el merge de `PATCH`
  no nulifica los campos omitidos → no cumple RFC 9110 (reemplazo completo). El test no lo cazaba
  porque reenviaba todos los valores en vez de omitir uno. Probar PUT **omitiendo** un campo opcional.
- **Hilo de fondo que muere ante caída de conexión** (`HALLAZGO-LIVE-CBS-01`): consumidor/relay/
  scheduler que no reconectan tras un reset del broker/BD → el trabajo de fondo (drenar el outbox,
  consumir eventos) se detiene permanentemente. Los tests con Testcontainers no lo exponen (no
  simulan cortes de conexión). Probar **reiniciando el broker/BD** en vivo y verificando que el hilo
  sobrevive (reconexión con backoff) y reanuda.
- **Semántica HTTP asumida:** 202 vs 201, aislamiento de tipo en recursos polimórficos (un `jobId` de
  un tipo no debe resolver en la ruta de otro), `problem+json` en TODOS los errores (no solo algunos).

## Resumen de la ronda

- Total casos: <N> · ✅ PASS: <N> · ❌/⚠️: <N> · N/A: <N>.
- Cobertura de categorías: <completa | faltan ...>.
