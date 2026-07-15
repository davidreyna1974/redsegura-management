# 🧰 Kit de plantillas de documentación para proyectos de software

Biblioteca de **plantillas reutilizables** (`*_TEMPLATE.*`) para arrancar la documentación,
los estándares y el contexto de Claude Code de cualquier proyecto de software nuevo —
**agnósticas de tecnología** y basadas en buenas prácticas.

> Origen: destiladas de un sistema real (full-stack web) llevado a producción y certificado
> con un protocolo de QA de 4 fases. Aquí queda **el método**, sin lo particular de aquel proyecto.

---

## 🎯 Para qué sirve

- Iniciar un proyecto con documentación profesional desde el commit 0.
- Dar a Claude Code un **contexto fuerte y consistente** (`CLAUDE.md`) en cada repo.
- Reutilizar una **metodología de calidad** probada (protocolo de 4 fases, gatekeeper, blast radius).
- Transmitir orden y madurez a cualquiera que revise el repositorio (equipo, cliente, portafolio).

---

## 🗂️ Contenido del kit

| Carpeta | Plantillas | Cuándo se usa |
|---|---|---|
| `contexto/` | `CLAUDE_TEMPLATE.md`, `estandares_desarrollo_TEMPLATE.md` | Al crear el repo (pieza central para Claude Code) |
| `arquitectura/` | `memoria_tecnica_global_TEMPLATE.md`, `diagrama_arquitectura_TEMPLATE.md`, `especificacion_tecnica_TEMPLATE.md` | Al definir la arquitectura del sistema |
| `modulos/` | `propuesta_modulo_TEMPLATE.md`, `memoria_tecnica_modulo_TEMPLATE.md` | Antes y durante cada módulo/feature |
| `qa/` | `casos_de_prueba_TEMPLATE.md`, `protocolo_verificacion_4_fases_TEMPLATE.md`, `reporte_qa_TEMPLATE.md`, `analisis_pruebas_TEMPLATE.md`, `verificacion_endpoints_TEMPLATE.md` | Para definir, ejecutar y certificar pruebas (incl. la verificación en vivo de endpoints por servicio) |
| `planificacion/` | `propuesta_proyecto_TEMPLATE.md`, `plan_trabajo_TEMPLATE.md`, `acta_cierre_proyecto_TEMPLATE.md` | Al inicio del proyecto / de una etapa, y al **cierre formal** |
| `despliegue/` | `plan_salida_produccion_TEMPLATE.md`, `runbook_despliegue_TEMPLATE.md` | Antes de salir a producción y para operar |
| `sesiones/` | `estado_sesion_activa_TEMPLATE.md`, `contexto_sesion_siguiente_TEMPLATE.md` | Para no perder contexto entre sesiones de trabajo |
| `proyecto/` | `README_TEMPLATE.md`, `CHANGELOG_TEMPLATE.md`, `pull_request_TEMPLATE.md`, `docs_indice_TEMPLATE.md`, `guia_rapida_usuario_TEMPLATE.md` | Higiene y presentación del repositorio |
| `ejemplos/` | `ejemplo_fullstack_web.md`, `ejemplo_api_servicio.md`, `ejemplo_datos_ia_python.md` | Muestras concretas resueltas por tipo de proyecto |

---

## 🚀 Cómo arrancar un proyecto nuevo con este kit

> Marca cada paso. Los primeros 4 bastan para empezar a programar con buen contexto.

```
[ ] 1. Crea el repo y copia la documentación base:
        - contexto/CLAUDE_TEMPLATE.md            → <repo>/CLAUDE.md
        - contexto/estandares_desarrollo_TEMPLATE.md → <repo>/docs/arquitectura/estandares_desarrollo.md
        - proyecto/README_TEMPLATE.md            → <repo>/README.md
        - proyecto/docs_indice_TEMPLATE.md       → <repo>/docs/README.md
        - proyecto/CHANGELOG_TEMPLATE.md         → <repo>/CHANGELOG.md
        - proyecto/pull_request_TEMPLATE.md      → <repo>/.github/pull_request_template.md
[ ] 2. Rellena en CLAUDE.md: stack, comandos de build/test/lint (gatekeeper), convenciones git, RBAC.
[ ] 3. Copia el anexo de ejemplo más cercano (ejemplos/) y úsalo como referencia de relleno.
[ ] 4. Crea la arquitectura: arquitectura/memoria_tecnica_global_TEMPLATE.md + diagrama_arquitectura_TEMPLATE.md.
[ ] 5. Por cada módulo/feature, ANTES de codificar:
        - modulos/propuesta_modulo_TEMPLATE.md
        - qa/casos_de_prueba_TEMPLATE.md
        - modulos/memoria_tecnica_modulo_TEMPLATE.md (documento vivo)
[ ] 6. Para verificar: qa/protocolo_verificacion_4_fases_TEMPLATE.md → qa/reporte_qa_TEMPLATE.md.
[ ] 7. Para producción: despliegue/plan_salida_produccion_TEMPLATE.md + runbook_despliegue_TEMPLATE.md.
[ ] 8. Mantén el contexto entre sesiones: sesiones/estado_sesion_activa_TEMPLATE.md.
[ ] 9. Al cerrar el proyecto (formalización): planificacion/acta_cierre_proyecto_TEMPLATE.md +
        declarar estado "estable · mantenimiento" en README y memoria global (NO archivar el repo).
```

---

## 🧩 Convenciones de los templates

- `<TEXTO>` — **placeholder obligatorio**: reemplázalo antes de usar el documento.
- `[opcional: ...]` — sección/dato opcional; bórralo si no aplica.
- `<!-- GUÍA: ... -->` — nota de buenas prácticas (explica el porqué). **Bórrala** al finalizar el documento real.
- `N/A` — usar explícitamente cuando algo no aplica (mejor que dejarlo en blanco).
- Idioma: español. Diagramas: [Mermaid](https://mermaid.js.org/) (se renderizan en GitHub/GitLab).

---

## 📌 Principios que atraviesan todo el kit (agnósticos)

1. **Documentar antes de codificar** un módulo (propuesta + casos de prueba + memoria técnica).
2. **Gatekeeper por cada cambio**: build + tests + lint/type-check en verde, sin excepción.
3. **Una ronda de pruebas es válida sólo sobre código congelado** (no corregir a mitad de ronda).
4. **Blast radius**: todo cambio declara qué alcance tiene (local vs global) y qué re-probar.
5. **Seguridad por defecto**: rutas/endpoints con autorización explícita; datos sensibles redactados por rol en el servidor, no sólo ocultos en el cliente.
6. **Evidencia verificable**, no afirmaciones: pegar salidas reales de comandos y resultados.
7. **Contexto persistente** para Claude Code: `CLAUDE.md` al día + estado de sesión.

---

<sub>Kit de plantillas de documentación · uso libre en tus proyectos · 2026</sub>
