# Propuesta de módulo — <NOMBRE DEL MÓDULO>

<!-- GUÍA: se crea ANTES de escribir código, junto con el documento de casos de prueba.
     Es la planificación previa: qué se va a construir, cómo y con qué contratos. Mantenla breve. -->

**Estado:** <propuesta | aprobada | en desarrollo> · **Fecha:** <YYYY-MM-DD>

## 1. Objetivo del módulo

<Qué funcionalidad aporta y a qué usuarios/roles sirve. 2-4 líneas.>

## 2. Alcance

- **Incluye:** <...>
- **No incluye:** <...>
- **Dependencias:** <otros módulos / servicios / librerías>.

## 3. Pantallas / unidades / endpoints

<!-- GUÍA: lista las unidades concretas a construir (pantallas, endpoints, comandos, jobs). -->

| Unidad | Tipo | Descripción | Rol(es) con acceso |
|---|---|---|---|
| <Lista de X> | <pantalla/endpoint> | <...> | <ROL_A, ROL_B> |
| <Detalle/Form X> | <...> | <...> | <...> |

## 4. Contratos con dependencias (verificados)

<!-- GUÍA: NO asumir nombres de campos ni códigos. Verificar contra Swagger/código/esquema real. -->

| Endpoint/función | Método/firma | Request | Response (campos EXACTOS) | Código |
|---|---|---|---|---|
| <...> | <...> | <...> | <...> | <...> |

Interfaces/DTOs relevantes:
```
<define aquí los tipos con nombres EXACTOS del backend/dependencia>
```

## 5. Reglas de negocio

- RN1: <regla> → rechazo con código/mensaje <...>.
- RN2: <...>.

## 6. Seguridad / RBAC del módulo

- Rutas/endpoints con autorización: <lista> → roles permitidos.
- Campos sensibles a redactar por rol: <...>.

## 7. Riesgos y decisiones de diseño

- <decisión> porque <motivo>.
- <riesgo> → <mitigación>.

## 8. Revisión contra estándares de industria

<!-- GUÍA: paso corto y REPETIBLE. NO re-derivar los estándares globales (ya son ADR + gobernados
     por CI): solo confirmar cumplimiento. El foco real es el análisis de estándares ESPECÍFICOS
     del DOMINIO de este servicio. Un hallazgo transversal nuevo se PROMUEVE a ADR global (+ regla
     de gobernanza) para que beneficie a todos y no se re-descubra. -->

**a) Cumplimiento de estándares globales** (heredados; la mayoría los verifica el gatekeeper /
gobernanza Spectral — aquí solo se confirma):
```
[ ] Errores RFC 7807 · [ ] probes liveness/readiness · [ ] Idempotency-Key en creaciones
[ ] ETag/If-Match en mutaciones · [ ] columnas de auditoría · [ ] redacción de campos por rol
[ ] paginación y filtrado estándar · [ ] RBAC por operación · [ ] logging estructurado
```

**b) Estándares específicos del DOMINIO de este servicio** (el análisis que sí cambia por servicio):

| Estándar/patrón del dominio | ¿Aplica? | Cómo se incorpora / decisión |
|---|---|---|
| <p. ej. CVSS/EPSS/KEV · CIS Benchmark/STIG · SNMP/gNMI · ITIL · SCAP…> | sí/no | <...> |

**c) Hallazgos transversales a promover:** <ninguno | proponer ADR-XX + regla de gobernanza>.

## 8b. Revisión de aplicabilidad de RNF (production readiness)

<!-- GUÍA: paso OBLIGATORIO pre-código. Recorre TODOS los RNF del proyecto (§8 del spec maestro) y
     decide, para ESTE servicio, si cada uno: Aplica (se cubre en DEV), es N/A (justificado), o se
     Difiere (con disparador: INT-CONS/INT-SYNC/DEPLOY/PRE-REL — ver preparacion_produccion.md). Así
     el servicio "nace revisado" y ningún RNF se salta por olvido. El resultado se materializa en la
     matriz `<servicio>/documentos/matriz_rnf.md` (desde qa/matriz_rnf_TEMPLATE.md). -->

Recorrido de los RNF (`proyecto_microservicios_redsegura.md §8`) para este servicio:

| Bloque de RNF | Aplica en DEV | N/A (justificado) | Diferido (disparador) |
|---|---|---|---|
| Seguridad (auth/RBAC/token/secretos/errores) | <RNF-…> | <RNF-… + motivo> | <RNF-… → etapa> |
| Cadena de suministro (SCA + imagen) | <RNF-08> | | |
| Resiliencia/eventos (Resilience4j, entrega de eventos, health) | <RNF-…> | | <RNF-10 → INT-SYNC; RNF-21 → INT-CONS> |
| Observabilidad · Calidad/CI/docs · API (Swagger runtime) | <RNF-…> | | |
| Escalabilidad/despliegue · Rendimiento | | | <RNF-13/22/23 → DEPLOY; RNF-01 → PRE-REL> |

- [ ] **Matriz de RNF creada** (`<servicio>/documentos/matriz_rnf.md`) con estado por RNF.
- [ ] Los diferidos quedan registrados en `arquitectura/preparacion_produccion.md` con su disparador.

## 9. Checklist de apertura (antes de codificar)

```
[ ] Propuesta creada (este documento).
[ ] documentos/casos_de_prueba.md creado desde el TEMPLATE (categorías completas).
[ ] Memoria técnica de módulo iniciada.
[ ] Contratos de dependencias verificados (Sección 4).
[ ] Revisión contra estándares de industria (Sección 8) completada.
[ ] Revisión de aplicabilidad de RNF (Sección 8b) + matriz_rnf.md creada.
[ ] Gate de seguridad previsto para todas las rutas/endpoints nuevos.
```
