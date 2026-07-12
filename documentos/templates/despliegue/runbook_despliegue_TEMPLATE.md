# Runbook de despliegue y operación — <NOMBRE DEL PROYECTO>

<!-- GUÍA: el "cómo" operativo, paso a paso, para desplegar y operar. Pensado para ejecutarse bajo
     presión: comandos exactos, copiables, idempotentes donde se pueda. Sin secretos en claro. -->

**Entorno:** <prod> · **Última actualización:** <YYYY-MM-DD>

## 1. Arquitectura de despliegue
<Breve: host(s), contenedores/servicios, BD, proxy inverso. Diagrama si ayuda.>

## 2. Requisitos previos
- Acceso: <SSH/usuario, gestor de secretos>.
- Herramientas en el host: <docker/runtime/CLI> versiones <...>.

## 3. Desplegar una nueva versión
```bash
# 1. Construir el artefacto desde el commit/tag exacto
<cmd build / docker build -t app:vX.Y.Z .>

# 2. Publicar/copiar el artefacto
<cmd push / scp>

# 3. Aplicar migraciones (si las hay)
<cmd migrate>

# 4. Levantar la nueva versión
<cmd deploy / docker compose up -d>

# 5. Verificar
<cmd health / curl https://.../health>
```

## 4. Smoke test post-despliegue
```
[ ] Health check responde 200.
[ ] Login con usuario de prueba funciona.
[ ] Ruta/endpoint principal de cada módulo responde.
[ ] Sin errores 5xx en los logs durante los primeros minutos.
```

## 5. Rollback
```bash
# Volver a la versión anterior estable
<cmd deploy app:vX.Y.(Z-1)>
# Revertir migraciones si aplica (con cuidado / desde backup)
<cmd>
```

## 6. Operación rutinaria (Day-2)
<!-- El MÍNIMO defendible de operación: sin esto, correr en producción es negligente
     (no solo "inmaduro"). Ver §Operación de producción de CLAUDE.md. -->

### 6.1 Confiabilidad y arranque
- Reinicio automático de servicios (`restart: unless-stopped` / política del orquestador).
- Límites de CPU/memoria por servicio (evitan que uno agote la RAM del host — efecto cascada/OOM).
- **Apagado ordenado (graceful):** drenar las peticiones en vuelo al detener.
  ⚠ El orquestador debe ESPERAR más que el timeout de apagado de la app antes del SIGKILL
  (p. ej. `stop_grace_period` > `timeout-per-shutdown-phase`), o el drenado se corta.
- Healthcheck por servicio (usar una herramienta PRESENTE en la imagen; si no, agregarla).
- Reinicio manual: `<cmd restart <servicio>>`.

### 6.2 Logs y retención
- Ubicación: `<ruta>`. Rotación + **retención** (por tiempo y tamaño) para no llenar el disco.
- Ver en vivo: `<cmd logs -f>`.

### 6.3 Credenciales
- **Cambiar TODA credencial por defecto en el primer arranque** (admin/seed) antes de exponer el sistema.
- Secretos por variable de entorno; rotación periódica documentada (y su efecto: p. ej. rotar el
  secreto de tokens invalida las sesiones vigentes → hacerlo en ventana de bajo uso).

### 6.4 Backup y restauración  ⚠ crítico
- Respaldo: `<cmd backup>` — **cifrado** + copia **off-site** (regla 3-2-1) + rotación. Programado: `<cron>`.
- **RPO** objetivo: `<p. ej. 24 h>` (define la frecuencia del backup). **RTO** objetivo: `<p. ej. < 1 h>`.
- **Restauración / DRILL:** `<cmd restore>` en una BD limpia; verificar conteos y MEDIR el tiempo (RTO real).
  **Un backup no probado NO cuenta como backup** — ejecutar el drill periódicamente.

### 6.5 Monitoreo y alertas
- Monitor de uptime **EXTERNO** apuntando al endpoint/URL público, con alerta al caer (detecta caídas
  totales que un monitor interno no vería).
- Probar la alerta (simular caída en staging) y registrar: URL, intervalo, contacto.

## 7. Diagnóstico de incidencias
| Síntoma | Causa probable | Acción |
|---|---|---|
| <502/503> | <servicio caído> | `<cmd status/restart>` |
| <login falla> | <token/secret/clock> | <revisar config/hora> |
| <BD no conecta> | <credenciales/red> | <revisar env/firewall> |

## 8. Contactos / escalamiento
- <rol> — <medio de contacto>.
