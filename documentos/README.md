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
| [Memoria técnica global](arquitectura/memoria_tecnica_global.md) | Visión, decisiones (ADR-01..17), contratos, RBAC, lecciones, estado. | ✅ |
| [Diagrama de arquitectura](arquitectura/diagrama_arquitectura.md) | Diagramas Mermaid (capas, eventos, golden path, despliegue). | ✅ |
| [Estándares de desarrollo](arquitectura/estandares_desarrollo.md) | Convenciones por stack (Java/Python/Angular), logging, pruebas, endurecimiento a producción. | ✅ |
| [Preparación a producción](arquitectura/preparacion_produccion.md) | Checklist de endurecimiento (RNF-31): gaps obligatorios-diferidos con disparador por etapa/servicio. | ✅ |
| [Comunicación por eventos](arquitectura/especificaciones/comunicacion_por_eventos.md) | Topología RabbitMQ y esquema JSON de cada evento. | ✅ |

## ✅ Calidad (QA) y aceptación (UAT)
| Documento | Contenido | Estado |
|---|---|---|
| [Estrategia de pruebas](qa/estrategia_de_pruebas.md) | Capas de test por servicio (Testcontainers, conformidad de contrato, BDD, **verificación en vivo de endpoints**), contratos compartidos, consideraciones por servicio. | ✅ |
| [Protocolo de 4 fases](qa/protocolo_verificacion_4_fases.md) | Metodología de QA técnica. | ✅ |
| [Reporte de QA](qa/reporte_qa.md) | Resultado consolidado de certificación por servicio (`asset-inventory` R1.2 ✅). | ✅ |
| [**UAT — validación con el cliente**](uat/README.md) | Paquete de aceptación consolidado: plan general + por servicio (guion de criterios/firma + **demo guiada**). Reporte técnico de verificación en vivo vive en `backend/<servicio>/documentos/`. | ✅ |

## 📦 Despliegue
| Documento | Contenido | Estado |
|---|---|---|
| Plan de salida a producción (plantilla: [`templates/despliegue/plan_salida_produccion_TEMPLATE.md`](templates/despliegue/plan_salida_produccion_TEMPLATE.md)) | Checklist de puesta en producción. Verifica la [preparación a producción](arquitectura/preparacion_produccion.md). | 🕓 se genera en **DEPLOY** |
| Runbook de despliegue y operación (plantilla: [`templates/despliegue/runbook_despliegue_TEMPLATE.md`](templates/despliegue/runbook_despliegue_TEMPLATE.md)) | Despliegue, operación y diagnóstico. | 🕓 se genera en **DEPLOY** |

## 📖 Guías
| Documento | Contenido | Estado |
|---|---|---|
| Guía rápida de usuario (plantilla: [`templates/proyecto/guia_rapida_usuario_TEMPLATE.md`](templates/proyecto/guia_rapida_usuario_TEMPLATE.md)) | Uso funcional del dashboard. | 🕓 con el `frontend` |

## 🧰 Otros
| Recurso | Contenido |
|---|---|
| [Kit de plantillas](templates/README.md) | Plantillas reutilizables de documentación. |
| [Anteproyecto](anteproyecto_microservicios.md) · [Prácticas base](especificaciones_diseño.md) | Contexto histórico del diseño. |
| Sesiones (plantilla: [`templates/sesiones/`](templates/sesiones/)) | Bitácora de continuidad entre sesiones. 🕓 |

---
> **Convención:** ante cualquier cambio se aplica el
> [Protocolo de 4 fases](qa/protocolo_verificacion_4_fases.md) y se actualiza la memoria técnica
> global + las lecciones. Contexto para Claude Code: [`../../backend/CLAUDE.md`](../../backend/CLAUDE.md).
