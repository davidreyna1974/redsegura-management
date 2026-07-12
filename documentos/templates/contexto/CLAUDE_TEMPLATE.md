# CLAUDE.md — <NOMBRE DEL PROYECTO>

<!-- GUÍA: Este archivo es el CONTEXTO PRIMARIO para Claude Code en cada sesión. Mantenlo al día.
     Las instrucciones aquí SOBRESCRIBEN el comportamiento por defecto del asistente.
     Sé conciso y operativo: comandos exactos, rutas reales, reglas no negociables. -->

Guía para Claude Code (claude.ai/code) al trabajar en este repositorio.

## Descripción del proyecto

<DESCRIPCIÓN EN 2-4 LÍNEAS: qué hace el sistema, para quién, y su alcance.>

**Stack tecnológico:**
<!-- GUÍA: lista SOLO lo que realmente usas. Aplica a cualquier tipo de proyecto. -->
- Tipo de proyecto: <web full-stack | API/servicio | móvil | CLI | datos/IA | librería>
- Lenguaje(s): <p.ej. TypeScript / Java / Python / Go>
- Framework(s): <p.ej. Angular / Spring Boot / FastAPI / React Native / N/A>
- Persistencia: <p.ej. PostgreSQL / MongoDB / archivos / N/A>
- Build / gestor de paquetes: <p.ej. npm / Maven / uv|pip / cargo>
- Tests: <unit: p.ej. Vitest/JUnit/pytest> · <e2e/integración: p.ej. Playwright/Testcontainers>
- Lint / formato / tipos: <p.ej. eslint+prettier / ruff+mypy / checkstyle>
- CI/CD: <p.ej. GitHub Actions / GitLab CI / N/A>

**Integraciones / contratos externos:** [opcional]
- API/base URL: <http://localhost:<puerto>/...>
- Documentación de contratos: <Swagger/OpenAPI URL · proto · esquema>
- Autenticación: <JWT/OAuth/API key>; expiración/renovación: <...>

---

## ⚙️ Comandos del proyecto (gatekeeper)

<!-- GUÍA: el "gatekeeper" es la barrera NO NEGOCIABLE que corre por CADA cambio antes de
     considerarlo terminado. Define los 3 comandos reales. Si el build estricto difiere del
     runner de tests, INCLUYE el build aparte (no lo des por implícito). -->

| Paso | Comando | Criterio de aprobación |
|---|---|---|
| 1. Build / compilación | `<cmd build>` | 0 errores |
| 2. Tests | `<cmd test>` | 0 fallos |
| 3. Lint / type-check | `<cmd lint/types>` | 0 errores |
| Cobertura | `<cmd coverage>` | ≥ <70>% statements |

Otros comandos: ejecutar `<cmd run>` · formato `<cmd format>`.

> Este gatekeeper lo ejecuta **automáticamente el CI** en cada `push`/`pull_request` (ver §CI/CD).
> Correrlo en local antes de pushear separa "¿falla mi código?" de "¿falla el CI (entorno limpio)?".

---

## ⚠️ Convenciones de Git — REGLAS CRÍTICAS

**NUNCA commitear directamente en `main` o `<rama de integración, p.ej. develop>`.**
Todo trabajo va en una rama `feature/` o `fix/` y se integra vía `git merge --no-ff`.

```bash
git checkout <develop>
git checkout -b feature/<nombre>
# ... trabajar y commitear ...
git checkout <develop>
git merge --no-ff feature/<nombre>
git push origin <develop>
```

| Prefijo | Cuándo |
|---|---|
| `feature/<nombre>` | Módulo nuevo o funcionalidad |
| `fix/<nombre>` | Corrección post-merge |
| `chore/<nombre>` | Infraestructura, configuración, documentación |

- Mensajes de commit: <convención, p.ej. Conventional Commits `tipo(scope): mensaje`>.
- Releases: `<develop> → main` es una RELEASE → SemVer + tag `vX.Y.Z` + CHANGELOG
  (ver §Versionado y releases — los tags publicados son **inmutables**).

---

