# Plan de UAT (User Acceptance Testing) — redSegura

> **Propósito:** cómo el **cliente/usuario final** (no técnico) valida las funcionalidades, entiende
> el avance y otorga su **visto bueno**. Es distinto de los tests técnicos (que verifican *que el
> código funciona*): la UAT verifica *que el sistema hace lo que el negocio necesita*.

**Última actualización:** 2026-07-15

---

## 1. Instrumentos de la UAT

| Instrumento | Para qué | Formato para el cliente |
|---|---|---|
| **Criterios de aceptación por RF** | Cada requisito funcional en lenguaje de negocio, con resultado esperado y **firma** | Tabla RF → criterio → resultado → estado → visto bueno |
| **Escenarios BDD / Gherkin** (Dado–Cuando–Entonces) | Casos concretos que el cliente **lee y aprueba**; están **automatizados** (Cucumber) y corren en CI | Texto casi natural en español |
| **Reporte de avance de negocio** | Estado de **funcionalidades** (no de tests) y % de avance por servicio/fase | Semáforo + % + próximos hitos |
| **Demo guiada** | Ver el software funcionando de extremo a extremo | Sesión en vivo: Swagger UI / Postman hoy; **frontend** cuando exista |

## 2. Proceso de aceptación

1. **Definir criterios de aceptación por RF** antes o durante el desarrollo del servicio (lenguaje de
   negocio), acordados con el cliente.
2. **Escribir los escenarios Gherkin** que ejemplifican cada criterio → el cliente los **aprueba**.
3. **Automatizarlos** (Cucumber/behave) → corren en el CI como pruebas de aceptación (evidencia
   objetiva y repetible de que el criterio se cumple).
4. **Demo/validación** con el cliente sobre un entorno de demostración; el cliente marca **PASS/FAIL**
   y **firma** los criterios aceptados.
5. **Reporte de avance de negocio** actualizado por hito (para dirección/cliente).

## 3. Relación con los tests técnicos

- Los **escenarios BDD** son el **puente**: negocio los lee/aprueba, ingeniería los ejecuta. Viven en
  `<servicio>/src/test/resources/features/*.feature` y se verifican en cada push (CI).
- La **certificación QA de 4 fases** (técnica) y la **UAT** (negocio) son complementarias: la primera
  garantiza calidad interna; la segunda, aceptación del cliente.

## 4. Estado y artefactos por servicio

| Servicio | UAT | Artefacto |
|---|---|---|
| `asset-inventory-service` | Escenarios BDD automatizados ✅; criterios por RF documentados | [`uat_asset-inventory-service.md`](uat_asset-inventory-service.md) |
| resto | pendiente (al desarrollar cada servicio) | `uat_<servicio>.md` (desde plantilla) |

## 5. Nota sobre el momento actual
El `frontend` aún no existe, así que la **demo plena** para un cliente no-técnico llegará con la UI.
Hoy la validación se apoya en: (a) **criterios de aceptación por RF** con firma, (b) **escenarios
Gherkin** aprobables y ya **automatizados**, y (c) para un stakeholder semi-técnico, una **demo
guiada** vía Swagger UI / colección Postman.

> Plantillas: [`../templates/uat/plan_uat_TEMPLATE.md`](../templates/uat/plan_uat_TEMPLATE.md) y
> [`../templates/uat/uat_servicio_TEMPLATE.md`](../templates/uat/uat_servicio_TEMPLATE.md).
