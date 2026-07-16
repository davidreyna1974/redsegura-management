# Demo guiada de UAT — <SERVICIO>

<!-- GUÍA: guion paso a paso para presentar el servicio al CLIENTE y recoger su visto bueno, usando
     peticiones reales por API (colección Postman) — sin necesidad de frontend. Va junto al
     guion_uat.md del mismo servicio en uat/<servicio>/. Borra estas notas de guía. -->

> **Para quién:** la persona que conduce la demo ante el **cliente/líder de proyecto**.
> **Objetivo:** mostrar que cada RF **funciona de verdad** sobre el software desplegado y recoger la
> **firma** de aceptación.

**Duración estimada:** <15–20 min> · **Última actualización:** <YYYY-MM-DD>

---

## 0. Preparación (antes de la reunión)

1. Levantar el entorno de desarrollo:
   `cd codigo/backend && docker compose -f docker-compose.dev.yml up --build`
2. Importar la colección Postman del servicio: `backend/<servicio>/postman/*.postman_collection.json`
   (guía: `backend/<servicio>/postman/GUIA_PRUEBAS_POSTMAN.md`).
3. Tener a la vista `guion_uat.md` (para firmar al final).

> **Encuadre para el cliente (30 s):** <frase que explica que hoy se valida por API, no por pantallas>.

---

## 1. Escenarios (mapeo RF → demo → qué ve el cliente)

<!-- GUÍA: una fila por escenario de negocio. Empieza por el 401 (seguridad), luego login, luego el
     camino feliz de cada RF, y cierra con los casos de rol (auditor/operador). "Request de Postman"
     debe citar carpeta › nombre exactos de la colección. -->

| # | RF | Qué mostrar | Request de Postman (carpeta › nombre) | Qué debe ver el cliente | ✔ |
|---|---|---|---|---|---|
| 1 | Seguridad | Nadie entra sin identificarse | *(sin token)* <endpoint protegido> | **401**: acceso rechazado | ☐ |
| 2 | — | Iniciar sesión (rol con permisos) | **Auth › Token (…)** | `200`; token guardado | ☐ |
| 3 | **RF-0X** | <acción de negocio> | **<carpeta › request>** | <resultado que valida el RF> | ☐ |
| … | | | | | |
| N | Seguridad | Rol de solo lectura no puede modificar | **Auth › Token (<rol>)** → <escritura> | **403**: escritura impedida | ☐ |

> <RF transversales (fuente de verdad / eventos / etc.) que se mencionan de palabra, respaldados por
> las pruebas automáticas.>

---

## 2. Cierre y visto bueno

1. Recapitular lo demostrado; señalar lo pendiente (p. ej. **frontend**).
2. Registrar el visto bueno en `guion_uat.md §1` y recoger la **firma**.
3. Evidencia de respaldo: BDD en CI + `backend/<servicio>/documentos/verificacion_endpoints.md`.

## Referencias
- Guion de UAT (criterios + firma): `guion_uat.md`
- Plan de UAT: `../plan_uat.md`
- Guía técnica Postman: `backend/<servicio>/postman/GUIA_PRUEBAS_POSTMAN.md`
- Escenarios BDD: `backend/<servicio>/src/test/resources/features/*.feature`