## 🔁 CI/CD (Integración y Entrega Continuas)

<!-- GUÍA: el CI automatiza el gatekeeper en CADA cambio; el CD publica el artefacto listo para
     desplegar. Reemplaza "correr los comandos a mano" por una compuerta repetible. Aplica a
     cualquier tipo de proyecto (ajusta los nombres de workflow/artefacto). -->

**CI (Integración Continua):** en cada `push`/`pull_request` a `<develop>`/`main`, un workflow
(`<GitHub Actions / GitLab CI>`, en `.github/workflows/ci.yml`) ejecuta el **gatekeeper**
(build + tests + lint) sobre el commit. Reemplaza el gatekeeper manual por uno automático,
repetible y en un **entorno limpio** (donde salen a la luz dependencias que la máquina de dev
daba "gratis": una BD, datos de referencia, servicios). Badge de estado en el README.
- Si los tests necesitan una dependencia (BD, cache), levántala como **service container** y
  siembra el esquema/datos de referencia obligatorios en el propio workflow.

**E2E / integración pesada en workflow APARTE.** Los E2E (stack completo, lentos) NO van en el
gate de cada push: workflow separado (`e2e.yml`), disparado a mano/por calendario, para no frenar
el feedback rápido del CI.

**CD (Entrega Continua):** al llegar a `main`, un workflow construye y **publica** el artefacto
versionado (imagen Docker a un registry / paquete), etiquetado por **SHA** (inmutable) + `latest`.
Queda *listo para desplegar*. NO desplegar automáticamente salvo decisión explícita
(Continuous Delivery ≠ Deployment): el despliegue real queda controlado.

**La compuerta (branch protection).** Para que el CI sea una compuerta real y no sólo informativa:
proteger `<develop>`/`main` con *"require pull request"* + *"require status checks"* (seleccionar
el check del CI). Así un PR con el CI en rojo **no se puede mergear**. Dos matices:
- El `git push` **nunca** espera al CI: el push ocurre y el CI corre **después**. El candado vive
  en el **merge del PR**, no en el push directo.
- En repos **privados de plan gratuito** la protección se **crea pero no se hace cumplir** (requiere
  plan de pago o repo público). Complementa con un **hook `pre-commit` local** que bloquee commits
  directos a `<develop>`/`main` en la máquina (defensa en 2 capas: local + servidor).

**Vínculo commit↔check (concepto clave):** el resultado del CI se graba sobre el **SHA** del
commit, no sobre la rama. Un commit verde sigue verde donde sea que lo lleves; `amend`/`rebase`/
`squash` cambian el SHA → el check se pierde y hay que re-evaluar.

---

## 🏷️ Versionado y releases (SemVer + tags inmutables)

<!-- GUÍA: cada merge `<develop> → main` que entrega valor es una RELEASE. El versionado le pone
     nombre estable a un punto del historial para poder referenciarlo, desplegarlo y auditarlo.
     Aplica a cualquier proyecto que produzca artefactos versionables. -->

**SemVer (`MAJOR.MINOR.PATCH`):** subir `PATCH` (correcciones), `MINOR` (funcionalidad
retrocompatible), `MAJOR` (cambios incompatibles). El CHANGELOG (formato *Keep a Changelog*, la
versión más reciente arriba) se actualiza **antes** de cortar el tag, no después.

**Qué es un tag (concepto clave):** un tag es un **puntero nombre→SHA**, no un texto dentro del
commit. `git tag -a vX.Y.Z -m "..."` crea un tag **anotado** (objeto propio con autor/fecha/mensaje)
— ése es el marcador de release, no un tag ligero. El tag apunta al **SHA** del commit de release en
`main`; igual que el check de CI, se ancla al SHA, no a la rama.

**Regla de oro — un tag publicado es INMUTABLE.** Nunca mover un tag ya liberado con `git tag -f` +
`git push -f` (anti-patrón): rompe la trazabilidad de quien ya lo consumió o lo ligó a un artefacto/CI.
¿Te equivocaste o hay una corrección? Sale como **versión nueva** (`vX.Y.Z+1`), no como re-etiquetado.
Corolario: no cortes el tag hasta que el commit de release en `main` sea el definitivo (CHANGELOG y
badges de versión ya incluidos).

