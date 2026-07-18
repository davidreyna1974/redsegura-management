# Preparación para producción (production readiness) — redSegura

> **Qué es (RNF-31):** el mecanismo que convierte cada requisito de **endurecimiento** en un
> ítem **obligatorio con disparador explícito** — para que no se olvide ni se posponga "al final".
> Cada ítem declara **cuándo se activa** (etapa o condición), **a qué servicios aplica** y su
> **estado**. Un ítem aplazado **no es opcional**: es *obligatorio-diferido*, y su incumplimiento en
> la etapa que le corresponde **bloquea** la salida a producción.

> **Principio:** *lo que no se gatea, deriva.* Cada ítem se atiende **en la etapa donde el esfuerzo de
> desarrollo es más eficiente** (no necesariamente al cierre), y donde sea posible se materializa como
> **gate ejecutable** (falla el CI/gatekeeper si no se cumple), no como recordatorio.

**Última actualización:** 2026-07-15 · **Estado:** vigente

---

## 1. Etapas del ciclo (disparadores)

Los ítems se anclan a una de estas etapas/condiciones, no a "el final":

| Clave | Etapa / condición | Cuándo ocurre |
|---|---|---|
| **DEV** | Desarrollo del microservicio | Siempre, dentro del ciclo del servicio |
| **INT-CONS** | Aparece el **primer consumidor** de un contrato (API/evento) del servicio | Al construir un servicio que consume a otro |
| **INT-SYNC** | El servicio hace su **primera llamada síncrona saliente** (a otro servicio o dispositivo) | Cuando se introduce esa llamada |
| **GP** | **Golden path** de extremo a extremo en Docker Compose | Al integrar el flujo Fase A |
| **DEPLOY** | Etapa de **despliegue** (k8s/EKS + IaC) | Al preparar el despliegue orquestado |
| **PRE-REL** | **Pre-release** a staging (verificación no funcional real) | Antes de promover a producción |

---

## 2. Checklist maestra (por ítem: qué · RNF · disparador · aplica a · estado)

### 2.1 Seguridad

| Ítem | RNF | Disparador | Aplica a | Estado (asset-inventory) |
|---|---|---|---|---|
| Validación de token: firma **+ issuer + audience + expiración** en el servicio | RNF-29 | **DEV** | todos los que exponen API protegida | ✅ hecho |
| RBAC validado server-side por endpoint (no confiar solo en el Gateway) | RNF-04 | **DEV** | todos | ✅ hecho |
| Errores sin fuga de internos (RFC 7807) | RNF-09 | **DEV** | todos | ✅ hecho |
| Redacción de datos sensibles por rol | RNF-04 | **DEV** | los que manejan datos sensibles | ✅ hecho |
| Secretos externalizados (env var mínimo; **gestor de secretos** en despliegue) | RNF-06 | env var **DEV**; gestor **DEPLOY** | todos | 🟢 env ✅ / 🔵 gestor diferido (DEPLOY) |
| Alcance de escaneo como control técnico obligatorio | RNF-07 | **DEV** | `scan-orchestrator-service` | 🔵 N/A (otro servicio) |
| TLS extremo-cliente + segmentación interna | RNF-05 | **DEPLOY** | infra | 🔵 diferido (DEPLOY) |
| **Contenedor como usuario NO-root** (hardening de imagen) | RNF-05/08 | **DEV** (al haber Dockerfile) | todos | ✅ hecho |
| **DAST** (escaneo dinámico tipo OWASP ZAP contra el servicio corriendo) | RNF-08 | **PRE-REL** | todos con API | 🔵 diferido (PRE-REL) |

### 2.2 Cadena de suministro (supply chain)

| Ítem | RNF | Disparador | Aplica a | Estado |
|---|---|---|---|---|
| SCA de dependencias **bloqueante en crítico** en CI (Dependabot + dependency-check/pip-audit) | RNF-08 | **DEV** | todos | ✅ hecho |
| Escaneo de **imagen de contenedor** (Trivy/Grype) bloqueante en crítico | RNF-08 | **DEV** (al haber Dockerfile) | todos | ✅ hecho |

### 2.3 Resiliencia y disponibilidad

| Ítem | RNF | Disparador | Aplica a | Estado |
|---|---|---|---|---|
| Timeouts + retry con backoff + circuit breaker (Resilience4j) | RNF-10 | **INT-SYNC** | los que llamen síncronamente a otro servicio/dispositivo | 🔵 N/A hoy → obligatorio en INT-SYNC |
| Degradación con gracia ante caída de dependencia no crítica | RNF-11 | **INT-SYNC / GP** | los que dependan de otros | 🔵 diferido (INT-SYNC) |
| Health probes liveness/readiness | RNF-12 | **DEV** | todos | ✅ hecho |
| **Entrega garantizada de eventos**: outbox + publisher confirms + relay multi-réplica (SKIP LOCKED) + DLQ | RNF-30 | **DEV** (productor) / **INT-CONS** (idempotencia consumidor) | productores y consumidores de eventos | ✅ hecho |
| **Graceful shutdown** (drena peticiones + tareas @Async en SIGTERM) | RNF-13 | **DEV** | todos | ✅ hecho |
| **Retención de datos operativos** (purga de outbox publicado/idempotencia/jobs — evita crecimiento sin límite) | RNF-14 | **DEV** | los que acumulan filas operativas | ✅ hecho |

