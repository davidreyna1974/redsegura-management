# Especificaciones de diseño — Prácticas base para el nuevo proyecto

> **Propósito.** Lista curada de las funcionalidades, buenas prácticas, estándares, protocolos,
> enfoques y metodologías que se implementaron en el proyecto **Almacenes** y que lograron un sistema
> robusto, estable, seguro y homogéneo. El objetivo es **adoptarlas desde el día 1** en el nuevo
> proyecto, en lugar de descubrirlas tarde.
>
> **Cómo usar este documento.** Es un **índice de prácticas por nombre identificativo**, agrupado por
> área. El **detalle de cada una** vive en los archivos **template y de contexto** reutilizables
> (CLAUDE.md, estándares de desarrollo, memoria técnica global, protocolo de QA, propuestas y planes,
> runbook de despliegue), que se usan como base del nuevo proyecto.
>
> **Alcance.** Extracción fiel de Almacenes (monolito). Nombres **agnósticos de tecnología**. Las
> prácticas que en Almacenes se incorporaron tarde (CI/CD, versionado, CHANGELOG, seguridad de
> dependencias, etc.) aparecen **integradas en su área**, para tratarlas como cualquier otra desde el
> inicio. Las prácticas específicas de una arquitectura de microservicios **no** están aquí (se
> definen por separado).

---

## 1. Planeación y diseño previo (antes de escribir código)

- Propuesta de proyecto antes de iniciar (problema, objetivos, alcance, usuarios/roles, criterios de éxito).
- Plan de trabajo con secuencia de módulos/etapas e hitos.
- Propuesta por módulo antes de codificar (alcance, unidades/endpoints, reglas de negocio, RBAC, riesgos).
- Especificación técnica para funcionalidades no triviales (diseño, alternativas, impacto, riesgos).
- Verificación de contratos de integración/API **antes** de codificar el consumidor.
- Documentación mandatoria por módulo desde su apertura (propuesta + casos de prueba + memoria técnica).
- Definición explícita de "done" por módulo, con condiciones verificables.
- Checklist de apertura de módulo antes de la primera línea de código.

## 2. Arquitectura

- Arquitectura en capas (presentación → servicio → persistencia) con módulos de negocio independientes.
- Núcleo transversal separado (seguridad, manejo de errores, configuración, componentes compartidos).
- Separación de responsabilidades UI: componentes contenedores (lógica) vs presentacionales (reutilizables).
- Contratos de integración documentados y verificados como fuente de verdad.
- Manejo global y centralizado de excepciones (global exception handler).
- Mapeo explícito DTO ↔ entidad (nunca exponer entidades de dominio).
- Máquinas de estado explícitas para flujos de negocio (transiciones válidas y bloqueos).
- Control de concurrencia (bloqueo optimista) en operaciones críticas.
- Inicializadores/seeding idempotentes ejecutados por la aplicación, no manuales.
- Memoria técnica global como fuente de contexto primaria (visión, decisiones, contratos, RBAC, lecciones).

## 3. Seguridad

- **RBAC de extremo a extremo — mandatorio**: en rutas/guards, en UI (visibilidad) y en datos.
- Redacción de campos sensibles en el servidor por rol (no solo ocultarlos en el cliente).
- Matriz de campos sensibles × roles documentada antes de implementar el endpoint.
- Autenticación stateless por token con expiración y re-login/refresh.
- Distinción coherente 401 (no autenticado / token inválido) vs 403 (autenticado sin permiso).
- Gate de seguridad por ruta (guard + roles en cada ruta nueva; ocultar el enlace no es protección).
- Rate limiting / lockout en endpoints de autenticación desde el primer commit.
- Secretos externalizados por variable de entorno; nunca en git ni en el historial; rotación si se exponen.
- Errores que no filtran tipos/detalles internos (entrada malformada → 400 sin exponer internos).
- CORS explícito y restrictivo.
- Cabeceras de seguridad HTTP (CSP, HSTS, X-Frame-Options, nosniff) verificadas pre-producción.
- Política de reporte de vulnerabilidades (SECURITY.md) con canal privado.
- Escaneo automatizado de dependencias (SCA): alertas/actualizaciones + escaneo en CI.

