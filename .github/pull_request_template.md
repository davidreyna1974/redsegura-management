<!-- Plantilla de Pull Request — redSegura (management) -->

## Descripción
<!-- ¿Qué cambia y por qué? -->

## Tipo de cambio
- [ ] `feature/` — nueva funcionalidad (documentación, orquestación, infraestructura)
- [ ] `fix/` — corrección
- [ ] `chore/` — infraestructura, configuración o mantenimiento

## Checklist (Protocolo del repositorio)
- [ ] La rama parte de `develop` y se integra vía `merge --no-ff` (nunca commit directo a `main`/`develop`).
- [ ] Documentos sin placeholders (`<...>`) ni notas de guía (`<!-- GUÍA -->`) residuales.
- [ ] Enlaces internos de la documentación verificados (no rotos).
- [ ] Diagramas Mermaid con sintaxis válida (renderizan).
- [ ] Si aplica a infraestructura: `terraform validate` / `docker compose config` en verde.
- [ ] Sin secretos en el diff; sin credenciales en archivos de configuración.
- [ ] Decisiones transversales reflejadas en la memoria técnica global; lecciones registradas.
- [ ] CHANGELOG (`[No publicado]`) actualizado.

## Evidencia
<!-- Pega el resumen de validación relevante (lint de OpenAPI, terraform validate, etc.) -->
