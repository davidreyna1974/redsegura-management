# Prueba de integración (golden path) — `asset-inventory` → `config-backup`

> **Tipo:** prueba de **integración cross-service** end-to-end sobre el artefacto desplegado
> (`docker-compose.dev.yml`), con **ambos microservicios reales**, **broker real** (RabbitMQ),
> **bases de datos reales** (una por servicio) y **JWT reales** de Keycloak. Complementa —no
> sustituye— los tests unit/integración por servicio y los **contratos Pact** (que verifican el
> contrato en aislamiento); esta prueba verifica el **cableado vivo** de extremo a extremo.
>
> **Estado:** diseñado (pre-ejecución). Los "Resultado" se llenan al ejecutar. Documento **vivo**.
> **Fecha de diseño:** 2026-07-25.

## 1. Objetivo

Validar que, en un entorno equivalente a producción, la **interacción real entre `asset-inventory` y
`config-backup`** funciona de extremo a extremo: que un cambio en el **inventario** (alta/edición/baja
de un dispositivo, vía REST) se **propaga por eventos** hasta que `config-backup` puede **operar sobre
ese dispositivo** (respaldarlo), y que los cambios posteriores se reflejan — todo **sin acoplamiento
directo** entre los servicios.

## 2. Problema que resuelve

Los tests por servicio y los contratos Pact validan cada pieza **en aislamiento** (con dobles o
contra un esquema compartido). **Ninguno prueba el sistema ensamblado**: que el evento que **produce**
un servicio, publicado por **su** relay a un broker **real**, es **enrutado** por el exchange correcto
y **consumido y proyectado** por el **otro** servicio real, con sus dos bases de datos independientes,
su seguridad, su serialización y su red entre contenedores. Es la clase de fallo que solo aparece al
integrar (nombres de exchange/routing distintos, envelope divergente, campos que no casan, colas mal
ligadas, consistencia eventual que no converge). Esta prueba cierra ese hueco de **cableado vivo**.

## 3. Justificación

- El **golden path de extremo a extremo en Docker Compose** es un hito explícito del proyecto (backend
  `CLAUDE.md` §Próximos pasos).
- `config-backup` es el **primer consumidor** de `asset.*`: es el primer momento en que existe el par
  productor↔consumidor y, por tanto, en que la interacción puede probarse de verdad (antes solo había
  Pact consumidor contra un esquema).
- Refuerza, con evidencia viva, los RNF de comunicación (RNF-21 Pact, RNF-30 entrega garantizada,
  RNF-E1 idempotencia) y el patrón arquitectónico (database-per-service, event-driven).

## 4. Estándares / buenas prácticas que se verifican

| Estándar / patrón | Qué implica en esta prueba |
|---|---|
| **Arquitectura orientada a eventos (coreografía, no orquestación)** | Los servicios reaccionan a eventos; **nadie llama por HTTP al otro**. Desacoplamiento temporal y de despliegue. |
| **Database-per-service** | `config-backup` **no** consulta la BD de `asset-inventory`; mantiene su **propia proyección** (vista de dispositivos) alimentada por eventos. |
| **Transactional outbox** (ADR-04, RNF-30) | El evento se escribe en la **misma transacción** que el cambio de negocio; un relay lo publica con **publisher confirms** (entrega garantizada, at-least-once). |
| **Consumidor idempotente** (RNF-E1) | Deduplicación por `eventId`; reprocesar un evento no duplica ni corrompe la proyección (seguro con at-least-once). |
| **Contrato de eventos compartido + Pact** | Productor y consumidor validan contra el **mismo** `asset-event.schema.json` (sobre §3.3 + payload §4.1). |
| **Consistencia eventual** | La vista de `config-backup` **converge** al estado del inventario tras procesar el evento (no instantáneo, sí acotado). |
| **Read model / CQRS-ish** | `config-backup` mantiene una **proyección de lectura** del inventario, optimizada para su necesidad (saber a qué dispositivos conectarse). |
| **Seguridad de extremo a extremo** | RBAC validado en cada servicio con **JWT reales** de Keycloak; no se confía en el gateway. |
| **Zero-trust del scope de red (RNF-07)** | El respaldo solo intenta SSH a IPs dentro de los CIDRs autorizados (red simulada en dev). |

## 5. Topología de la interacción

