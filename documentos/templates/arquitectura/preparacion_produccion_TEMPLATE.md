# Preparación para producción (production readiness) — <PROYECTO>

<!-- GUÍA: mecanismo para que cada requisito de ENDURECIMIENTO sea obligatorio con DISPARADOR
     explícito (etapa/condición), no algo que se hace "al final". Convierte los gaps diferidos en
     obligatorios-diferidos rastreables. Ajusta las etapas y los ítems a tu proyecto; borra estas
     notas. Principio: "lo que no se gatea, deriva" → materializa como gate ejecutable donde puedas. -->

> **Regla:** un ítem aplazado no es opcional; es *obligatorio-diferido*. Su incumplimiento en la etapa
> que le corresponde **bloquea** la salida a producción. La "definición de done" funcional **no**
> equivale a *production-ready*: se rastrean por separado.

**Última actualización:** <YYYY-MM-DD>

---

## 1. Etapas del ciclo (disparadores)

<!-- GUÍA: define las etapas/condiciones de TU proyecto. Ejemplos abajo; adáptalos. -->

| Clave | Etapa / condición | Cuándo ocurre |
|---|---|---|
| **DEV** | Desarrollo del módulo/servicio | Siempre, dentro de su ciclo |
| **INT-CONS** | Aparece el primer consumidor de un contrato | Al integrar el par consumidor/productor |
| **INT-SYNC** | Primera llamada síncrona saliente | Al introducir esa llamada |
| **DEPLOY** | Despliegue orquestado / infra | Al preparar el despliegue |
| **PRE-REL** | Pre-release a staging | Antes de producción |

---

## 2. Checklist maestra (qué · requisito · disparador · aplica a · estado)

<!-- GUÍA: una fila por ítem de endurecimiento. Categoriza (seguridad, cadena de suministro,
     resiliencia, despliegue, observabilidad, API/calidad, rendimiento…). Marca disparador y estado. -->

### 2.1 Seguridad
| Ítem | Req. | Disparador | Aplica a | Estado |
|---|---|---|---|---|
| Validación de token (firma + issuer + audience + expiración) en el servicio | <RNF-xx> | DEV | <servicios con API protegida> | <✅/🟡/🔵> |
| Secretos externalizados (env → gestor de secretos) | <RNF-xx> | DEV / DEPLOY | todos | <…> |
| … | | | | |

### 2.2 Cadena de suministro
| Ítem | Req. | Disparador | Aplica a | Estado |
|---|---|---|---|---|
| SCA de dependencias bloqueante en crítico (CI) | <RNF-xx> | DEV | todos | <…> |
| Escaneo de imagen de contenedor | <RNF-xx> | DEV | todos | <…> |

### 2.3 Resiliencia y disponibilidad
| Ítem | Req. | Disparador | Aplica a | Estado |
|---|---|---|---|---|
| Timeouts + retry + circuit breaker | <RNF-xx> | INT-SYNC | los que llamen síncronamente a otros | <…> |
| Entrega garantizada de eventos (outbox + confirms + relay multi-réplica + DLQ) | <RNF-xx> | DEV / INT-CONS | prod./cons. de eventos | <…> |

### 2.4 Despliegue y escalabilidad · 2.5 Observabilidad · 2.6 API/calidad · 2.7 Rendimiento
<!-- GUÍA: replica el patrón para cada categoría relevante. -->

---

## 3. Leyenda
✅ hecho · 🟡 en curso · 🔵 diferido (obligatorio en su disparador) · 🔵 N/A (justificado).

## 4. Cómo se hace cumplir
1. **Planificación:** revisión de RNF en `propuesta_modulo` (aplica/N-A/diferido con disparador).
2. **Matriz por servicio:** `<servicio>/documentos/matriz_rnf.md` con evidencia.
3. **Done + QA:** no está "done" con ítems DEV pendientes; no *production-ready* con ítems de su etapa abiertos.
4. **Gate ejecutable** donde se pueda (CI falla si faltan).
5. **Salida a producción:** el plan de salida verifica ítems ≤ PRE-REL en ✅.
