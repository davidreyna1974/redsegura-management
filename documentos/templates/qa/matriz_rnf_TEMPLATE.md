# Matriz de trazabilidad de RNF — <SERVICIO>

<!-- GUÍA: rastrea, requisito no funcional por requisito no funcional, cómo lo cumple ESTE servicio y
     DÓNDE está la evidencia. Es al RNF lo que casos_de_prueba.md es a lo funcional: sin esta matriz un
     RNF puede saltarse sin dejar rastro. Se llena en la planificación (propuesta_modulo) y se mantiene
     hasta la certificación. Borra estas notas de guía. -->

**Servicio:** <servicio> · **Última actualización:** <YYYY-MM-DD> ·
**Fuente de RNF:** `proyecto_microservicios_redsegura.md §8`

**Leyenda:** ✅ cumplido · 🟡 en curso · 🔵 diferido (obligatorio en su disparador, ver
`arquitectura/preparacion_produccion.md`) · ⬜ N/A (justificado).

| RNF | Estado | Cómo se satisface en este servicio | Evidencia (código / test / config) | Disparador si diferido/N-A |
|---|---|---|---|---|
| RNF-01 (rendimiento p95) | <✅/🟡/🔵/⬜> | <cómo> | <ruta a evidencia> | <etapa / motivo> |
| RNF-03 (auth OAuth2/JWT) | | | | |
| RNF-04 (RBAC server-side) | | | | |
| RNF-08 (SCA + imagen) | | | | |
| RNF-09 (sin fuga de internos) | | | | |
| RNF-10 (Resilience4j) | | | | <INT-SYNC si no hay llamadas salientes> |
| RNF-12 (health) | | | | |
| RNF-15/16/17 (observabilidad) | | | | |
| RNF-18/19/20 (calidad/CI/docs) | | | | |
| RNF-21 (Pact) | | | | <INT-CONS si no hay consumidor> |
| RNF-27 (OpenAPI + Swagger runtime) | | | | |
| RNF-29 (token issuer/audience) | | | | |
| RNF-30 (entrega de eventos) | | | | |
| RNF-31 (readiness) | | | | |
| … (todos los RNF aplicables) | | | | |

> **Regla:** el servicio no está **"done"** con RNF de etapa **DEV** en 🟡; no es **production-ready**
> con ítems de su etapa (INT-*/DEPLOY/PRE-REL) sin cerrar. Los ⬜/🔵 deben tener disparador explícito.
