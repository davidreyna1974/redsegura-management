# Changelog — redSegura (management)

Todos los cambios notables de este repositorio se documentan en este archivo.

Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/);
versionado según [SemVer](https://semver.org/lang/es/).

## [No publicado]
### Añadido
- Inicialización del repositorio `management` con git-hook `pre-commit` (bloquea commits
  directos a `main`/`develop`) y `.gitignore`.
- Reorganización documental: la documentación general del sistema se consolida bajo
  `documentos/` (Especificación, anteproyecto, prácticas base, kit de plantillas).
- Plan general del proyecto (`documentos/planificacion/plan_general_proyecto_redSegura.md`).
- Arquitectura global: memoria técnica global, diagrama de arquitectura (Mermaid), estándares de
  desarrollo (Java/Python/Angular + logging) y especificación de comunicación por eventos.
- Decisiones de arquitectura registradas: ADR-01 (almacenamiento Git+PostgreSQL de respaldos),
  ADR-02 (running/startup + `unsavedChanges`), ADR-03 (canales de notificación enchufables),
  ADR-04 (transactional outbox).
- Protocolo de verificación en 4 fases (metodología de QA global).
- Documentación del repositorio: README raíz del sistema, SECURITY, plantilla de PR, índice de
  documentación.
