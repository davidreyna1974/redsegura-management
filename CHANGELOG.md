# Changelog — redSegura (management)

Todos los cambios notables de este repositorio se documentan en este archivo.

Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/);
versionado según [SemVer](https://semver.org/lang/es/).

## [No publicado]
### Añadido
- **Checkpoint de pausa prolongada (2026-08-11):** documentos de sesión en `documentos/sesiones/`
  — `contexto_sesion_siguiente.md` (handoff ejecutivo maestro: avance, punto exacto de retomada,
  bifurcación de arranque) y `estado_sesion_activa.md` (tablero vivo detallado). Actualizados
  `plan_general_proyecto_redSegura.md` (v1.3.0) y `qa/reporte_qa.md` con el estado de pausa y el
  plan de validación de dispositivos (Fase 0). Template `sesiones/contexto_sesion_siguiente_TEMPLATE.md`
  con ruta de bitácora consistente (`documentos/sesiones/`).
- Inicialización del repositorio `management` con git-hook `pre-commit` (bloquea commits
  directos a `main`/`develop`) y `.gitignore`.
- Reorganización documental: la documentación general del sistema se consolida bajo
  `documentos/` (Especificación, anteproyecto, prácticas base, kit de plantillas).
- Plan general del proyecto (`documentos/planificacion/plan_general_proyecto_redSegura.md`).
- Arquitectura global: memoria técnica global, diagrama de arquitectura (Mermaid), estándares de
  desarrollo (Java/Python/Angular + logging) y especificación de comunicación por eventos.
- Decisiones de arquitectura registradas: ADR-01 (almacenamiento Git+PostgreSQL de respaldos),
  ADR-02 (running/startup + `unsavedChanges`), ADR-03 (canales de notificación enchufables),
  ADR-04 (transactional outbox), ADR-05 (contract-first Java / code-first + check Python),
  ADR-06 (MapStruct / Pydantic para entidad↔DTO), ADR-07 (columnas de auditoría estándar),
  ADR-08 (RFC 7807 Problem Details + JSON Merge Patch), ADR-09 (Idempotency-Key + ETag/If-Match),
  ADR-10 (health probes liveness/readiness), ADR-11 (redacción por rol + log de auditoría de
  seguridad), ADR-12 (gobernanza de contratos con Spectral en CI).
- Protocolo de verificación en 4 fases (metodología de QA global).
- Documentación del repositorio: README raíz del sistema, SECURITY, plantilla de PR, índice de
  documentación.
- Repositorios remotos publicados en GitHub (`redsegura-management`, `redsegura-backend`,
  públicos) con branch protection (`main` requiere PR; `develop` con historial protegido).
