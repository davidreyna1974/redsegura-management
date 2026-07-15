# Reporte de Aseguramiento de Calidad (QA) — <NOMBRE DEL PROYECTO>

<!-- GUÍA: documento de presentación del resultado de QA. Pensado para que un revisor entienda
     el rigor del proceso en 1 minuto. Usa datos REALES; nada de números aspiracionales. -->

**Versión certificada:** <vX.Y.Z> · **Fecha de cierre:** <YYYY-MM-DD>
**Resultado:** <✅ CERTIFICADO | ⚠️ con observaciones> — <N módulos, 0 bugs funcionales, 0 regresiones>.

## 1. Resumen ejecutivo

Campaña de QA bajo el **Protocolo de verificación en 4 fases**. Se ejecutaron **<N> casos de prueba**
en <12> categorías sobre los <N> módulos, con verificación por rol/condición, pruebas de seguridad
server-side y una suite automatizada de **<N> tests**.

| Métrica | Valor |
|---|---|
| Casos de prueba ejecutados | **<N>** |
| Resultado | **0 FAIL · 0 bugs funcionales sin resolver** |
| Casos N/A (justificados) | <N> |
| Tests automatizados | <N> · 0 fallos · <cobertura %> |
| Módulos certificados | **<N / N>** |
| Regresión final | **<0 regresiones>** |

## 2. Metodología (4 fases)

| Fase | Qué se hace | Entregable |
|---|---|---|
| 1 — Inventario | Casos sobre código congelado, sin corregir | Lista de bugs (ABIERTO) |
| 2 — Corrección | Fixes + gatekeeper (build+test+lint) | Código corregido, gatekeeper verde |
| 3 — Re-ejecución | Re-ejecutar; lectura estricta habilita certificar | Casos 100% PASS/N/A |
| 4 — Certificación | Gatekeeper + cobertura + commit `chore(qa)` | Módulo certificado |

## 3. Categorías de prueba

`SEC · RBAC · CRUD · VAL · BSRCH · UI · FLOW · RN · ERR · EMPTY · VIS · CYBER`
(adapta las no aplicables a `N/A` según el tipo de proyecto).

## 4. Resultados por módulo

| Módulo | Ronda | PASS · FAIL · N/A | Notas de cierre |
|---|---|---|---|
| <Módulo 1> | <R#> | <n · 0 · n> | <Fase 3 estricta / fixes ...> |
| **TOTAL** | — | **<N · 0 · N>** | **0 bugs funcionales sin resolver.** |

## 5. Fixes destacados (Fase 2)

1. **<título del fix>** — <síntoma> → <solución>. Blast radius: <local/global>.
2. <...>

## 6. Verificación de regresión final (<fecha>)

- Congelamiento: git limpio (0/0 vs origin).
- Gatekeeper: `<build>` 0 errores · `<test>` <X/X> · `<lint>` 0.
- Matriz de autorización por rol intacta; sin token → 401, sin permiso → 403.
- Datos de prueba creados y limpiados (0 residuales).

**Resultado: <0 regresiones>.**

<!-- GUÍA: si el servicio expone una API desplegable, añade esta sección. -->
## 6b. Verificación en vivo de endpoints (<fecha>)

- Prueba manual/en vivo de los **<N/N> endpoints** por HTTP real (curl/Postman) contra el artefacto
  **desplegado** (Docker Compose), con auth y dependencias reales → **<N/N> ✅**. Reporte detallado:
  [`verificacion_endpoints_<servicio>.md`](verificacion_endpoints_<servicio>.md).
- Hallazgos: <ninguno | `HALLAZGO-LIVE-0X` (corregido + test de regresión)>.

## 7. Lecciones de QA destacadas

<!-- GUÍA: 3-7 lecciones transversales que un revisor valore. -->
- <L## — regla a futuro y por qué>.

## 8. Trazabilidad

- Casos por módulo: [`casos_de_prueba_modulo_*.md`](.).
- Protocolo: [`protocolo_verificacion_4_fases.md`](protocolo_verificacion_4_fases.md).
- Decisiones globales: [`../arquitectura/memoria_tecnica_global.md`](../arquitectura/memoria_tecnica_global.md).