```
Cliente (Postman/curl, JWT)
      │  REST POST/PUT/DELETE /api/v1/devices
      ▼
┌─────────────────────┐   asset.*     ┌──────────────────────┐   config.*
│  asset-inventory     │──(outbox+     │  RabbitMQ             │──(bind        (a alerting, futuro)
│  (Java, :8081)       │   relay)─────▶│  exchange topic       │   q.alerting…)
│  BD: asset_inventory │  publisher    │  redsegura.events     │
└─────────────────────┘  confirms     └──────────┬───────────┘
                                                  │ routing asset.* → q.config-backup.asset-events
                                                  ▼
                                       ┌──────────────────────┐
                                       │  config-backup        │  proyecta el dispositivo en su
                                       │  (Python, :8082)      │  vista local (BD config_backup) y
                                       │  BD: config_backup    │  luego lo respalda (SSH→Git→config.*)
                                       └──────────────────────┘
```

- **Exchange:** `redsegura.events` (topic, durable) — declarado por ambos de forma idempotente.
- **Routing keys (productor asset-inventory):** `asset.created`, `asset.updated`, `asset.decommissioned`.
- **Cola (consumidor config-backup):** `q.config-backup.asset-events` (durable, con DLQ) ligada a `asset.*`.
- **Sobre común (§3.3):** `{eventId, eventType, version, occurredAt, source, payload}`.

## 6. Precondiciones

1. `docker compose -f docker-compose.dev.yml up -d --build postgres rabbitmq keycloak asset-inventory config-backup`.
2. Postgres con **dos** bases: `asset_inventory` y `config_backup` (database-per-service).
3. Keycloak sembrado (realm `redsegura`, usuarios admin/operador/auditor, cliente `redsegura-postman`).
4. Ambos servicios `healthy` (`:8081/actuator/health`, `:8082/api/v1/health/readiness`).
5. JWT de `admin` (rol ADM) obtenido por password grant.

## 7. Pasos de la interacción (cómo funciona · qué se espera · cómo se verifica · resultado)

### Paso 1 — Alta de dispositivo en el inventario (REST → `asset-inventory`)
- **Cómo funciona:** el cliente hace `POST /api/v1/devices` a `asset-inventory` con JWT (ADM/OPE) y un
  `DeviceCreateRequest`. El servicio valida (RBAC server-side + reglas de negocio), persiste el
  dispositivo en su BD y, en la **misma transacción**, escribe un evento `asset.created` en su outbox.
- **Qué se espera:** `HTTP 201`; cuerpo con `id` (UUID) + campos; header `ETag`; una fila en la outbox
  de `asset-inventory` con `event_type = asset.created`.
- **Cómo se verifica:** `curl` (código 201 + `id`); *(opcional)* `SELECT` en `asset_inventory.outbox_events`.
- **Resultado:** _(pendiente de ejecución)_

### Paso 2 — Publicación del evento (outbox relay `asset-inventory` → RabbitMQ)
- **Cómo funciona:** el `OutboxRelay` (@Scheduled, `SKIP LOCKED`, **publisher confirms**) toma el evento
  pendiente y lo publica al exchange `redsegura.events` con routing key `asset.created` y el sobre
  común. Marca `published_at` **solo tras el ACK** del broker (RNF-30).
- **Qué se espera:** el evento queda con `published_at` no nulo; el mensaje entra al exchange.
- **Cómo se verifica:** `SELECT published_at` en `asset_inventory.outbox_events`; *(opcional)* contadores
  de publish en la RabbitMQ management API.
- **Resultado:** _(pendiente)_

### Paso 3 — Enrutamiento y entrega (RabbitMQ topic → cola de `config-backup`)
- **Cómo funciona:** el topic exchange enruta `asset.created` (match `asset.*`) a la cola durable
  `q.config-backup.asset-events`. La cola tiene DLQ para mensajes no procesables.
- **Qué se espera:** el mensaje llega a la cola del consumidor; **no** va a la DLQ.
- **Cómo se verifica:** RabbitMQ management API (mensajes entregados a la cola / DLQ vacía); o de forma
  indirecta por el éxito del Paso 4.
- **Resultado:** _(pendiente)_

### Paso 4 — Consumo y proyección (`config-backup`)
- **Cómo funciona:** el consumidor de `config-backup` recibe el mensaje, deserializa el sobre, **deduplica
  por `eventId`** (idempotencia, tabla `processed_events`) y **proyecta** el dispositivo en su vista
  local (`config_backup.devices`). Hace ACK al broker.
- **Qué se espera:** el dispositivo aparece en `config_backup.devices` con `hostname`, `mgmt_ipv4`,
  `status = ACTIVO`; el `eventId` queda en `processed_events`.
- **Cómo se verifica:** `SELECT` en `config_backup.devices` filtrando por `device_id`.
- **Resultado:** _(pendiente)_

### Paso 5 — Operación sobre el dispositivo proyectado (`config-backup` cierra el golden path)
- **Cómo funciona:** ya que `config-backup` "conoce" el dispositivo, `POST /api/v1/backups`
  (`scope=all`) lo **resuelve** y encola un job; el respaldo intenta SSH (en dev **no hay equipo real**
  → termina `FAILED` con gracia, RF-10) y emite `config.backup_failed` (sobre §4.3) por su propio
  outbox→relay.