## 4. Gestión de código y control de versiones

- Flujo Git estricto: nunca commit directo a las ramas protegidas; ramas feature/fix/chore + merge no-fast-forward.
- Doble capa de protección de ramas: hook local pre-commit + branch protection en el servidor.
- Convención de mensajes de commit (Conventional Commits) con el "por qué" en el cuerpo.
- Versionado semántico (SemVer) con tags anotados **inmutables** (un tag publicado no se mueve).
- Releases publicados con notas legibles; la más reciente marcada como vigente.
- CHANGELOG (formato Keep a Changelog) con sección "No publicado" mantenida en cada PR.
- Enlaces de comparación por versión.

## 5. CI/CD y automatización

- CI que automatiza el gatekeeper (build + tests + lint) en entorno limpio en cada push/PR.
- Gatekeeper obligatorio por cada fix (build + tests verdes) antes de continuar.
- Build de producción con verificación estricta (tipos/plantillas) en CI — atrapa lo que los tests no ven.
- Pruebas pesadas (E2E, escaneo profundo de dependencias) en workflows **separados**, no en el gate de cada push.
- Entrega continua (CD) que publica el artefacto versionado (imagen/paquete) etiquetado por commit.
- Compuerta real vía branch protection (requiere PR + checks de estado); el resultado se ancla al commit (SHA).
- Dependencias de test (BD, cache) provistas como servicios efímeros en el propio pipeline.

## 6. Calidad y pruebas (QA)

- Casos de prueba por módulo definidos **antes** de codificar (criterio de aceptación).
- Categorías de prueba obligatorias por pantalla/endpoint (seguridad, RBAC, CRUD, validación, búsqueda,
  UI, flujo/estado, reglas de negocio, errores, estados vacíos, visual, ciberseguridad).
- Verificación por componente con el rol correcto (no acumular deuda de verificación para el final).
- Protocolo de verificación en fases sobre **código congelado** (inventario → corrección → re-ejecución → certificación).
- Análisis de "blast radius" por cada fix (local vs global) para acotar la re-prueba.
- Taxonomía de tests clara (unitarios de servicio y de componente, integración, seguridad, persistencia, E2E).
- Cobertura mínima por módulo como umbral.
- Pruebas de seguridad server-side por rol (enforcement de RBAC + redacción de campos).
- Trazabilidad regla-de-negocio → componente/endpoint responsable (mostrar el dato correcto, validar, mensaje útil).
- Usuarios de prueba permanentes por rol + convención de datos de prueba (prefijo y limpieza).

## 7. Estándares de código transversales

- Documentación de API (Swagger/OpenAPI) desde el primer módulo.
- Paginación estándar desde el primer endpoint de colección; reset del paginador al cambiar filtros (punto único).
- Excepciones tipadas para errores de negocio (nunca genéricas) → códigos y mensajes coherentes.
- Inyección de dependencias por constructor.
- Transacciones explícitas en la capa de servicio.
- Formularios reactivos con validación centralizada.
- Gestión de suscripciones con limpieza (evitar fugas de memoria).
- Servicios de datos que retornan flujos observables (sin suscribirse dentro del servicio).
- Combinación de llamadas concurrentes tolerante a fallo por fuente según RBAC.
- Respuestas HTTP coherentes en controladores (201/204/400/422 según corresponda).
- Nomenclatura y estructura de proyecto homogéneas.

## 8. Datos y persistencia

- Columnas de auditoría estándar (creado/actualizado por/en) con un patrón único.
- Campos de solo lectura protegidos (p. ej. stock modificado solo vía movimientos, no editable directo).
- Búsqueda de texto insensible a acentos y mayúsculas (normalización) con anti-rebote y omisión de filtro vacío.
- Reglas de negocio e integridad referencial en la capa de servicio.
- Datos de referencia sembrados por la aplicación (idempotente), no por scripts manuales.
- Magnitudes de negocio bien diferenciadas (p. ej. total / reservado / disponible) — usar el dato que la regla exige.
- Migraciones/esquema versionados y validados.

