# UAT — <SERVICIO> (<NOMBRE DE NEGOCIO>)

<!-- GUÍA: validación orientada al cliente. Un archivo por servicio, desde esta plantilla.
     Los escenarios deben estar automatizados en <servicio>/src/test/resources/features/. -->

> Validación orientada al **cliente/usuario final**. Complementa la certificación técnica.

**Última actualización:** <YYYY-MM-DD> · **Estado:** <en desarrollo | listo para visto bueno>

---

## 1. Criterios de aceptación por requisito funcional (RF)

| RF | Criterio de aceptación (lenguaje de negocio) | Escenario BDD que lo evidencia | Estado | Visto bueno cliente |
|---|---|---|---|---|
| <RF-XX> | <qué debe poder hacer el usuario, en lenguaje llano> | <"nombre del escenario Gherkin"> | <✅/⏳> | ☐ |

## 2. Reporte de avance de negocio (funcionalidades)

| Funcionalidad | Estado | Nota para el cliente |
|---|---|---|
| <funcionalidad> | <🟢 Completo / 🟡 En curso / ⚪ Pendiente> | <nota> |

**Avance del servicio:** <resumen en lenguaje de negocio>.

## 3. Cómo validar hoy
- Lectura y **firma** de la tabla §1 con el cliente.
- **Demo guiada** (Swagger UI / Postman / frontend según disponibilidad).
- Evidencia objetiva: escenarios BDD **verdes en CI**.

> Firma de aceptación: __________________________  Fecha: ____________
