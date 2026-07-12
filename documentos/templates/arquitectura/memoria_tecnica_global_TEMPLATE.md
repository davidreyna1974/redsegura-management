# Memoria técnica global — <NOMBRE DEL PROYECTO>

<!-- GUÍA: documento maestro del sistema. Es la FUENTE PRIMARIA de contexto arquitectónico.
     Leer al iniciar trabajo en cualquier módulo; actualizar al cerrar cada módulo si hay
     decisiones transversales nuevas. Mantenlo conciso: decisiones y contratos, no tutoriales. -->

**Versión:** <0.1.0> · **Última actualización:** <YYYY-MM-DD> · **Estado:** <en desarrollo | estable · mantenimiento>
<!-- Al cerrar el proyecto: cambiar a "estable · mantenimiento" y enlazar el acta de cierre
     (ver planificacion/acta_cierre_proyecto_TEMPLATE.md). -->

## 1. Visión y alcance

- **Problema que resuelve:** <...>
- **Usuarios / actores:** <...>
- **Alcance (in/out):** dentro: <...> · fuera: <...>
- **Tipo de sistema:** <web full-stack | API | móvil | CLI | datos/IA | ...>

## 2. Decisiones arquitectónicas (ADR resumidas)

<!-- GUÍA: registra el QUÉ y el PORQUÉ de cada decisión clave. Una fila por decisión. -->

| # | Decisión | Alternativas consideradas | Motivo |
|---|---|---|---|
| A1 | <p.ej. arquitectura en capas> | <monolito modular vs micro> | <escala media, simplicidad> |
| A2 | <p.ej. estado con X> | <...> | <...> |
| A3 | <persistencia: BD elegida> | <...> | <...> |

## 3. Arquitectura del sistema

- **Componentes principales:** <cliente · API · BD · workers · servicios externos>.
- **Diagrama:** ver [`diagrama_arquitectura.md`](diagrama_arquitectura.md).
- **Patrón por capa/módulo:** <p.ej. controller → service → repository>.

## 4. Contratos de integración

<!-- GUÍA: clave para evitar bugs de integración. Documenta nombres EXACTOS y formas reales. -->

| Contrato/endpoint | Método/firma | Request (campos) | Response (forma + campos) | Código(s) |
|---|---|---|---|---|
| <GET /recurso> | <...> | <...> | <objeto | colección paginada | void(204)> | <200/4xx> |

Formato de paginación/estándar de respuesta (si aplica):
```json
{ "content": [], "currentPage": 0, "totalPages": 0, "totalElements": 0, "size": 20, "first": true, "last": true }
```

## 5. Seguridad y RBAC

- **Autenticación:** <JWT/OAuth/...>; expiración/renovación: <...>.
- **Matriz de acceso por rol:**

| Módulo / Acción | <ROL_A> | <ROL_B> | <ROL_C> |
|---|:---:|:---:|:---:|
| <Módulo 1 (lectura)> | ✓ | ✓ | — |
| <Módulo 1 (escritura)> | ✓ | — | — |

- **Campos sensibles por rol (redacción server-side):**

| Campo | Sensible para | Valor por rol no autorizado |
|---|---|---|
| <precio/costo/PII> | <ROL_C> | `null` / redactado |

## 6. Configuración y entornos

| Variable | Descripción | Dev | Prod |
|---|---|---|---|
| `<VAR>` | <...> | `<...>` | `<secreto>` |

Entornos: <local / staging / prod>. Puesta en producción: ver `docs/despliegue/`.

### 6.1 CI/CD (Integración y Entrega Continuas)

<!-- GUÍA: documenta el pipeline como es hoy. El CI automatiza el gatekeeper; el CD publica el
     artefacto listo para desplegar. Ver también §CI/CD de CLAUDE.md. -->

| Workflow | Repo/servicio | Dispara en | Qué hace |
|---|---|---|---|
| `ci.yml` | <cada repo> | push/PR a `<develop>`/`main` | build + tests + lint (gatekeeper), sobre entorno limpio |
| `cd.yml` | <cada repo> | push a `main` | build + publicación del artefacto versionado (imagen/paquete) al registry |
| `e2e.yml` | <frontend/servicio> | manual/calendario | E2E de stack completo (aparte del gate de CI) |

