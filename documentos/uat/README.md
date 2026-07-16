# UAT — Validación con el cliente (redSegura)

**UAT** (*User Acceptance Testing*) = la **aceptación del cliente/usuario final**. Aquí vive, **todo
junto**, el material que se **presenta y firma con el cliente** para dar por buenas las
funcionalidades de cada microservicio — separado de la verificación técnica (que es del equipo de
desarrollo y vive en el repo `backend` y en `qa/`).

## Cómo está organizado

```
uat/
├── README.md                 ← este índice
├── plan_uat.md               ← plan GENERAL de UAT (proceso, instrumentos, roles) — todo el proyecto
└── <servicio>/               ← un paquete por microservicio
    ├── guion_uat.md          ← criterios de aceptación por RF + reporte de avance + FIRMA del cliente
    └── demo_guiada.md        ← guion paso a paso de la demo en vivo (qué mostrar y qué verá el cliente)
```

## Contenido por microservicio

| Servicio | Guion (criterios + firma) | Demo guiada | Estado |
|---|---|---|---|
| `asset-inventory-service` | [guion_uat.md](asset-inventory-service/guion_uat.md) | [demo_guiada.md](asset-inventory-service/demo_guiada.md) | Listo para visto bueno del cliente |

> Los demás servicios tendrán su carpeta `uat/<servicio>/` al implementarse (se generan desde
> `../templates/uat/`).

## Cómo se usa (flujo de una sesión de aceptación)

1. **Preparar:** levantar el entorno de desarrollo y la colección Postman del servicio
   (ver la §0 de la `demo_guiada.md` del servicio).
2. **Presentar:** conducir la **demo guiada** escenario por escenario frente al cliente.
3. **Aceptar:** el cliente marca los criterios en `guion_uat.md §1` y **firma** la aceptación.
4. **Respaldo:** la evidencia técnica (BDD en CI + verificación en vivo 10/10) está en el repo
   `backend` (`<servicio>/documentos/verificacion_endpoints.md`) y en el reporte de QA consolidado
   ([`../qa/reporte_qa.md`](../qa/reporte_qa.md)).

## Relación con otros documentos

| Quiero… | Está en… | Público |
|---|---|---|
| Validar/firmar funcionalidades con el cliente | **este `uat/`** | Cliente / líder de proyecto |
| El detalle técnico de qué se probó y cómo | `backend/<servicio>/documentos/verificacion_endpoints.md` + `qa/reporte_qa.md` | Equipo de desarrollo |
| Ejecutar yo mismo las pruebas por API | `backend/<servicio>/postman/GUIA_PRUEBAS_POSTMAN.md` | Técnico |
| La metodología de pruebas del proyecto | [`../qa/estrategia_de_pruebas.md`](../qa/estrategia_de_pruebas.md) | Equipo de desarrollo |
