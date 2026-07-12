# Plan de salida a producción — <NOMBRE DEL PROYECTO> <vX.Y.Z>

<!-- GUÍA: checklist de "lo que debe estar listo antes de exponer el sistema". Cúbrelo de arriba
     a abajo; nada se da por hecho. Marca responsable y estado de cada ítem. -->

**Fecha objetivo:** <YYYY-MM-DD> · **Responsable:** <...> · **Entorno destino:** <prod / staging>

## 1. Pre-requisitos de calidad (puerta de entrada)
```
[ ] Todos los módulos certificados (Propuesta D) — reporte de QA al día.
[ ] Gatekeeper en verde sobre el commit a desplegar: build + tests + lint + cobertura ≥ 70%.
[ ] Sin secretos en el repo; .env.example completo y documentado.
[ ] CHANGELOG y versión (SemVer + tag) actualizados.
```

## 2. Infraestructura
```
[ ] Servidor/host aprovisionado (CPU/RAM/disco) y endurecido (usuarios, SSH, actualizaciones).
[ ] Dominio y DNS configurados.
[ ] TLS/HTTPS (certificado válido, redirección 80→443).
[ ] Firewall: sólo puertos necesarios abiertos.
[ ] Base de datos/almacenamiento provisionado, con usuario de app de privilegios mínimos.
```

## 3. Configuración de la aplicación
```
[ ] Variables de entorno de producción cargadas (sin valores de dev).
[ ] Orígenes CORS / hosts permitidos restringidos a los reales.
[ ] Nivel de logs adecuado (sin debug/PII); rotación de logs.
[ ] Migraciones de BD aplicadas y verificadas.
[ ] Datos semilla / usuario administrador inicial creados de forma segura.
```

## 4. Seguridad
```
[ ] Autenticación y autorización por rol verificadas en el entorno destino.
[ ] Rate limiting/lockout en endpoints de autenticación.
[ ] Cabeceras de seguridad (HSTS, X-Content-Type-Options, etc.) si aplica.
[ ] Secretos en gestor seguro; rotados si estuvieron expuestos.
[ ] Revisión de dependencias (vulnerabilidades conocidas).
```

## 5. Observabilidad y resiliencia (Day-2 — mínimo)
```
[ ] Health checks / readiness endpoints por servicio (con herramienta presente en la imagen).
[ ] Monitoreo de uptime EXTERNO + alerta (caída total, errores, disco).
[ ] Backup CIFRADO + copia off-site (regla 3-2-1) + rotación; RPO/RTO definidos.
[ ] Restauración PROBADA (drill en BD limpia; medir el RTO real). Un backup no probado no cuenta.
[ ] Confiabilidad: graceful shutdown (stop_grace_period > timeout de apagado) + restart automático + límites cpu/mem.
[ ] Retención de logs (rotación por tiempo/tamaño — no llenar el disco).
[ ] Plan de rollback definido y probado.
```

## 6. Despliegue
```
[ ] Artefacto/imagen construido desde el commit/tag exacto a liberar.
[ ] Procedimiento de despliegue ejecutado (ver runbook_despliegue.md).
[ ] Smoke test post-despliegue: login, ruta principal de cada módulo, health check.
```

## 7. Post-salida
```
[ ] Verificación funcional por rol en producción.
[ ] Monitoreo observado durante <ventana, p.ej. 24-48 h>.
[ ] Estado de sesión / memoria global actualizados con la release.
[ ] Comunicación a interesados.
```

## 8. Criterio de éxito / abort
- **Éxito:** <smoke test verde + sin errores 5xx en la ventana de observación>.
- **Abort/rollback si:** <criterio claro: errores críticos, datos corruptos, caída>.
