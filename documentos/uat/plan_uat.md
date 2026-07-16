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
| **Demo guiada** | Ver el software funcionando de extremo a extremo | Sesión en vivo con **colección Postman** (guion en `<servicio>/demo_guiada.md`) hoy; **frontend** cuando exista |

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

| Servicio | UAT | Artefactos (carpeta `<servicio>/`) |
|---|---|---|
| `asset-inventory-service` | Escenarios BDD automatizados ✅; criterios por RF + demo guiada | [`asset-inventory-service/`](asset-inventory-service/) — [guion_uat.md](asset-inventory-service/guion_uat.md) · [demo_guiada.md](asset-inventory-service/demo_guiada.md) |
| resto | pendiente (al desarrollar cada servicio) | `<servicio>/{guion_uat.md, demo_guiada.md}` (desde `../templates/uat/`) |

## 5. Nota sobre el momento actual
El `frontend` aún no existe, así que la **demo plena** para un cliente no-técnico llegará con la UI.
Hoy la validación se apoya en: (a) **criterios de aceptación por RF** con firma, (b) **escenarios
Gherkin** aprobables y ya **automatizados**, y (c) una **demo guiada** en vivo con la **colección
Postman** (guion paso a paso en cada `<servicio>/demo_guiada.md`).

> Plantillas ([`../templates/uat/`](../templates/uat/)): `plan_uat_TEMPLATE.md`,
> `uat_servicio_TEMPLATE.md` (→ `<servicio>/guion_uat.md`) y `demo_guiada_TEMPLATE.md`
> (→ `<servicio>/demo_guiada.md`).
