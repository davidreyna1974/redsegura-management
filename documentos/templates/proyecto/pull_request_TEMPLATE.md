<!-- GUÍA: copia este archivo a .github/pull_request_template.md en el repo. GitHub/GitLab lo
     cargan automáticamente al abrir un PR. Ajusta los comandos del gatekeeper a tu proyecto. -->
<!-- Plantilla de Pull Request — <NOMBRE DEL PROYECTO> -->

## Descripción
<!-- ¿Qué cambia y por qué? -->

## Tipo de cambio
- [ ] `feature/` — nueva funcionalidad o módulo
- [ ] `fix/` — corrección
- [ ] `chore/` — infraestructura, configuración o documentación

## Checklist (Protocolo del repositorio)
- [ ] La rama parte de `<develop>` y se integra vía `merge --no-ff` (nunca commit directo a `main`/`<develop>`).
- [ ] `<cmd build>` → 0 errores.
- [ ] `<cmd test>` → 0 fallos.
- [ ] `<cmd lint/type>` → 0 errores; cobertura ≥ <70>%.
- [ ] Puntos de entrada nuevos con autorización explícita por rol (gate de seguridad).
- [ ] Datos sensibles redactados server-side para roles no autorizados.
- [ ] Casos de prueba (`docs/qa/`) y memoria técnica del módulo (`docs/modulos/`) actualizados.
- [ ] Sin datos de prueba residuales; sin secretos en el diff.

## Evidencia de pruebas
<!-- Pega el resumen de tests / verificación relevante -->
