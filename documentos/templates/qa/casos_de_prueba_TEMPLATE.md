# Casos de prueba — módulo <NOMBRE DEL MÓDULO>

<!-- GUÍA: se crea ANTES de escribir código. Es el CRITERIO DE ACEPTACIÓN del módulo.
     Debe existir al menos un caso por cada categoría aplicable, por cada pantalla/unidad.
     Un módulo NO está "done" mientras haya casos sin estado ✅ PASS. -->

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

<!-- GUÍA: para proyectos sin UI (API/CLI/datos), UI/VIS suelen ser N/A; refuerza SEC/RBAC/RN/ERR/CYBER. -->

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

- Botón de guardar en edición habilitado sin cambios → exigir formulario "dirty".
- Acciones en filas clicables sin detener la propagación del evento → click burbujea y navega.
- Botón "Consultar" con rango de fechas inválido (desde > hasta) no deshabilitado.
- Campo de sólo-lectura "bloqueado" visualmente pero editable (no deshabilitado de verdad).
- Dato sensible oculto con CSS pero presente en el DOM/respuesta.

## Resumen de la ronda

- Total casos: <N> · ✅ PASS: <N> · ❌/⚠️: <N> · N/A: <N>.
- Cobertura de categorías: <completa | faltan ...>.
