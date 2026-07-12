# Documentación general — redSegura

Índice navegable de la documentación general del sistema. Visión general en el
[README principal](../README.md). Estado: ✅ disponible · 🕓 pendiente.

## 🗺️ Planificación
| Documento | Contenido | Estado |
|---|---|---|
| [Plan general del proyecto](planificacion/plan_general_proyecto_redSegura.md) | Secuencia de tareas, contratos, criterios de éxito, hoja de ruta, ADR. | ✅ |
| [Especificación del proyecto](proyecto_microservicios_redsegura.md) | Fuente de verdad: RF/RNF, RBAC, arquitectura, alcance. | ✅ |

## 🏗️ Arquitectura y decisiones
| Documento | Contenido | Estado |
|---|---|---|
| [Memoria técnica global](arquitectura/memoria_tecnica_global.md) | Visión, decisiones (ADR-01..04), contratos, RBAC, lecciones. | ✅ |
| [Diagrama de arquitectura](arquitectura/diagrama_arquitectura.md) | Diagramas Mermaid (capas, eventos, golden path, despliegue). | ✅ |
| [Estándares de desarrollo](arquitectura/estandares_desarrollo.md) | Convenciones por stack (Java/Python/Angular), logging, pruebas. | ✅ |
| [Comunicación por eventos](arquitectura/especificaciones/comunicacion_por_eventos.md) | Topología RabbitMQ y esquema JSON de cada evento. | ✅ |

## ✅ Calidad (QA)
| Documento | Contenido | Estado |
|---|---|---|
| [Protocolo de 4 fases](qa/protocolo_verificacion_4_fases.md) | Metodología de QA global. | ✅ |
| [Reporte de QA](qa/reporte_qa.md) | Resultado consolidado de certificación por servicio. | 🕓 |

## 📦 Despliegue
| Documento | Contenido | Estado |
|---|---|---|
| [Plan de salida a producción](despliegue/plan_salida_produccion.md) | Checklist de puesta en producción. | 🕓 |
| [Runbook de despliegue y operación](despliegue/runbook_despliegue.md) | Despliegue, operación y diagnóstico. | 🕓 |

## 📖 Guías
| Documento | Contenido | Estado |
|---|---|---|
| [Guía rápida de usuario](guias/guia_rapida_usuario.md) | Uso funcional del dashboard. | 🕓 |

## 🧰 Otros
| Recurso | Contenido |
|---|---|
| [Kit de plantillas](templates/README.md) | Plantillas reutilizables de documentación. |
| [Anteproyecto](anteproyecto_microservicios.md) · [Prácticas base](especificaciones_diseño.md) | Contexto histórico del diseño. |
| [Sesiones](sesiones/) | Bitácora de continuidad entre sesiones. |

---
> **Convención:** ante cualquier cambio se aplica el
> [Protocolo de 4 fases](qa/protocolo_verificacion_4_fases.md) y se actualiza la memoria técnica
> global + las lecciones. Contexto para Claude Code: [`../../backend/CLAUDE.md`](../../backend/CLAUDE.md).