**GitHub/GitLab Release:** objeto que se **adjunta a un tag** con notas legibles (resumen de cambios,
enlace al CHANGELOG). Marca la más reciente como *Latest*. Las versiones históricas conservan su
release; no se reescriben.

**Flujo de release (resumen):**
```bash
# en main, tras integrar develop→main y con CHANGELOG + badge de versión ya commiteados:
git tag -a vX.Y.Z -m "Release X.Y.Z — <resumen>"   # anotado; SIN -f (inmutable)
git push origin vX.Y.Z
<gh/glab> release create vX.Y.Z --title "..." --notes "..."   # notas legibles, marcar Latest
```

---

## 🔐 Gobernanza de seguridad de dependencias (SCA)

<!-- GUÍA: además del RBAC/secretos del propio código (§Seguridad y RBAC), un repo público maduro
     vigila sus DEPENDENCIAS (CVEs) y ofrece un canal para reportar vulnerabilidades. Complementa al
     CI/CD; aplica a cualquier stack (ajusta las herramientas). -->

- **SECURITY.md** (raíz del repo): política de reporte de vulnerabilidades. Canal recomendado:
  *GitHub Private Vulnerability Reporting* (Settings → Code security), que **no expone email**. Nunca
  pedir que se abra un issue público para una vulnerabilidad.
- **Dependabot** (`.github/dependabot.yml`): GitHub vigila las dependencias en **su servidor** (escaneo
  inicial + semanal) y abre PRs; no depende del CI. Agrupar **solo minor/patch**
  (`update-types: ["minor","patch"]`) para que el PR grupal sea mergeable en verde; los **major** llegan
  como PR individual (rompen el build → revisión manual, o `@dependabot ignore this major version` para
  posponer). Cubrir también el ecosistema `github-actions`.
- **Escaneo en CI (SCA):** el analizador **rápido** (p.ej. `npm audit`) va como paso del CI (gate
  bloqueante en `critical`, informativo en `high`); el **pesado** (p.ej. OWASP dependency-check con base
  NVD) va en un **workflow SEPARADO programado**, no en el gate de cada push.
- **Secretos:** nunca en git (ni en el historial); externalizar con `${VAR:default}`. Un secreto que
  toca git se considera comprometido → rotar.

---

## 🛠️ Operación de producción (Day-2 ops)

<!-- GUÍA: hay dos mundos — Day-1 (construir y desplegar; lo cubre CI/CD) y Day-2 (OPERAR el sistema
     vivo: observarlo, recuperarlo, mantenerlo estable). Define el MÍNIMO defendible desde el diseño,
     no al final. Máximas: no operas lo que no observas; no recuperas lo que no respaldaste; un backup
     no restaurado no es un backup. Aplica a cualquier stack; se consolida en el RUNBOOK de operación. -->

**Mínimo indispensable** (por debajo de esto, operar en prod es negligente, no solo "inmaduro"):
- **Backup + restauración PROBADA:** cifrado + off-site (regla 3-2-1) + rotación; con **RPO/RTO**
  definidos. El **drill de restore** es obligatorio — un backup sin probar no cuenta.
- **Monitoreo de uptime EXTERNO + alerta:** saber cuándo está caído (un monitor interno no ve la caída total).
- **Retención de logs:** rotación por tiempo/tamaño para no llenar el disco (self-DoS).
- **Higiene de credenciales:** cambiar toda credencial por defecto en el primer arranque; secretos por env; rotación.
- **Confiabilidad barata:** graceful shutdown (drenar peticiones) + restart automático + límites de cpu/mem.
  ⚠ El orquestador debe esperar más que el timeout de apagado antes del SIGKILL (`stop_grace_period` >
  `timeout-per-shutdown-phase`) o el drenado se corta.
- **Runbook de operación** lleno: deploy, rollback, restore, monitoreo, diagnóstico.

