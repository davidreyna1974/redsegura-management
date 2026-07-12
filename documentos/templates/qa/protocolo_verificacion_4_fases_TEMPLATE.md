# Protocolo de verificación en 4 fases — <NOMBRE DEL PROYECTO>

<!-- GUÍA: protocolo permanente para CUALQUIER ronda de QA (módulo nuevo, re-ejecución, post-fix).
     Nace de un anti-patrón real: mezclar "probar" y "corregir" invalida los casos ya ejecutados.
     Adapta los comandos del gatekeeper; NO cambies la regla fundamental. -->

## Regla fundamental (inamovible)

> Una ronda de pruebas sólo es válida si se ejecuta **íntegra sobre una versión congelada del código**,
> sin modificaciones entre el primer y el último caso. Si durante la ronda se encuentra un bug y se
> **corrige**, la ronda se **invalida** y se reinicia desde cero.

## Las 4 fases

### FASE 1 — Inventario (código congelado)
- Ejecutar TODOS los casos de TODOS los módulos sin tocar código.
- Documentar cada bug con estado `⚠️ ABIERTO`. **No corregir nada.**
- Objetivo: conocer el estado real del sistema.

### FASE 2 — Corrección + gatekeeper
- Corregir los bugs del inventario en ciclo normal de desarrollo.
- **Gatekeeper obligatorio por cada fix**, en orden:
  1. `<cmd build>` → 0 errores
  2. `<cmd test>` → 0 fallos
  3. `<cmd lint/type>` → 0 errores
- Documentar el **blast radius** de cada fix (ver tabla abajo).

### FASE 3 — Re-ejecución completa desde cero
- Re-ejecutar los casos de los módulos con blast radius afectado, en una sola sesión continua.
- Si aparece un bug nuevo → documentar, **NO corregir**, terminar la fase → volver a Fase 2.
- **Lectura estricta vs por blast radius:** declarar cuál se aplicó.
  - *Por blast radius* = sólo zonas tocadas por los fixes (iteración rápida).
  - *Estricta* = TODOS los casos del módulo, una sesión continua, bundle congelado.
  - **Sólo una Fase 3 estricta habilita declarar el módulo CERTIFICADO.**
- Antes del primer caso: verificar congelamiento (git limpio, servicios arriba, build vigente — no stale).

### FASE 4 — Certificación
- `<cmd build>` → 0 errores · `<cmd test --coverage>` → cobertura ≥ <70>%, 0 fallos.
- Actualizar el estado de sesión con resultado **CERTIFICADO**.
- Actualizar el resumen de cobertura en cada documento de casos afectado.
- Commit: `chore(qa): verificación completa 4 fases — <fecha>`.

## Blast radius (documentar por cada fix)

| Tipo de cambio | Alcance | Re-probar en Fase 3 |
|---|---|---|
| Estilo/CSS de un componente | Local | Sólo ese módulo |
| Lógica de un componente/servicio del módulo | Local | Sólo ese módulo |
| Interceptor/middleware HTTP | Global | Todos los módulos |
| Guard/Auth/sesión | Global | Todos los módulos |
| Config de seguridad del servidor | Global | Todos los módulos |
| Servicio compartido / manejador global de errores | Global | Todos los módulos |

## Catálogo de técnicas de verificación (por categoría)

- **VAL:** tecleo real y lectura del mensaje de error inline (no inspección de estado interno).
- **VIS:** medir color/estilo computado real (RGB exacto), no asumir del CSS fuente.
- **RBAC:** comprobar **ausencia en el DOM/respuesta**, no sólo ocultamiento visual.
- **SEC:** navegación/llamada por ruta directa con el rol menos privilegiado.
- **CYBER (server-side):** `<curl/cliente>` con credencial por rol → verificar enforcement y redacción de campos.
- **Resiliencia:** apagado controlado de la dependencia/backend → verificar mensaje útil, no crash.
- **Búsqueda:** server-side vs client-side; parcial, case- y accent-insensitive, sin resultados.

## Estado de la sesión

El archivo `docs/qa/_bitacora/estado_sesion_activa.md` (ver template de sesiones) persiste el avance.
**Leerlo al iniciar cualquier sesión de pruebas** para retomar sin pérdida de contexto.
