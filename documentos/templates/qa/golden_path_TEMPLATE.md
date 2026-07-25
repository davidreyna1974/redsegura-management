# Prueba de integración (golden path) — `<SERVICIO_ORIGEN>` → `<SERVICIO_DESTINO>`

<!-- GUÍA (RNF-32, L-QA-09): un archivo por VECTOR de interacción (una dirección). Si un servicio
     interactúa con varios, hay un golden path por cada contraparte. Se DISEÑA antes de ejecutar y se
     ubica en `backend/documentos/integracion/`. Se registra en `matriz_interaccion.md` (lo exige el
     gate `scripts/check_golden_paths.py`). Marca el estado EJECUTADO al llenar los resultados. -->

> **Tipo:** integración **cross-service** end-to-end sobre el artefacto desplegado
> (`docker-compose.dev.yml`), con **ambos servicios reales**, broker/BD/IdP reales y JWT reales.
> Complementa —no sustituye— los tests por servicio y los contratos Pact (RNF-21, que validan el
> contrato en aislamiento); esto verifica el **cableado vivo** de extremo a extremo.
>
> **Estado:** <diseñado (pre-ejecución) | ✅ EJECUTADO — ÉXITO (<fecha>)>. **Fecha:** <YYYY-MM-DD>.

## 1. Objetivo
<!-- Qué interacción de negocio se valida, en una frase. -->

## 2. Problema que resuelve
<!-- Por qué los tests por servicio + Pact no bastan: solo esta prueba valida el SISTEMA ENSAMBLADO
     (evento producido por uno → enrutado por el broker real → consumido/proyectado por el otro). -->

## 3. Justificación
<!-- RNF-32; primer momento en que existe el par productor↔consumidor; RNF que refuerza. -->

## 4. Estándares / buenas prácticas que se verifican
<!-- event-driven/coreografía, database-per-service, transactional outbox + confirms, consumidor
     idempotente, contrato Pact compartido, consistencia eventual, seguridad e2e, RNF-07 si aplica. -->

## 5. Topología de la interacción
<!-- Diagrama del flujo: origen --(mecanismo)--> broker/HTTP --> destino. Exchange, routing keys,
     colas, sobre común. -->

## 6. Precondiciones
<!-- Stack levantado (ambos servicios + infra), BD por servicio, IdP sembrado, ambos healthy, JWT. -->

## 7. Pasos de la interacción (cómo funciona · qué se espera · cómo se verifica · resultado)

### Paso 1 — <...>
- **Cómo funciona:** <...>
- **Qué se espera:** <...>
- **Cómo se verifica:** <curl / SELECT en BD / inspección del broker>
- **Resultado:** _(pendiente de ejecución)_

<!-- ... un bloque por paso, cubriendo el ciclo completo del vector (alta/edición/baja si aplica). -->

## 8. Verificaciones transversales (propiedades del sistema)

| ID | Propiedad | Cómo se verifica | Resultado |
|---|---|---|---|
| GP-X1 | Desacoplamiento (sin acoplamiento directo indebido) | <...> | _(pendiente)_ |
| GP-X2 | Database-per-service | <...> | _(pendiente)_ |
| GP-X3 | Idempotencia del consumidor | mismo `eventId` 2× → no duplica | _(pendiente)_ |
| GP-X4 | Entrega garantizada / resiliencia | cola durable / confirms | _(pendiente)_ |
| GP-X5 | Consistencia eventual acotada | converge en < N s | _(pendiente)_ |

## 9. Criterios de éxito (checklist)
- [ ] <Paso 1 ...>
- [ ] Transversales GP-X1..X5.
- [ ] 0 regresiones en los servicios.

## 10. Fuera de alcance (registrado con disparador)
<!-- p. ej. SSH real (PRE-REL, red emulada); consumidores que aún no existen. -->

## 11. Reproducción
<!-- Comandos: docker compose up ambos servicios; token; pasos 1..N. -->

## 12. Resultados de la ejecución
<!-- fecha, versión de imágenes/commits, resultado por paso y criterio, hallazgos + blast radius,
     veredicto. Al llenarse, cambia el Estado del encabezado a EJECUTADO y actualiza matriz_interaccion.md. -->
