# Estrategia de pruebas — redSegura (todos los microservicios)

> Aplica a **todos** los microservicios (Java y Python). Complementa el
> [Protocolo de verificación en 4 fases](protocolo_verificacion_4_fases.md) (el *cómo* de cada ronda)
> y el [Reporte de QA](reporte_qa.md) (resultados). Este documento define **qué tipos de prueba** debe
> tener cada servicio y **cómo** se estandarizan, para que todo servicio nuevo nazca con ellas.
> **Referencia implementada:** `asset-inventory-service` (105 tests + verificación en vivo 10/10).

**Última actualización:** 2026-07-15

---

## 1. Capas de prueba obligatorias (pirámide)

| Capa | Qué verifica | Herramienta (Java) | Herramienta (Python) |
|---|---|---|---|
| **Unit** | Lógica de dominio aislada, sin BD ni red | JUnit 5 + AssertJ | pytest |
| **Integración** | Interacción con dependencias **reales** (BD/broker), sin mocks de infraestructura | **Testcontainers** (PostgreSQL/RabbitMQ) + MockMvc | `testcontainers-python` + `httpx`/TestClient |
| **Conformidad de contrato — API** | Que las respuestas cumplen el `openapi.yaml` propio | `swagger-request-validator` (opcional) | `schemathesis` / validación OpenAPI |
| **Conformidad de contrato — eventos** | Que el payload emitido cumple su **JSON Schema** compartido | `json-schema-validator` (networknt) | `jsonschema` |
| **Aceptación (BDD)** | Reglas de negocio en lenguaje del cliente (base de la UAT) | **Cucumber** (Gherkin español) | `behave` / `pytest-bdd` |
| **Contrato consumidor↔productor** | Que un servicio no rompe lo que otro consume | **Pact** (cuando exista el par consumidor/productor) | Pact-python |
| **Verificación en vivo de endpoints** | Que **todos** los endpoints responden sobre el **artefacto empaquetado y desplegado** (no solo en el harness de test), con auth y dependencias reales | curl / **colección Postman** contra `docker-compose.dev.yml` | curl / Postman contra `docker-compose.dev.yml` |

**Reglas transversales:**
- **Cobertura mínima ≥ 70 % statements** por servicio (gate del `mvn verify` / `pytest --cov`).
- **Gate único e insaltable:** todo (build + tests + cobertura + lint) ligado a un solo comando
  (`mvn verify` en Java; equivalente en Python) y ejecutado por el **CI por servicio** (ver L04/L05).
- **Sin mocks de infraestructura:** BD y broker **reales** vía Testcontainers (RNF-14, database-per-service).
- **Categorías obligatorias por endpoint:** `SEC, RBAC/AUTHZ, CRUD, VAL, FLOW, RN, ERR, CYBER`
  (ver `casos_de_prueba_TEMPLATE.md`).

## 1b. Verificación en vivo de endpoints (OBLIGATORIA por servicio)

> **Regla:** además de la suite automatizada, **todo microservicio** debe pasar una verificación
> **manual/en vivo** de **todos** sus endpoints por HTTP real, contra el **artefacto empaquetado y
> desplegado** en el entorno de desarrollo (Docker Compose), con **auth y dependencias reales**. No
> sustituye a los tests automatizados: los **complementa**, porque prueba cosas que el harness de
> test no cubre (imagen/Dockerfile, arranque real, wiring de config/secretos por entorno, JWT reales
> del IdP, red entre contenedores, serialización HTTP de extremo a extremo).

**Por qué es obligatoria (lección `L-QA-05`).** En `asset-inventory-service`, esta pasada detectó
`HALLAZGO-LIVE-01` (PUT no cumplía reemplazo completo RFC 9110) que la suite automatizada **no**
cazaba porque el test reenviaba los mismos valores en vez de omitir un campo. La verificación en vivo
es la red de seguridad contra defectos de semántica HTTP, de despliegue y de configuración.

**Alcance mínimo (todos los servicios):**
1. **Cobertura total de endpoints:** los **N/N** del `openapi.yaml` (o de la API del servicio),
   cada uno con su código HTTP esperado.
2. **Auth real:** tokens emitidos por **Keycloak** (realm `redsegura`), un usuario por rol
   (ADM/OPE/AUD según la matriz §5).
3. **Dimensiones de seguridad transversales** (adaptar a lo que aplique al servicio):
   - `401` sin token en endpoints protegidos.
   - `403` con el rol **menos** privilegiado sin permiso (Gate C del `CLAUDE.md`).
   - Redacción de datos sensibles por rol (si el servicio los maneja).
   - Concurrencia optimista (`ETag`/`If-Match`) si el servicio la expone.
4. **Semántica HTTP correcta**, en particular: **PUT = reemplazo completo** (RFC 9110, los campos
   omitidos se limpian) **vs PATCH = merge** (RFC 7386, solo lo enviado) — ver `L-QA-04`.
5. **Entregables:** una **colección Postman** en `backend/<servicio>/postman/` (con ejemplos de
   request y **respuestas esperadas** guardadas) + un **reporte**
   `backend/<servicio>/documentos/verificacion_endpoints.md` (desde
   `templates/qa/verificacion_endpoints_TEMPLATE.md`) con la tabla N/N y los hallazgos.

