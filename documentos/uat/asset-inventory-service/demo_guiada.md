# Demo guiada de UAT — asset-inventory-service

> **Para quién:** guion que conduce la persona que presenta la demo al **cliente/líder de proyecto**.
> **Objetivo:** mostrar, escenario por escenario, que cada requisito funcional (RF) **funciona de
> verdad** sobre el software desplegado, y recoger el **visto bueno** del cliente.
> **Cómo:** peticiones reales por API con la **colección Postman** del servicio, sin necesidad de
> frontend. Cada paso indica *qué ejecutar*, *qué decir/mostrar* y *qué debe ver el cliente*.

**Duración estimada:** 15–20 min · **Última actualización:** 2026-07-15

---

## 0. Preparación (antes de la reunión)

1. Levantar el entorno de desarrollo (una vez):
   ```bash
   cd codigo/backend && docker compose -f docker-compose.dev.yml up --build
   ```
2. Importar la colección Postman del servicio:
   `codigo/backend/asset-inventory-service/postman/redsegura-asset-inventory.postman_collection.json`
   (guía detallada: [`GUIA_PRUEBAS_POSTMAN.md`](../../../../backend/asset-inventory-service/postman/GUIA_PRUEBAS_POSTMAN.md)).
3. Tener a la vista el [`guion_uat.md`](guion_uat.md) (tabla de criterios por RF, para firmar al final).

> **Encuadre para el cliente (30 s):** "Aún no hay pantallas; hoy validamos que la lógica de negocio
> y la seguridad **ya funcionan** pidiéndoselo directamente al sistema. Cada acción que vería en una
> pantalla, la haremos por API y usted verá el resultado real."

---

## 1. Escenarios (mapeo RF → demo → qué ve el cliente)

| # | RF | Qué mostrar | Request de Postman (carpeta › nombre) | Qué debe ver el cliente | ✔ |
|---|---|---|---|---|---|
| 1 | **Seguridad** | Nadie entra sin identificarse | *(sin token)* **Dispositivos › Consultar dispositivo** | **401**: el sistema rechaza el acceso sin credencial | ☐ |
| 2 | — | Iniciar sesión como Administrador | **Auth › Token (Administrador)** | `200`; el token queda guardado para los siguientes pasos | ☐ |
| 3 | **RF-01** | Registrar un dispositivo | **Dispositivos › Registrar dispositivo (IPv4)** | **201 Creado**; devuelve el dispositivo con su `id` | ☐ |
| 4 | **RF-02** | Consultarlo en el inventario | **Dispositivos › Consultar dispositivo** | `200`; aparece el dispositivo recién dado de alta | ☐ |
| 5 | **RF-04** | Buscar/filtrar por hostname (ignora acentos) | **Dispositivos › Listar / buscar (por hostname)** | `200`; el dispositivo aparece en los resultados | ☐ |
| 6 | **RF-05a** | Registrar dirección **IPv6** (dual-stack) | **Dispositivos › Registrar dispositivo (IPv6)** | **201**; la IPv6 se guarda **normalizada** (estándar de industria) | ☐ |
| 7 | **RF-02** | Editar un dispositivo | **Dispositivos › Editar (PATCH, requiere If-Match)** | `200`; el cambio queda aplicado (control de versión) | ☐ |
| 8 | **RF-02** | Dar de baja (baja lógica) | **Dispositivos › Dar de baja (DELETE, requiere If-Match)** | `204`; se retira del inventario **sin borrar historial** | ☐ |
| 9 | **RF-01** | Alta masiva (por lotes) | **Dispositivos › Importación masiva** → **Estado del job** | `202` + seguimiento; `COMPLETED` con resultado por dispositivo | ☐ |
| 10 | **Seguridad** | Un **Auditor** solo lee | **Auth › Token (Auditor)** → **Consultar dispositivo** | La dirección IP aparece **enmascarada** (`10.0.0.***`) | ☐ |
| 11 | **Seguridad** | Un **Operador** no puede modificar | **Auth › Token (Operador)** → **Registrar dispositivo** | **403**: el sistema impide la escritura al rol de solo lectura | ☐ |

> **RF-03 (fuente de verdad)** y **RF-05 (notificación al resto del sistema)** son transversales: se
> mencionan de palabra ("cada cambio que acaba de ver queda disponible para los demás módulos y emite
> un aviso automático"), respaldados por las pruebas automáticas de contrato de eventos.

---

## 2. Cierre y visto bueno

1. Recapitular: **todas** las funcionalidades de negocio quedaron demostradas en vivo; lo único
   pendiente es la **capa visual (frontend)**.
2. Registrar el visto bueno en la tabla §1 del [`guion_uat.md`](guion_uat.md) (columna *Visto bueno
   cliente*) y recoger la **firma de aceptación**.
3. Evidencia objetiva de respaldo: los mismos escenarios corren **verdes en CI** y en el reporte de
   verificación en vivo del servicio
   ([`verificacion_endpoints.md`](../../../../backend/asset-inventory-service/documentos/verificacion_endpoints.md), 10/10).

---

## Referencias

- Guion de UAT (criterios por RF + firma): [`guion_uat.md`](guion_uat.md)
- Plan de UAT (proceso general): [`../plan_uat.md`](../plan_uat.md)
- Guía técnica de la colección Postman: [`../../../../backend/asset-inventory-service/postman/GUIA_PRUEBAS_POSTMAN.md`](../../../../backend/asset-inventory-service/postman/GUIA_PRUEBAS_POSTMAN.md)
- Escenarios automatizados (BDD): `codigo/backend/asset-inventory-service/src/test/resources/features/inventario.feature`
