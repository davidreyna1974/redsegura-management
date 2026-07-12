# Acta de cierre de proyecto — <NOMBRE DEL PROYECTO>

<!-- GUÍA: formaliza el cierre del proyecto (PMBOK "Close Project or Phase" + prácticas de release).
     Una sola página que consolida qué se entregó, la calidad, las versiones, las brechas resueltas/
     diferidas, la retrospectiva y el trabajo futuro. Es lo que un revisor/auditor/portafolio busca.
     Ubicación sugerida: docs/ del repo principal (o del repo paraguas si es multi-repo). -->

**Proyecto:** <nombre> · **Versión de cierre:** <vX.Y.Z> · **Fecha de cierre:** <YYYY-MM-DD>
**Estado:** ✅ **Completado · estable · en modo mantenimiento** (no archivado)
**Autor/es:** <...> · **Licencia:** <MIT / ...>

---

## 1. Resumen ejecutivo
<2-4 líneas: qué es el sistema, para quién, y declaración de que se da por cerrado en <vX.Y.Z>,
estable y documentado, en modo mantenimiento.>

## 2. Entregables
- <Repositorios / componentes entregados.>
- <Módulos de negocio / capacidades principales.>
- <Seguridad transversal (p. ej. RBAC), DevOps (CI/CD, despliegue), documentación.>

## 3. Calidad / certificación
- <N tests automatizados · 0 fallos · cobertura %.>
- <N casos de prueba · módulos certificados · protocolo aplicado (p. ej. 4 fases / Propuesta D).>
- <0 bugs funcionales · 0 regresiones · verificación de seguridad.>

## 4. Versiones liberadas (SemVer + tags inmutables)
| Versión | Fecha | Resumen |
|---|---|---|
| `vX.Y.Z` | <fecha> | <qué entregó> |

## 5. Cierre de brechas de madurez (estándar de industria)
<!-- Evaluar contra: SRE PRR, DORA, OWASP ASVS, 12-Factor, Well-Architected. Marcar cada una. -->
| # | Brecha | Estado |
|---|---|---|
| 1 | CI/CD | <✅ / ⏳ diferida / N/A> |
| 2 | Versionado / releases (SemVer, tags, Releases) | <...> |
| 3 | CHANGELOG (Keep a Changelog) | <...> |
| 4 | Gobernanza de seguridad (SECURITY.md, SCA) | <...> |
| 5 | Archivos de comunidad (CONTRIBUTING, CODE_OF_CONDUCT) | <...> |
| 6 | Operación de producción (Day-2 ops: backup/restore, monitoreo, runbook) | <...> |

## 6. Retrospectiva / lecciones aprendidas
- **Qué funcionó bien:** <p. ej. respuesta sistémica a bugs; verificación sobre código congelado; memoria viva.>
- **Qué se detectó tarde (área de oportunidad):** <p. ej. prácticas de madurez incorporadas al cierre.>
- **Acción derivada:** <p. ej. destilar prácticas base para el próximo proyecto; reforzar templates.>
- **Lecciones técnicas consolidadas:** <enlace al registro de lecciones / §lecciones de la memoria global.>

## 7. Trabajo futuro (backlog diferido)
- <Brechas diferidas.>
- <Módulos/funcionalidades futuras (del roadmap).>
- <Mejoras opcionales.>

## 8. Estado y modo de mantenimiento
- El proyecto se declara **cerrado en <vX.Y.Z>**, **estable** y **documentado**.
- **Modo mantenimiento** (no archivado): ante cambios nuevos se aplica el protocolo de calidad y se
  actualizan memoria + CHANGELOG; el trabajo diferido se toma del backlog (§7).
- **No se archiva** el repositorio (archivar = solo lectura; señala abandono e impide actualizar).

## 9. Referencias
- <Reporte de QA · Memoria técnica global (estado/lecciones/roadmap) · Runbook · Planes de implementación.>