- **Qué se espera:** `202` + `jobId`; el job llega a `COMPLETED` incluyendo el dispositivo
  (`outcome=FAILED` por SSH); el evento `config.backup_failed` se publica **con sobre conforme**.
- **Cómo se verifica:** `curl` (202 + `GET /backups/jobs/{id}`); inspección de una cola ligada a
  `config.*` en el broker (sobre completo + payload §4.3).
- **Resultado:** _(pendiente)_

### Paso 6 — Propagación de cambios (edición y baja)
- **Cómo funciona:** `PUT`/`PATCH` en `asset-inventory` → `asset.updated` → `config-backup` actualiza la
  vista (hostname/IP). `DELETE` (baja) → `asset.decommissioned` → `config-backup` marca `status=BAJA`
  (RN-CB6); a partir de ahí `scope=all` **deja de incluirlo**.
- **Qué se espera:** la vista de `config-backup` refleja el cambio; el dado de baja se excluye de
  `scope=all`.
- **Cómo se verifica:** `curl` (`PUT`/`DELETE` en `asset-inventory`, con `If-Match`/ETag); `SELECT` en
  `config_backup.devices`; `POST /backups scope=all`.
- **Resultado:** _(pendiente)_

## 8. Verificaciones transversales (propiedades del sistema)

| ID | Propiedad | Cómo se verifica | Resultado |
|---|---|---|---|
| GP-X1 | **Desacoplamiento** (sin HTTP directo entre servicios) | `config-backup` no tiene cliente HTTP a `asset-inventory`; su única fuente del inventario es el stream de eventos | _(pendiente)_ |
| GP-X2 | **Database-per-service** | `config-backup` nunca consulta la BD `asset_inventory`; opera sobre su propia proyección | _(pendiente)_ |
| GP-X3 | **Idempotencia del consumidor** | Reenviar el mismo `eventId` no duplica ni pisa un update posterior | _(pendiente)_ |
| GP-X4 | **Entrega garantizada / resiliencia** | Si `config-backup` está caído al publicarse, al reconectar consume (cola durable, at-least-once) | _(pendiente)_ |
| GP-X5 | **Consistencia eventual acotada** | La proyección converge en un tiempo razonable (< ~10 s en dev) | _(pendiente)_ |

## 9. Criterios de éxito (checklist)

- [ ] **Paso 1:** alta REST → `201` + `id` + `ETag` en `asset-inventory`.
- [ ] **Paso 2:** `asset.created` publicado (`published_at`) con sobre §3.3.
- [ ] **Paso 3:** mensaje entregado a `q.config-backup.asset-events`; DLQ vacía.
- [ ] **Paso 4:** dispositivo proyectado en `config_backup.devices` (< ~10 s) con datos correctos.
- [ ] **Paso 5:** `config-backup` opera sobre el dispositivo (job lo resuelve; emite `config.*` conforme).
- [ ] **Paso 6:** edición y baja se propagan; el dado de baja se excluye de `scope=all`.
- [ ] **Transversales:** GP-X1..X5 satisfechas.
- [ ] **0 regresiones** en los servicios (ambos siguen certificados).

## 10. Fuera de alcance (registrado)

- **SSH exitoso contra un dispositivo real:** en dev no hay equipo; el respaldo termina `FAILED` con
  gracia (RF-10). El camino SSH exitoso se cubre con dobles (tests por servicio) y se validará en
  PRE-REL contra red emulada real (GNS3/Containerlab) — `preparacion_produccion.md §2.9`.
- **Consumidores de `config.*`** (`alerting`): aún no existen; su golden path se documentará al construirlos.
- **Prueba de carga / latencia de propagación bajo volumen** (RNF-01/02): PRE-REL.

## 11. Reproducción

```bash
# 1. Levantar el stack completo (backend/)
docker compose -f docker-compose.dev.yml up -d --build postgres rabbitmq keycloak asset-inventory config-backup
# 2. Token de admin
curl -s -X POST http://localhost:8080/realms/redsegura/protocol/openid-connect/token \
  -d grant_type=password -d client_id=redsegura-postman -d username=admin -d password=password | jq -r .access_token
# 3. Alta en asset-inventory (:8081) → observar la proyección en config-backup (:8082 / BD config_backup)
# (pasos 1..6 arriba). Colecciones Postman por servicio en <servicio>/postman/.
```

## 12. Resultados de la ejecución

_(se completa al ejecutar — fecha, versión de imágenes/commits, resultado por paso y por criterio,
hallazgos si los hubo con su blast radius, y veredicto.)_
