# Plan de UAT (User Acceptance Testing) — <NOMBRE DEL PROYECTO>

<!-- GUÍA: metodología de aceptación con el cliente. Uno por proyecto. Las instancias por servicio
     usan uat_servicio_TEMPLATE.md. -->

> Cómo el **cliente/usuario final** valida las funcionalidades y otorga su **visto bueno**. Distinto
> de los tests técnicos: la UAT verifica que el sistema hace lo que el **negocio** necesita.

**Última actualización:** <YYYY-MM-DD>

## 1. Instrumentos
| Instrumento | Para qué |
|---|---|
| **Criterios de aceptación por RF** | Cada requisito en lenguaje de negocio, con firma |
| **Escenarios BDD / Gherkin** | Casos aprobables por el cliente y **automatizados** (Cucumber/behave) |
| **Reporte de avance de negocio** | Estado de funcionalidades y % por servicio/fase |
| **Demo guiada** | Software funcionando (Swagger/Postman → frontend) |

## 2. Proceso
1. Definir criterios de aceptación por RF (con el cliente).
2. Escribir escenarios Gherkin → el cliente los aprueba.
3. Automatizarlos → corren en CI.
4. Demo/validación → PASS/FAIL + firma.
5. Reporte de avance por hito.

## 3. Artefactos por servicio
| Servicio | UAT | Artefacto |
|---|---|---|
| <servicio> | <estado> | `uat_<servicio>.md` |