- **Servicios del runner:** <p.ej. BD como service container + carga de esquema/datos de referencia>.
- **Artefacto/registry:** <p.ej. imágenes Docker en GHCR, etiquetadas por SHA + `latest`>.
- **Compuerta:** branch protection en `<develop>`/`main` (require PR + require status checks = check de CI).
  ⚠️ En repos **privados de plan gratuito** la protección se crea pero **no se hace cumplir**
  (requiere plan de pago o repo público); complementar con hook `pre-commit` local.
- **Despliegue:** el CD deja el artefacto *listo*, pero el despliegue real es <manual/controlado>
  (Continuous Delivery, no Deployment automático).

### 6.2 Versionado y releases

<!-- GUÍA: cómo se nombran y liberan las versiones. Ver también §Versionado y releases de CLAUDE.md. -->

- **Esquema:** SemVer `MAJOR.MINOR.PATCH`; CHANGELOG (*Keep a Changelog*) actualizado **antes** de tagear.
- **Tags:** anotados (`git tag -a vX.Y.Z`), apuntan al SHA del commit de release en `main`.
  **Inmutables una vez publicados** — nunca `git tag -f`; una corrección sale como versión nueva.
- **Releases (<GitHub/GitLab>):** objeto adjunto al tag con notas legibles; la más reciente = *Latest*.
- **Historial de versiones del proyecto:**

  | Versión | Fecha | Commit (`main`) | Resumen |
  |---|---|---|---|
  | `vX.Y.Z` | <fecha> | `<sha>` | <qué entrega> |

### 6.3 Gobernanza de seguridad de dependencias

<!-- GUÍA: cómo se vigilan dependencias y se reciben reportes. Ver §Gobernanza de seguridad de CLAUDE.md. -->

- **SECURITY.md** en cada repo: canal de reporte (recomendado: *GitHub Private Vulnerability Reporting*).
- **Dependabot** (`.github/dependabot.yml`): SCA server-side + PRs; agrupar minor/patch, majors
  individuales (`@dependabot ignore this major version` para posponer). Cubrir `github-actions`.
- **Escaneo en CI:** rápido en el gate (p.ej. `npm audit`, bloquea en `critical`); pesado (OWASP/NVD) en
  **workflow separado** programado.
- **Sin secretos en git** (ni en el historial); externalizados con `${VAR:default}`.

### 6.4 Operación de producción (Day-2)

<!-- GUÍA: el mínimo para OPERAR el sistema vivo. Ver §Operación de producción de CLAUDE.md y el runbook. -->

- **Backup + restauración probada:** cifrado + off-site + rotación; **RPO/RTO** definidos; **drill** de restore obligatorio.
- **Monitoreo de uptime externo + alerta**; **retención de logs** (no llenar disco).
- **Credenciales:** cambiar las por defecto en el primer arranque; secretos por env + rotación.
- **Confiabilidad:** graceful shutdown (con `stop_grace_period` > timeout de apagado) + restart + límites de recursos.
- **Runbook de operación** lleno (deploy/rollback/restore/monitoreo/diagnóstico). Diferido: métricas, tracing, SLOs, load testing.

## 7. Estándares transversales

Enlace: [`estandares_desarrollo.md`](estandares_desarrollo.md). Reglas que aplican a todas las capas:
<p.ej. estándar de búsqueda de texto, manejo global de errores, paginación, formato de fechas>.

## 8. Estado del proyecto y métricas

- Módulos: <lista con estado>.
- Tests: <X unit · cobertura Y%> · <Z casos de prueba documentados>.
- Última release: <vX.Y.Z — fecha>.

## 9. Registro de lecciones globales

<!-- GUÍA: lecciones transversales (afectan a más de un módulo). Mandatorias desde el diseño. -->

| ID | Lección | Origen | Aplicación |
|---|---|---|---|
| L01 | <regla a futuro> | <bug/decisión> | <dónde aplicarla> |

## 10. Roadmap / pendientes

- [ ] <siguiente módulo/etapa>
- [ ] <deuda técnica conocida>
