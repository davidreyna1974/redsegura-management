# Política de seguridad — redSegura

## Reporte de vulnerabilidades

Si descubres una vulnerabilidad en redSegura, **no la publiques en un issue público**. Usa un
**canal privado**:

- Preferido: **GitHub Private Vulnerability Reporting** (pestaña *Security → Report a
  vulnerability* del repositorio), una vez publicados los repos remotos.
- Alternativo: contacto directo con el mantenedor del proyecto.

Incluye, en la medida de lo posible: descripción, pasos de reproducción, impacto estimado,
versión/commit afectado y cualquier evidencia (sin exponer datos sensibles de terceros).

Objetivo de respuesta: **acuse en 72 horas** y un plan de mitigación acordado según la severidad.

## Alcance

Aplica a los tres repositorios del sistema (`management`, `backend`, `frontend`) y a su
configuración de despliegue.

## Prácticas de seguridad del proyecto

- **Sin secretos en git:** credenciales SSH de dispositivos, claves de API y credenciales de BD
  se externalizan por variables de entorno o gestor de secretos; nunca en el código ni en el
  historial (RNF-06).
- **RBAC de extremo a extremo:** autorización validada en el API Gateway **y** de forma
  independiente en cada microservicio (RNF-04).
- **SCA de dependencias:** Dependabot (`maven`, `pip`, `github-actions`) + escaneo en CI,
  bloqueante en severidad `critical` (RNF-08); escaneo profundo (OWASP/NVD) en workflow
  programado.
- **Alcance autorizado de escaneo (RNF-07):** control técnico obligatorio; durante el desarrollo
  y las pruebas de este proyecto apunta **exclusivamente** a la red simulada (GNS3/Packet
  Tracer), nunca a redes de terceros sin autorización explícita.
- **Errores sin fuga de internos:** las respuestas de API no exponen stack traces ni detalles de
  implementación (RNF-09); los logs no contienen PII ni secretos (RNF-17).

## Divulgación responsable

Agradecemos la divulgación responsable y coordinada. No se emprenderán acciones contra quienes
reporten de buena fe respetando esta política.