### 2.4 Escalabilidad y despliegue

| Ítem | RNF | Disparador | Aplica a | Estado |
|---|---|---|---|---|
| Stateless + autoscaling (HPA) demostrado en k8s | RNF-13 | **DEPLOY** | todos | 🔵 diferido (DEPLOY) |
| Paridad Docker Compose ↔ k8s sin cambios de código (12-factor) | RNF-22 | **DEPLOY** | todos | 🔵 diferido (DEPLOY) |
| Infra como código (Terraform), reproducible | RNF-23 | **DEPLOY** | infra | 🔵 diferido (DEPLOY) |
| Manifiestos de despliegue (k8s/Helm) del servicio | RNF-13/22 | **DEPLOY** | todos | 🔵 diferido (DEPLOY) |

### 2.5 Observabilidad

| Ítem | RNF | Disparador | Aplica a | Estado |
|---|---|---|---|---|
| Métricas Prometheus (JVM/HTTP) + **métricas de dominio** (altas, eventos publicados) | RNF-15 | **DEV** | todos | ✅ hecho |
| Trazas distribuidas con correlación | RNF-16 | **DEV** (instrumentación) / **GP** (propagación entre servicios) | todos | 🟢 instrumentado ✅ / 🔵 export por entorno (DEPLOY) |
| Logs estructurados sin PII/secretos | RNF-17 | **DEV** | todos | ✅ hecho |
| **Dashboards (Grafana) + reglas de alerta / SLO** | RNF-15 | **DEPLOY** | todos | 🔵 diferido (DEPLOY) |

### 2.6 API, calidad y contratos

| Ítem | RNF | Disparador | Aplica a | Estado |
|---|---|---|---|---|
| Cobertura ≥ 70 % + gatekeeper en CI | RNF-18/19 | **DEV** | todos | ✅ hecho |
| Documentación previa al código (propuesta/casos/memoria) | RNF-20 | **DEV** | todos | ✅ hecho |
| **Swagger UI en runtime** (springdoc) sirviendo el contrato | RNF-27 | **DEV** | todos | ✅ hecho |
| Contract testing (Pact) consumidor↔productor | RNF-21 | **INT-CONS** | pares consumidor/productor | 🔵 N/A hoy → obligatorio en INT-CONS |
| Verificación en vivo de endpoints (10/10) | (metodología QA) | **DEV** | todos | ✅ hecho |
| **Mutation testing** (PIT) para medir la *calidad* de los tests (no solo cobertura) | (calidad) | **opcional** | todos | 🔵 registrado (opcional) |

### 2.7 Rendimiento

| Ítem | RNF | Disparador | Aplica a | Estado |
|---|---|---|---|---|
| GET p95 < 300 ms verificado con **prueba de carga** | RNF-01 | **PRE-REL** | todos | 🔵 diferido (PRE-REL) |
| Respaldo de config < 30 s | RNF-02 | **PRE-REL** | `config-backup-service` | 🔵 N/A (otro servicio) |

### 2.8 Operación y gobernanza

| Ítem | RNF | Disparador | Aplica a | Estado |
|---|---|---|---|---|
| **Runbook operativo** del servicio (desplegar/rollback/diagnóstico/alertas) | RNF-20 | **DEPLOY** | todos | 🔵 diferido (DEPLOY) — plantilla en `templates/despliegue/runbook_despliegue_TEMPLATE.md` |
| **LICENSE** del repositorio (hoy "por definir") | — | **decisión** | repos | 🔵 registrado (decisión legal/producto) |

---

## 3. Leyenda de estado

- ✅ **hecho** · 🟡 **en curso** (este ciclo de desarrollo) · 🔵 **diferido** (obligatorio en su
  disparador) · 🔵 **N/A** (no aplica a este servicio, justificado).

## 4. Cómo se hace cumplir (no es un recordatorio pasivo)

1. **Planificación (pre-código):** cada servicio revisa esta checklist en su `propuesta_modulo`
   (sección "Revisión de RNF"), marcando aplica/N-A/diferido con disparador.
2. **Matriz de RNF por servicio:** `<servicio>/documentos/matriz_rnf.md` rastrea el cumplimiento y
   enlaza la evidencia (código/test).
3. **Definición de "done" y QA de 4 fases:** un servicio no está "done" con RNF DEV pendientes; no es
   *production-ready* con ítems de su etapa sin cerrar (ver `CLAUDE.md` y
   `qa/protocolo_verificacion_4_fases.md`).
4. **Gate ejecutable donde se pueda:** SCA/imagen en CI, cobertura, lint, contrato — el pipeline
   falla si faltan.
5. **Salida a producción:** `despliegue/plan_salida_produccion.md` verifica que **todos** los ítems
   de etapas ≤ PRE-REL estén ✅ para el servicio antes de promover.

## 5. Trazabilidad
- Requisitos: `proyecto_microservicios_redsegura.md §8` (RNF).
- Matriz por servicio: `backend/<servicio>/documentos/matriz_rnf.md`.
- Decisiones: `arquitectura/memoria_tecnica_global.md` (ADRs).
- Plantilla: `templates/arquitectura/preparacion_produccion_TEMPLATE.md`.