**Herramienta y entorno:** `docker-compose.dev.yml` (infra compartida del backend: PostgreSQL/broker
+ **Keycloak sembrado**) + el servicio empaquetado por su `Dockerfile`; guía de reproducción en
`backend/<servicio>/postman/GUIA_PRUEBAS_POSTMAN.md`. Cualquier hallazgo se corrige, se añade **test
de regresión automatizado** (para que no vuelva a escaparse) y se registra en `reporte_qa.md`.

> Está integrada en la **definición de "done"** (`CLAUDE.md`, Propuesta D) y en la **Fase 3**
> (re-ejecución) del protocolo de 4 fases: una ronda de QA no está completa sin la pasada en vivo.

## 2. Contratos compartidos (clave para Pact)

- **Esquemas de eventos** viven en una **ubicación compartida del monorepo**:
  `codigo/backend/contracts/events/<evento>.schema.json`. El **productor** valida lo que emite y cada
  **consumidor** valida lo que recibe **contra el mismo archivo** → base directa del Pact futuro.
- **Contratos de API:** el `openapi.yaml` de cada servicio (gobernado por Spectral, ADR-12).
- Referencia: `asset-event.schema.json` + `AssetEventContractIT` en `asset-inventory-service`.

## 3. Andamiaje compartido (a materializar al 2.º servicio Java)

- **Módulo commons de test** (o plantilla replicable): base de test de integración (`AbstractIntegrationTest`
  con Testcontainers), helpers de observabilidad, y la config de conformidad de contrato. Hoy vive en
  `asset-inventory-service`; se extrae a un commons cuando se scaffoldee `config-backup` para no
  duplicar. Los `.feature` de aceptación y los JSON Schema de eventos ya están en rutas compartibles.

## 4. Consideraciones de prueba **por servicio** (#2 — considerar desde ahora)

> Los patrones (Testcontainers, conformidad de contrato, BDD, **verificación en vivo de endpoints**,
> 4 fases) son iguales; lo **específico** se define en la sesión de desarrollo de cada servicio, pero
> se anota aquí desde ahora. **Todos** los servicios requieren, además de lo específico, la
> verificación en vivo de endpoints de §1b y un `docker-compose.dev.yml` + colección Postman.

| Servicio | Fase | Particularidades de prueba a considerar |
|---|---|---|
| `asset-inventory-service` | A | ✅ **Implementado** (referencia): CRUD, RBAC, ETag/If-Match, idempotencia, dual-stack IPv4/IPv6, redacción, outbox, bulk, observabilidad, conformidad de eventos, BDD. **Verificación en vivo 10/10** ✅ (`backend/asset-inventory-service/documentos/verificacion_endpoints.md`). |
| `config-backup-service` | A | **Conexión SSH a dispositivos (Netmiko/NAPALM):** mockear/simular el dispositivo (o GNS3 de prueba) — no conectar a equipos reales en tests. `running-config` vs `startup-config`, `diff`, flag `unsavedChanges`. Jobs asíncronos de respaldo. **Primer consumidor de `asset.*`** → habilita el **Pact real** (valida contra `asset-event.schema.json`). |
| `compliance-audit-service` | A | Motor de reglas/hallazgos: casos de política (cumple/incumple), histórico. Consumidor de `asset.*`. |
| `alerting-service` | A | Reglas de alerta (CRUD), evaluación, deduplicación; consumidor de eventos (`config.*`, `compliance.*`). Verificar la regla de `alert.created`. |
| `notification-service` | A | Envío por email/Telegram: **mockear** los proveedores externos (no enviar de verdad); plantillas; reintentos; historial. |
| `scan-orchestrator-service` | B | **Control técnico de alcance de escaneo (RNF-07):** tests de seguridad que confirmen que **solo** se escanean rangos autorizados (red simulada) y se **rechaza** todo lo demás. Orquestación de escaneos. |
| `vulnerability-service` | B | Correlación de vulnerabilidades; severidad; emite `vulnerability.critical_found`. |
| `remediation-tracking-service` | B | Ciclo de vida de remediación (estados), SLA. |
| `reporting-service` | B | Generación de reportes (PDF/датos): validar contenido y agregaciones. |
| `telemetry-collector-service` | B | **TimescaleDB** (Testcontainers con imagen Timescale); ingesta SNMP: mockear el origen; ventanas temporales. |

## 5. Trazabilidad
- Casos por servicio: `<servicio>/documentos/casos_de_prueba.md` (matriz + estado).
- BDD/aceptación: `<servicio>/src/test/resources/features/*.feature` (Java) — base de la UAT.
- **Verificación en vivo:** `backend/<servicio>/documentos/verificacion_endpoints.md` + colección
  Postman + guía en `backend/<servicio>/postman/` (`GUIA_PRUEBAS_POSTMAN.md`).
- UAT (cliente): [`../uat/`](../uat/README.md) — `plan_uat.md` + `<servicio>/{guion_uat.md, demo_guiada.md}`.