**Diferible (madurez, no mínimo):** métricas (Prometheus/Grafana), trazado distribuido, pruebas de
carga, SLO/error budgets, agregación centralizada de logs.

**Regla:** ningún proceso de activación en el despliegue queda sin documentar (cada uno con su sección
en el runbook / instructivo).

---

## 📄 Documentación obligatoria por módulo

Todo módulo nuevo requiere, **antes de implementar**:
- `docs/modulos/<nombre>/propuesta_modulo_<nombre>.md` — planificación previa al código.
- `docs/qa/casos_de_prueba_modulo_<nombre>.md` — casos definidos ANTES de codificar (copiar del template).
- `docs/modulos/<nombre>/memoria_tecnica_modulo_<nombre>.md` — documento vivo (se actualiza por fase).

**Secciones de la memoria técnica de módulo** (10): 1) Contexto y justificación · 2) Decisiones de diseño ·
3) Componentes/interfaces · 4) Contratos con dependencias (campos EXACTOS) · 5) Algoritmos/lógica no trivial ·
6) Seguridad/RBAC por rol · 7) Ejecución de tests (evidencia verificable) · 8) Bugs y retos ·
9) Estándares aplicados · 10) Cumplimiento y validación.

---

## ⚠️ Protocolo de pruebas — Propuestas A–D (permanente, todos los módulos)

<!-- GUÍA: estas 4 reglas evitan el anti-patrón "declarar terminado lo que no funciona".
     Aplican a CUALQUIER tipo de proyecto. Ajusta los nombres de comando, no el espíritu. -->

**A — Documento de casos de prueba por módulo (pre-código).** Se crea junto con la propuesta, copiando
`docs/qa/casos_de_prueba_modulo_TEMPLATE.md`. Categorías obligatorias por pantalla/unidad:
`SEC, RBAC/AUTHZ, CRUD, VAL, BSRCH, UI, FLOW, RN, ERR, EMPTY, VIS, CYBER` (adapta las no aplicables a `N/A`).
Es el **criterio de aceptación**: un módulo no está "done" si hay casos sin `✅ PASS`.

**B — Verificación por unidad, no por módulo.** Una unidad (componente/endpoint/comando) no está terminada
hasta que TODOS sus casos están en `✅ PASS` con el rol/condición correctos. No acumular deuda de verificación.

**C — Gate de seguridad por cada punto de entrada nuevo.** Toda ruta/endpoint/comando nuevo:
```
[ ] Tiene autorización explícita (guard/decorator/middleware) con los roles permitidos.
[ ] Se probó el acceso con el rol MENOS privilegiado SIN acceso (no basta esconder el enlace en UI).
[ ] La autorización vive en el backend/servidor, no sólo en el cliente.
```

**D — Definición de "done" (no ofrecer continuar hasta cumplir las 4):**
```
[ ] 1. Todos los casos de prueba en ✅ PASS.
[ ] 2. Gatekeeper en verde (build + tests + lint) y cobertura ≥ <70>%.
[ ] 3. Verificación funcional ejecutada y documentada para todos los roles/condiciones.
[ ] 4. Columna "Estado" del documento de casos completa (ningún ⏳ PENDIENTE).
```

---

## ⚠️ Protocolo pre-código — Consulta de contratos

**Antes de escribir cualquier cliente/modelo que consuma una dependencia (API, librería, esquema),
verifica los contratos REALES.** No asumir nombres de campos ni códigos de respuesta.

Para cada endpoint/función externa verifica: ruta/firma exacta · código/forma de respuesta (incluido vacío/204) ·
nombres EXACTOS de campos · forma del request (obligatorio vs opcional) · colección vs objeto vs void.
Documenta lo verificado en la Sección 4 de la memoria técnica del módulo **antes** de codificar.

---

## ⚠️ Protocolo post-código — Cierre de unidad/módulo

