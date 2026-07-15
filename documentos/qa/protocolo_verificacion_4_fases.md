# Protocolo de verificación en 4 fases — redSegura

> Metodología **permanente y global** para cualquier ronda de QA (microservicio nuevo,
> re-ejecución, post-fix). Una sola metodología para todo el sistema. Adapta los comandos del
> gatekeeper al stack del servicio; **la regla fundamental no se cambia**.

**Última actualización:** 2026-07-15 · **Estado:** vigente

---

## Regla fundamental (inamovible)

> Una ronda de pruebas solo es válida si se ejecuta **íntegra sobre una versión congelada del
> código**, sin modificaciones entre el primer y el último caso. Si durante la ronda se
> encuentra un bug y se **corrige**, la ronda se **invalida** y se reinicia desde cero.

Motivo: mezclar "probar" y "corregir" en la misma ronda invalida los casos ya ejecutados (no se
sabe si pasaron sobre el código final o sobre el previo).

---

## Alcance del blast radius en este monorepo

- **Cambio local** (lógica interna de un microservicio) → re-probar **solo ese servicio**.
- **Cambio global** (contrato de API/evento compartido, API Gateway, o configuración de
  seguridad Keycloak/JWT) → re-probar **todos los microservicios consumidores** de ese contrato,
  verificado con **Pact**.

---

## Las 4 fases

### FASE 1 — Inventario (código congelado)
- Ejecutar **todos** los casos de prueba del alcance, **sin tocar código**.
- Documentar cada bug con estado `⚠️ ABIERTO`. **No corregir nada.**
- Objetivo: conocer el estado real del sistema antes de intervenir.

### FASE 2 — Corrección + gatekeeper
- Corregir los bugs del inventario en ciclo normal de desarrollo (rama `fix/…`).
- **Gatekeeper obligatorio por cada fix**, por microservicio afectado:

  **Servicios Java (Maven):**
  1. `mvn -pl <servicio> -am clean package` → 0 errores
  2. `mvn -pl <servicio> test` → 0 fallos
  3. `mvn -pl <servicio> checkstyle:check spotless:check` → 0 errores

  **Servicios Python (FastAPI):**
  1. `ruff check <servicio>/` && `mypy <servicio>/` → 0 errores
  2. `pytest <servicio>/tests --cov=<servicio>` → 0 fallos

- Documentar el **blast radius** de cada fix (tabla abajo). Un fix a un contrato/evento
  compartido escala el alcance a **global**.

### FASE 3 — Re-ejecución completa desde cero
- Re-ejecutar los casos de los microservicios con blast radius afectado, en una sesión continua.
- Si aparece un bug nuevo → documentar, **NO corregir**, terminar la fase → volver a Fase 2.
- **Lectura estricta vs por blast radius:** declarar cuál se aplicó.
  - *Por blast radius* = solo servicios tocados por los fixes (iteración rápida).
  - *Estricta* = **todos** los casos del servicio, sesión continua, sobre build congelado.
  - **Solo una Fase 3 estricta habilita declarar el microservicio CERTIFICADO.**
- Antes del primer caso, verificar el **congelamiento**: git limpio, dependencias arriba
  (PostgreSQL/RabbitMQ vía Docker/Testcontainers), build vigente (no *stale*).
- **Verificación en vivo de endpoints (obligatoria):** además de la suite automatizada, ejecutar la
  pasada manual/en vivo de **todos** los endpoints por HTTP real (curl/Postman) contra el artefacto
  desplegado en `docker-compose.dev.yml` (auth y dependencias reales). Prueba lo que el harness no
  cubre (imagen, arranque, config/secretos, JWT reales, red entre contenedores). Ver
  `estrategia_de_pruebas.md §1b`. Todo hallazgo → Fase 2 + test de regresión automatizado.

### FASE 4 — Certificación
- Gatekeeper completo del servicio: build + tests con cobertura **≥ 70 % statements**, 0 fallos,
  lint en verde.
