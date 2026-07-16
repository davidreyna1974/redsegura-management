# Verificación en vivo de endpoints — <NOMBRE-DEL-SERVICIO>

<!-- GUÍA: registro de la prueba MANUAL/EN VIVO de TODOS los endpoints del servicio, por HTTP real
     (curl / colección Postman) contra el ARTEFACTO EMPAQUETADO Y DESPLEGADO en el entorno de
     desarrollo (Docker Compose), con auth y dependencias reales. Complementa —no sustituye— la suite
     automatizada. Es OBLIGATORIA por servicio (ver qa/estrategia_de_pruebas.md §1b y la definición
     de "done" del CLAUDE.md). Usa datos REALES de la ejecución; borra estas notas de guía. -->

Registro de la prueba en vivo de los **<N> endpoints** del contrato de `<servicio>`, ejecutada por
HTTP real contra el **entorno de desarrollo** (Docker Compose: BD + broker + Keycloak sembrado + el
servicio empaquetado por su `Dockerfile`). Complementa la certificación automatizada (**<N> tests**,
gatekeeper) con una pasada de humo end-to-end sobre el artefacto desplegado.

- **Fecha:** <YYYY-MM-DD>
- **Entorno:** `docker-compose.dev.yml` (servicio en `localhost:<puerto>`, Keycloak en `localhost:8080`)
- **Autenticación:** JWT reales de Keycloak (realm `redsegura`), roles <ADM/OPE/AUD según aplique>
- **Guía para reproducir:** `backend/<servicio>/postman/GUIA_PRUEBAS_POSTMAN.md`

---

## Resultado: <N/N> endpoints ✅

<!-- GUÍA: una fila por CADA operación del openapi.yaml. El estado solo es ✅ si el código HTTP y el
     cuerpo son los esperados. Si algo falla, márcalo y documenta el hallazgo abajo. -->

| # | Método + endpoint | Roles | Prueba | Resultado | Estado |
|---|---|---|---|---|---|
| 1 | `GET /...` | <roles> | <qué se probó> | `<código>` <resumen> | ✅ |
| … | | | | | |
| N | `... /...` | | | | |

### Dimensiones transversales de seguridad (verificadas en vivo)

<!-- GUÍA: adapta a lo que el servicio expone (redacción/ETag pueden ser N/A). El 401 y el 403 del
     rol menos privilegiado son OBLIGATORIOS si hay endpoints protegidos (Gate C del CLAUDE.md). -->

| Caso | Resultado | Estado |
|---|---|---|
| Sin token en endpoints protegidos | `401` `problem+json` | ✅ |
| Rol menos privilegiado sin permiso intenta escribir | `403` | ✅ |
| Redacción de datos sensibles por rol | <resultado / N/A> | <✅/N/A> |
| Concurrencia optimista (`ETag`/`If-Match`) | <resultado / N/A> | <✅/N/A> |
| Semántica **PUT = reemplazo completo** vs **PATCH = merge** | <resultado / N/A> | <✅/N/A> |

---

## Hallazgos de la prueba

<!-- GUÍA: si no hubo hallazgos, escribe "Sin hallazgos: 10/10 a la primera." Si los hubo, uno por
     bloque con: endpoint, síntoma, causa raíz, impacto, por qué la suite no lo cazó, corrección
     (incl. TEST DE REGRESIÓN automatizado añadido) y blast radius. Todo hallazgo se corrige y se
     registra también en reporte_qa.md. -->

### `HALLAZGO-LIVE-0X` (<estado: corregido/abierto>)

- **Endpoint:** `<método + ruta>`.
- **Síntoma:** <qué se observó>.
- **Causa raíz:** <por qué ocurría>.
- **Impacto productivo:** <consecuencia real>.
- **Por qué la suite no lo detectó:** <gap del test automatizado>.
- **Corrección:** <fix> + **test de regresión** `<nombre>` (suite <N-1> → <N>).
- **Blast radius:** <local / global si toca contrato/evento compartido>.

---

## Cómo reproducir

1. `cd <repo> && docker compose -f docker-compose.dev.yml up --build`
2. Importar la colección Postman (`backend/<servicio>/postman/`) o usar los `curl` de `GUIA_PRUEBAS_POSTMAN.md`.
3. Ejecutar en orden: token (por rol) → <flujo de endpoints del servicio>.
4. Probar el rol menos privilegiado (403) y sin token (401); redacción si aplica.
5. Al terminar: `docker compose -f docker-compose.dev.yml down -v`.
