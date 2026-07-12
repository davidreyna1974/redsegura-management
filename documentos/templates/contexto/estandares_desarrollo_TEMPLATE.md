# Estándares de desarrollo — <NOMBRE DEL PROYECTO>

<!-- GUÍA: documento de referencia de convenciones. Lo que aquí se acuerda se asume en TODO el código.
     Mantenlo corto y accionable: reglas, no ensayos. Borra las secciones que no apliquen. -->

## 1. Principios generales

- **Legibilidad sobre astucia.** El código nuevo se parece al que lo rodea (nombres, idioma, densidad de comentarios).
- **Una fuente de verdad.** No duplicar lógica/constantes; centralizar configuración.
- **Fallar pronto y claro.** Validar entradas en el borde; errores con mensaje útil.
- **Seguro por defecto.** Lo no autorizado se niega; los secretos nunca se hardcodean ni se loguean.
- **Pequeño y testeable.** Funciones/unidades con una responsabilidad; dependencias inyectables.

## 2. Nomenclatura y estructura

| Elemento | Convención | Ejemplo |
|---|---|---|
| Archivos/carpetas | <kebab-case / snake_case> | `<...>` |
| Clases/Tipos | PascalCase | `<...>` |
| Funciones/variables | <camelCase / snake_case> | `<...>` |
| Constantes | UPPER_SNAKE_CASE | `<...>` |
| Ramas git | `feature/` `fix/` `chore/` | `feature/login` |

Estructura de carpetas de referencia: <describe o enlaza el árbol del proyecto>.

## 3. Estilo y herramientas

- Formateador: `<prettier/ruff format/gofmt>` — corre en pre-commit/CI; el formato no se discute en review.
- Linter: `<eslint/ruff/checkstyle>` con reglas en `<archivo de config>`; 0 warnings en main.
- Tipos: `<strict mode / mypy --strict>` activado; sin `any`/`# type: ignore` sin justificar.

## 4. Manejo de errores

- Distinguir errores de **validación** (4xx) de errores **internos** (5xx); no filtrar detalles internos al cliente.
- Manejo centralizado (handler/middleware global) para respuestas de error consistentes.
- Nunca tragar excepciones en silencio; loguear con contexto suficiente (sin datos sensibles).

## 5. Configuración y secretos

- Toda config sensible vía **variables de entorno**; nunca en el código ni en el repo.
- Provee un `.env.example` con las claves (sin valores) y documenta cada una.
- Secretos reales fuera del control de versiones; rotación si se exponen.

## 6. Logging y observabilidad [opcional]

- Niveles consistentes (`debug/info/warn/error`); nada de PII/secretos en logs.
- Correlación de peticiones (request id) si aplica; métricas/health checks para servicios.
- **Rotación + retención de logs** en producción (por tiempo y tamaño) — no llenar el disco (self-DoS).
- Servicios de larga vida: **apagado ordenado (graceful shutdown)** para drenar peticiones en vuelo
  (el orquestador debe esperar más que el timeout de apagado antes del SIGKILL).

## 7. Estándar de búsqueda de texto [opcional, si hay búsquedas por texto libre]

<!-- GUÍA: lección real: LOWER() quita mayúsculas pero NO acentos. "galon" no encuentra "Galón". -->
- Búsquedas insensibles a **mayúsculas y acentos** (normalizar/`unaccent`), no sólo a mayúsculas.
- En cliente: `debounce` (~300-350 ms); búsqueda vacía = omitir el filtro (no enviar cadena vacía).

## 8. Accesibilidad / UX [opcional, si hay UI]

- Contraste mínimo WCAG AA; foco de teclado navegable; `aria-label` en íconos interactivos.
- Estados vacíos diferenciados ("sin datos" vs "sin resultados"); feedback de éxito/error visible.
- Mensajes de error inline junto al campo; confirmación para acciones destructivas.

## 9. Pruebas

- Cobertura mínima **<70>% statements** por módulo; lo crítico (seguridad/negocio) con prioridad.
- Tests deterministas (sin dependencias de orden/tiempo real); datos de prueba prefijados y limpiados.
- Cada bug corregido nace con un test de regresión que lo reproduce.

## 10. Documentación

- Cada módulo: propuesta + casos de prueba + memoria técnica (ver `CLAUDE.md`).
- Decisiones transversales → memoria técnica global; lecciones → registro de lecciones.
- README, CHANGELOG y diagrama al día como parte de "done".

---

## 📒 Registro de lecciones (vivo)

<!-- GUÍA: cada bug/decisión no obvia que deba prevenirse a futuro se vuelve una "lección" numerada.
     Es uno de los activos más valiosos del proyecto. Formato sugerido: -->

| ID | Lección (regla a futuro) | Origen (bug/decisión) | Alcance |
|---|---|---|---|
| L01 | <regla accionable, en imperativo> | <de dónde salió> | local/global |
| L02 | <...> | <...> | <...> |