No basta con que compile y pasen los tests unitarios. Antes de declarar terminado:
```
[ ] Casos de prueba de ESTA unidad en ✅ PASS (categorías SEC/RBAC/CRUD/VAL/BSRCH/UI/FLOW/RN/ERR/EMPTY).
[ ] Gatekeeper en verde sobre la suite completa.
[ ] Verificación por rol/condición: autorización, mensajes de error útiles, estados vacíos diferenciados.
[ ] Datos sensibles (precios/costos/PII/secretos) ausentes para roles no autorizados (en datos, no sólo en UI).
[ ] Campos de sólo-lectura realmente deshabilitados; validación preventiva antes del submit.
[ ] Revisé las lecciones documentadas buscando el mismo patrón de bug.
```

---

## ⚠️ Protocolo de verificación en 4 fases (rondas de QA)

<!-- GUÍA: ver qa/protocolo_verificacion_4_fases_TEMPLATE.md para el detalle. Resumen: -->
> **Regla inamovible:** una ronda de pruebas sólo es válida si se ejecuta íntegra sobre una versión
> **congelada** del código. Si se corrige un bug a mitad, la ronda se invalida y se reinicia.

1) **Inventario** (código congelado, documentar bugs, NO corregir) → 2) **Corrección + gatekeeper**
(documentar *blast radius* de cada fix) → 3) **Re-ejecución** completa desde cero → 4) **Certificación**
(gatekeeper + cobertura + commit `chore(qa): ...`).

**Blast radius:** cambio local (componente/servicio del módulo) → re-probar sólo ese módulo;
cambio global (auth, interceptor, config de seguridad, servicio compartido, manejador global de errores) →
re-probar **todos** los módulos.

---

## 🔐 Seguridad y RBAC (si aplica)

<!-- GUÍA: si el proyecto tiene roles/permisos, documenta la matriz aquí o enlaza la memoria global. -->
- Roles: <ROL_A, ROL_B, ...>. Matriz de acceso por módulo en `docs/arquitectura/memoria_tecnica_global.md`.
- **Campos sensibles por rol:** documenta qué campos se redactan y para qué roles; aplica la redacción
  **en el servidor** de forma centralizada (no sólo ocultar en el cliente).
- Errores de auth coherentes: `401` (no autenticado/token inválido) vs `403` (autenticado sin permiso).
- Endpoints de autenticación nuevos: rate limiting/lockout desde el primer commit.

---

## 🧱 Estándares de código

<!-- GUÍA: enlaza el documento de estándares; aquí sólo las reglas que Claude debe respetar siempre. -->
- Convenciones y buenas prácticas: `docs/arquitectura/estandares_desarrollo.md`.
- Nunca hardcodear URLs/secretos: usar variables de entorno/configuración.
- Gestión de recursos/suscripciones con limpieza (sin fugas).
- Componentes/funciones pequeños y testeables; separar lógica de presentación.

---

## 🧪 Taxonomía de tests

| Tipo | Herramienta | Qué verifica |
|---|---|---|
| Unit | `<unit tool>` | Lógica de unidades aisladas |
| Integración | `<integration tool>` | Interacción entre componentes / con dependencias reales |
| E2E | `<e2e tool>` | Flujo completo de extremo a extremo |

Cobertura mínima: **<70>% statements** por módulo.

---

## 📦 Estado actual

**Fase:** <inicialización | en desarrollo (módulo X) | QA | producción | cerrado (estable · mantenimiento)>.
<2-3 líneas del estado real: qué módulos existen, métricas de tests, qué sigue.>

> **Mantenimiento:** ante cualquier cambio, seguir el Protocolo de 4 fases y actualizar la memoria
> técnica del módulo + la global. Mantener este `CLAUDE.md` al día.
>
> **Cierre de proyecto (formalización):** al dar el proyecto por terminado, crear un **acta de cierre**
> (ver `planificacion/acta_cierre_proyecto_TEMPLATE.md`: entregables, calidad, versiones, brechas,
> retrospectiva y trabajo futuro), declarar el estado **estable / mantenimiento** en el README y la
> memoria global, y **NO archivar** el repositorio (archivar = solo lectura, señala abandono; el modo
> mantenimiento lo mantiene vivo y actualizable).