- **Verificación en vivo de los N/N endpoints ✅** registrada en `qa/verificacion_endpoints_<servicio>.md`
  (desde `templates/qa/verificacion_endpoints_TEMPLATE.md`).
- Si el servicio expone/consume un contrato: **Pact en verde** para todos los consumidores.
- Actualizar el estado de sesión con resultado **CERTIFICADO** y el resumen de cobertura en el
  documento de casos de prueba del servicio.
- Commit: `chore(qa): verificacion completa 4 fases <servicio> — <fecha>`.
- Consolidar el resultado en el reporte de QA global (`../qa/reporte_qa.md`).

---

## Blast radius (documentar por cada fix)

| Tipo de cambio | Alcance | Re-probar en Fase 3 |
|---|---|---|
| Lógica interna de un microservicio (service/repository) | Local | Solo ese servicio |
| Endpoint no compartido de un servicio | Local | Solo ese servicio |
| **Contrato OpenAPI compartido** (consumido por otro servicio/frontend) | **Global** | Todos los consumidores + Pact |
| **Esquema de un evento** del message broker | **Global** | Todos los servicios que publican/consumen ese evento + Pact |
| API Gateway (enrutamiento/agregación/rate limit) | **Global** | Todos los servicios expuestos |
| Configuración de seguridad (Keycloak/JWT/RBAC transversal) | **Global** | Todos los servicios |
| Librería/módulo compartido, manejador global de errores | **Global** | Todos los servicios que lo usan |

---

## Categorías de prueba obligatorias (por microservicio/endpoint)

`SEC · RBAC/AUTHZ · CRUD · VAL · FLOW · RN · ERR · CYBER` (adaptar a `N/A` las que no apliquen;
`UI/VIS/BSRCH` pertenecen al repo frontend). Un microservicio **no está "done"** si hay casos sin
`✅ PASS`.

## Catálogo de técnicas de verificación (por categoría)

- **VAL:** enviar entrada real y leer el mensaje de error (400/422), no inspeccionar estado interno.
- **RBAC/AUTHZ:** comprobar **ausencia en la respuesta** (403/404), no solo que la UI oculte el botón.
- **SEC:** llamada por ruta directa con el rol **menos** privilegiado (Auditor) que no debe tener acceso.
- **CYBER (server-side):** `curl`/cliente con credencial (JWT) por rol → verificar enforcement y
  **redacción de campos sensibles** (credenciales de dispositivos nunca expuestas).
- **FLOW (máquinas de estado):** intentar transiciones inválidas → rechazo (409/422) e historial auditable.
- **Resiliencia:** apagar una dependencia (BD/broker/dispositivo) → verificar timeout/circuit
  breaker y mensaje útil, no *crash* (RNF-10, RNF-11).
- **Eventos:** publicar → consumir → idempotencia (reproceso sin efecto) → reintento → DLQ tras N fallos.
- **Búsqueda:** parcial, insensible a mayúsculas y acentos, y caso "sin resultados".
- **Semántica HTTP:** **PUT = reemplazo completo** (RFC 9110: los campos omitidos se limpian) **vs
  PATCH = merge** (RFC 7386: solo lo enviado). Un test de PUT debe **ejercitar un campo omitido** y
  verificar que queda en `null`, no solo reenviar los mismos valores (lección `L-QA-04`).
- **Verificación en vivo (endpoints):** correr **todos** los endpoints por HTTP real contra el
  artefacto desplegado (Docker Compose), con JWT reales del IdP — cubre defectos de despliegue,
  config y semántica HTTP que el harness de test no ve (lección `L-QA-05`).

---

## Estado de la sesión

El archivo `../sesiones/estado_sesion_activa.md` persiste el avance de la ronda.
**Leerlo al iniciar cualquier sesión de pruebas** para retomar sin pérdida de contexto.
El reporte consolidado de resultados por servicio vive en `../qa/reporte_qa.md`.
