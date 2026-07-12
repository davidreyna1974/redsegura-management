# Análisis de pruebas — <ALCANCE: módulo / capa / tipo de test>

<!-- GUÍA: documento de trabajo para planear o diagnosticar la suite de pruebas (no es el reporte final).
     Útil para: decidir qué probar y cómo, analizar fallos/flaky, o auditar cobertura por capa. -->

**Fecha:** <YYYY-MM-DD> · **Alcance:** <unit servicios / integración controllers / E2E / curl seguridad>

## 1. Objetivo del análisis

<Qué se quiere lograr: diseñar la suite, cerrar huecos de cobertura, estabilizar tests, etc.>

## 2. Inventario de lo que se prueba

| Unidad bajo prueba | Tipo de test | Casos previstos | Estado |
|---|---|---|---|
| <Servicio/Controller/Flujo> | <unit/integración/e2e> | <happy + bordes + errores> | <pendiente/listo> |

## 3. Estrategia por capa

- **Unit:** aislar con dobles de prueba; verificar lógica y ramas (incluido error).
- **Integración:** componente + dependencia real (BD en memoria/Testcontainers); contratos.
- **E2E / seguridad:** flujo completo y enforcement server-side por rol (`<curl/cliente>` con credencial por rol).

## 4. Matriz de cobertura objetivo

| Capa | Cobertura actual | Objetivo | Hueco principal |
|---|---|---|---|
| <servicios> | <%> | ≥ <70>% | <...> |
| <controllers> | <%> | ≥ <70>% | <...> |

## 5. Hallazgos

| # | Hallazgo (hueco/fallo/flaky) | Causa | Acción |
|---|---|---|---|
| 1 | <...> | <...> | <...> |

## 6. Casos borde y de seguridad a no olvidar

- Entrada nula/vacía/límite; tipos inválidos; colección vacía vs error.
- Autorización: rol sin permiso (403), sin credencial (401), token manipulado.
- Concurrencia/estado: transiciones inválidas de la máquina de estados.
- Redacción de campos sensibles por rol (verificar en la respuesta real).

## 7. Conclusiones y plan

- [ ] <acción 1>
- [ ] <acción 2>