## 9. UX/UI y diseño de interfaz

- Identidad visual y paleta definidas en un tema centralizado.
- Layout maestro-detalle reutilizable.
- Feedback consistente al usuario (notificaciones semánticas, indicadores de progreso, carga esqueleto).
- Estados vacíos diferenciados (sin datos iniciales vs sin resultados de búsqueda).
- Tooltips en íconos/acciones y en texto truncado.
- Diálogos con cierre controlado por defecto cuando contienen formularios.
- Patrón único de acciones en tablas (fila accionable; sin columnas de íconos redundantes).
- Badges/indicadores de estado con colores semánticos consistentes en toda la app.
- Botón de guardar deshabilitado si el formulario es inválido, no modificado o está cargando.
- Accesibilidad mínima (contraste WCAG AA, etiquetas ARIA, navegación por teclado, foco atrapado en diálogos).
- Formato consistente de fecha y moneda (adaptadores propios).
- Estilos repetidos centralizados (DRY: mixins/placeholders compartidos).
- Validación de entradas dependientes antes de disparar acciones (p. ej. rangos de fecha).

## 10. Documentación

- Memoria técnica viva por módulo (secciones fijas) actualizada al cerrar cada fase.
- Memoria técnica global del sistema como única fuente de contexto.
- Registro de lecciones vivo (numeradas, con causa y "cómo aplicar").
- Índice navegable de documentación por repositorio.
- READMEs con badges (versión, CI/CD, cobertura, QA, seguridad, licencia).
- Diagramas de arquitectura (renderizables, p. ej. Mermaid).
- Archivo de contexto para el asistente de IA con convenciones y protocolos del proyecto.
- Biblioteca de **templates reutilizables** de todos los artefactos (contexto, arquitectura, módulos, QA, planeación, despliegue).
- Organización documental clara: documentación general del sistema separada de la propia de cada repositorio.

## 11. Despliegue y operación

- Despliegue agnóstico del dominio (parametrizado; sin valores hardcodeados).
- Runbook de despliegue y operación (desplegar por versión, smoke test, rollback, diagnóstico, escalamiento).
- Scripts de despliegue numerados y reproducibles.
- Contenerización del artefacto.
- Checklist pre-producción (SSL, firewall, cabeceras de seguridad, verificación).
- Smoke test post-despliegue + endpoint de salud (health) público.
- Respaldo/recuperación (backup/DR), monitoreo/alertas y retención de logs.

## 12. Gestión y metodología de proceso

- Trabajar por fases delimitadas, una a la vez (no mezclar "probar" y "corregir" en la misma ronda).
- Institucionalizar cada bug como protocolo sistémico (convertir la lección en regla permanente).
- "Done" redefinido con condiciones verificables antes de ofrecer avanzar.
- No declarar nada completo sin evidencia verificable (tests + navegador + seguridad).
- Estado de sesión persistente (bitácora) para retomar sin pérdida de contexto.
- Actualizar la memoria técnica global y el archivo de contexto al cerrar cada módulo/entrega.
- Explicar el concepto/decisión antes de implementar (reforzar el aprendizaje del equipo).

---

## Fuera de alcance (definir por separado)

Prácticas y estándares **propios de una arquitectura de microservicios** que Almacenes (monolito) no
necesitaba: API gateway, descubrimiento de servicios, configuración distribuida, comunicación
inter-servicio y contratos (contract testing), patrones de resiliencia (timeouts, reintentos, circuit
breaker), base de datos por servicio y consistencia eventual, observabilidad distribuida (tracing,
correlación), y seguridad entre servicios (mTLS / propagación de identidad). Se recomienda elaborar un
documento complementario para estas.
