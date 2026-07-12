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

## 8. Checklist de apertura (antes de codificar)

```
[ ] Propuesta creada (este documento).
[ ] docs/qa/casos_de_prueba_modulo_<nombre>.md creado desde el TEMPLATE (categorías completas).
[ ] Memoria técnica de módulo iniciada.
[ ] Contratos de dependencias verificados (Sección 4).
[ ] Gate de seguridad previsto para todas las rutas/endpoints nuevos.
```
