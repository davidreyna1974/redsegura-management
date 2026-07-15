# UAT — asset-inventory-service (Inventario de activos de red)

> Validación orientada al **cliente/usuario final**. Complementa la certificación técnica (QA R1).
> Los escenarios están **automatizados** en `asset-inventory-service/src/test/resources/features/`.

**Última actualización:** 2026-07-15 · **Estado:** listo para visto bueno del cliente

---

## 1. Criterios de aceptación por requisito funcional (RF)

| RF | Criterio de aceptación (lenguaje de negocio) | Escenario BDD que lo evidencia | Estado | Visto bueno cliente |
|---|---|---|---|---|
| **RF-01** | Un administrador puede **registrar** un dispositivo con sus datos (serie, hostname, dirección de gestión, criticidad…) y queda en el inventario | "Un administrador registra y consulta un dispositivo" | ✅ verificado | ☐ |
| **RF-02** | Se puede **consultar, editar y dar de baja** (baja lógica, sin borrar historial) un dispositivo | "…registra y consulta…", "…da de baja un dispositivo" | ✅ verificado | ☐ |
| **RF-03** | El inventario se expone por API como **fuente única de verdad** para el resto del sistema | (transversal; consumido por otros servicios) | ✅ verificado | ☐ |
| **RF-04** | Se puede **buscar y filtrar** dispositivos (por hostname, dirección, ubicación, criticidad…), incluso ignorando acentos | "Búsqueda de dispositivos por hostname" | ✅ verificado | ☐ |
| **RF-05** | Cada cambio del inventario **notifica** al resto del sistema (evento) | (conformidad de eventos, `AssetEventContractIT`) | ✅ verificado | ☐ |
| **RF-05a** | Se puede registrar dirección **IPv4, IPv6 o ambas**, con su prefijo y puerta de enlace | "Registro de un dispositivo con dirección IPv6" | ✅ verificado | ☐ |
| **Seguridad** | Solo perfiles autorizados **modifican**; un **auditor** solo lee y ve la **IP enmascarada** | "Un auditor no puede registrar…", "El auditor ve la dirección enmascarada" | ✅ verificado | ☐ |

## 2. Reporte de avance de negocio (funcionalidades)

| Funcionalidad | Estado | Nota para el cliente |
|---|---|---|
| Alta / consulta / edición / baja de dispositivos | 🟢 Completo | Con control de acceso por perfil |
| Búsqueda y filtrado (incl. acentos) | 🟢 Completo | |
| Direccionamiento IPv4 / IPv6 / dual-stack | 🟢 Completo | Estándar de industria (IPAM) |
| Seguridad de datos (IP enmascarada para auditor) | 🟢 Completo | Dato sensible no visible a perfiles de solo lectura |
| Notificación de cambios al resto del sistema | 🟢 Completo | Base para respaldo y auditoría automáticos |
| Importación masiva de dispositivos | 🟢 Completo | Alta por lotes con resultado por dispositivo |
| **Pantalla / dashboard (uso visual)** | ⚪ Pendiente | Llega con el `frontend`; hoy se valida por API/demo guiada |

**Avance del servicio:** funcionalidades de negocio **completas**; pendiente la **capa visual** (frontend).

## 3. Cómo validar hoy (sin frontend)
- **Lectura y firma** de la tabla §1 (criterios por RF) con el cliente.
- **Demo guiada** por API (Swagger UI / colección Postman) mostrando cada escenario.
- Evidencia objetiva: los escenarios BDD corren **verdes en CI** en cada cambio.

> Firma de aceptación: __________________________  Fecha: ____________
