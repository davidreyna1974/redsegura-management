# Memoria técnica de módulo — <NOMBRE DEL MÓDULO>

<!-- GUÍA: documento VIVO. Se inicia con el módulo y se actualiza al cerrar cada fase.
     Es la memoria de "qué se hizo y por qué" del módulo. Las 10 secciones son obligatorias
     (usa N/A donde no aplique). La Sección 7 exige EVIDENCIA verificable, no afirmaciones. -->

**Estado:** <en desarrollo | completo> · **Última actualización:** <YYYY-MM-DD>

## 1. Contexto y justificación
<Por qué existe el módulo, qué problema resuelve, cómo encaja en el sistema.>

## 2. Decisiones de diseño
| Decisión | Alternativas | Motivo |
|---|---|---|
| <...> | <...> | <...> |

## 3. Componentes / interfaces / vistas
<Lista de las unidades implementadas con su responsabilidad. Smart vs dumb / capa a la que pertenecen.>

## 4. Contratos con dependencias
<!-- GUÍA: documentar ANTES de codificar. Nombres de campos EXACTOS. Forma real de la respuesta. -->
| Endpoint/función | Método/firma | Request | Response (campos) | Código |
|---|---|---|---|---|
| <...> | <...> | <...> | <...> | <...> |

## 5. Algoritmos y lógica no trivial
<Describe lo que no se entiende solo con leer el código: cálculos, máquinas de estado, casos borde.>

## 6. Seguridad / RBAC del módulo
- Matriz de acceso de las unidades del módulo por rol.
- Campos sensibles redactados por rol (y dónde se aplica la redacción).

## 7. Ejecución de tests (evidencia verificable)
<!-- GUÍA: pega salidas REALES de comandos. "Pasa todo" no es evidencia. -->
- Por clase/suite: `<cmd test --filtro>` → `<X specs, 0 failures>`.
- Suite completa: `<cmd test>` → `<X specs, 0 failures>`.
- Cobertura: `<cmd coverage>` → `<L/S/B %>` (mínimo <70>%).
- E2E/integración: `<cmd e2e>` → `<X/Y, 0 fallos>`.
- Regresión: pre-módulo `<X/X>` → post-módulo `<X/X>`.

## 8. Bugs y retos durante el desarrollo
<!-- GUÍA: cada bug relevante con ID, causa raíz y fix. Alimenta el registro de lecciones. -->
| ID | Síntoma | Causa raíz | Fix | ¿Lección? |
|---|---|---|---|---|
| BUG-<MOD>-01 | <...> | <...> | <...> | L<nn> |

## 9. Estándares y buenas prácticas aplicadas
<Qué estándares del documento global se respetaron; excepciones justificadas.>

## 10. Cumplimiento y validación (definición de "done")
```
[ ] Todos los casos de prueba en ✅ PASS (documento de casos).
[ ] Gatekeeper en verde (build + tests + lint) y cobertura ≥ <70>%.
[ ] Verificación por rol/condición ejecutada y documentada.
[ ] Gate de seguridad de rutas/endpoints verificado.
[ ] Memoria global actualizada si hubo decisiones transversales.
```
