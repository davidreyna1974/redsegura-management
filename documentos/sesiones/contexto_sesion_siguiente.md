# Contexto para la sesión siguiente — redSegura

<!-- Handoff explícito de traspaso entre sesiones (instancia de templates/sesiones/
     contexto_sesion_siguiente_TEMPLATE.md). Documento EJECUTIVO y maestro: la próxima sesión de
     Claude Code lo lee PRIMERO para retomar sin perder contexto. El tablero detallado vivo está en
     `estado_sesion_activa.md` (misma carpeta). -->

**Fecha de cierre de sesión:** 2026-08-11 · **Motivo:** pausa prolongada del desarrollo.
**Próxima sesión arranca con:** decidir el arranque del **Nivel 1** de validación de dispositivos
(fixtures) — el usuario envía capturas reales de Cisco DevNet, o se avanza el **Nivel 2a**.

---

## 1. Resumen ejecutivo del avance (dónde estamos)

**redSegura** = plataforma de microservicios (10 servicios, monorepo poliglota Java/Python) para
automatización, monitoreo y gestión de vulnerabilidades de red. **Fundación completa; Fase A en curso.**

| Bloque | Estado |
|---|---|
| **Fundación** (arquitectura global, ADR-01..17, POM padre, 5 contratos OpenAPI Fase A, CI, repos GitHub) | ✅ **Completa** |
| `asset-inventory-service` (Fase A, Java) | ✅ **Implementado y QA certificado** (R1.3, 112 tests, cobertura ≥70%, CI verde con Trivy) |
| `config-backup-service` (Fase A, Python) | ✅ **Implementado y QA certificado** (4 fases, R-C1/2/3, 105 tests, cobertura 95%, CI con SCA/Trivy) |
| `compliance-audit-service`, `alerting-service`, `notification-service` (Fase A) | ⬜ **Sin iniciar** (placeholders: solo `openapi.yaml` de la fundación) |
| 5 servicios de **Fase B** | ⬜ Sin iniciar |
| **RNF-32** (integración cross-service obligatoria + gate `check_golden_paths.py`) | ✅ Implementado; 1.er golden path `asset-inventory → config-backup` **ejecutado** (6/6) |
| **Plan de validación de dispositivos** (fidelidad SSH multi-vendor) | ✅ **Fase 0 (plan) completa** + guías de captura. **Ejecución pendiente** (ver §3) |

**Lo último que se hizo en esta sesión:** se elaboró el **plan maestro de validación de dispositivos**
(Fase 0) y **dos guías de captura de fixtures** (general + Cisco DevNet). Todo commiteado y pusheado.

## 2. Lo primero que debe hacer la próxima sesión

1. **Leer** este documento y `estado_sesion_activa.md` (tablero detallado).
2. **Leer** el plan y las guías de validación (raíz del trabajo pendiente):
   `codigo/backend/documentos/validacion_dispositivos/` →
   `plan_validacion_dispositivos.md`, `guia_captura_fixtures.md`, `guia_captura_devnet.md`.
3. **Verificar entorno/git:**
   ```bash
   cd "codigo/backend" && git status && git log --oneline -3          # esperado: develop limpio
   cd "codigo/management" && git status && git log --oneline -3        # esperado: develop limpio
   ```
4. **Retomar en el punto de decisión** (§3): esperar capturas DevNet del usuario (Nivel 1) o avanzar
   el Nivel 2a (implica el cambio multi-vendor del conector).

## 3. Punto exacto de retomar — la escalera de validación de dispositivos

La validación de fidelidad con dispositivos reales es **una escalera de 3 niveles** (plan §6). Estado:

| Nivel | Qué valida | Estado | Bloqueo / disparador |
|---|---|---|---|
| **1 · Fixtures (replay)** | Parsing de output real (banner, paginación, `end`, running≠startup, secretos) | 🟡 **Listo para ejecutar** | **Espera capturas reales del usuario** (vía Cisco DevNet: IOS-XE + NX-OS, gratis — ver `guia_captura_devnet.md`) |
| **2a · NOS libre emulado** | Mecanismo completo SSH→Git→drift contra NOS real contenedor-nativo (no-Cisco) | 🔵 Pendiente | **Prerrequisito:** soporte **multi-vendor del conector** (hoy `device_type` global) — ver §5 |
| **2b · Cisco emulado (IOSv/GNS3)** | Fidelidad Cisco licenciada | 🔵 Pendiente | Coordinación GNS3 (usuario: VM GCP → VPN → GUI → me pasa endpoint API) |

**Bifurcación de decisión al retomar (elige el usuario):**
- **(A)** El usuario envía ≥1 captura DevNet redactada → **ejecuto el Nivel 1** (creo `fixtures/`,
  tests de replay, `procedimiento_nivel1_fixtures.md`, actualizo `matriz_compatibilidad.md`).
- **(B)** Avanzo el **Nivel 2a**: implemento `device_type` **por dispositivo** en el conector +
  smoke con un NOS libre (SR Linux / VyOS / cEOS / FRR) en Docker local.
- **(C)** Se pospone la validación y se arranca `compliance-audit-service` (nace declarando su vector
  `asset.*` en la propuesta y con el gate RNF-32 activo).

## 4. Trabajo en progreso (no terminado)

| Tema | Estado | Qué falta |
|---|---|---|
| Validación de dispositivos Nivel 1 | Andamiaje listo (plan + guías) | Capturas reales → fixtures + tests replay + reporte + matriz |
| Validación Nivel 2a | Diseñado en el plan §11 | Implementar multi-vendor del conector + smoke NOS libre |
| Entregables del plan aún no creados | — | `matriz_compatibilidad.md`, `procedimiento_nivel1/2a/2b_*.md`, carpeta `fixtures/` |
| Pact de `asset-inventory` (productor) | Diferido | Se activa: ya existe consumidor (`config-backup` consume `asset.*`) |

## 5. Nota de diseño clave (multi-vendor del conector)

`config-backup` hoy usa un **`device_type` global** (`CBS_SSH_DEVICE_TYPE`=`cisco_ios`) en
`app/connectors/netmiko_connector.py` y comandos fijos `show running-config`/`show startup-config`.
El alcance **multi-vendor amplio** (decisión de producto: redSegura abierto/compatible con lo más
usado en la industria, **no** el inventario de un cliente) exige **`device_type` por dispositivo**,
derivado del `vendor`/`model` que ya llega por eventos `asset.*`. Alternativa a evaluar: **NAPALM**
(`get_config()` uniforme). Es **prerrequisito del Nivel 2a**; registrado como deuda con disparador
INT-SYNC. Ver plan §11.

## 6. Cosas a NO hacer / cuidado

- **NUNCA commitear directo a `main`/`develop`.** Rama `feature/`/`fix/`/`chore/`/`docs/` + `merge --no-ff`.
- **RNF-07 (alcance de red):** en Nivel 2, `config-backup` solo conecta a IPs de la **red simulada**
  dentro de `CBS_ALLOWED_SCAN_CIDRS`; **jamás** a redes de terceros. (En Nivel 1 no aplica: la captura
  la hace el usuario a mano, el servicio no se conecta.)
- **Secretos:** las capturas de fixtures se **redactan** antes de versionarse (RNF-06/17).
- No dar un servicio por "done" sin las **8 condiciones de DoD** (CLAUDE.md) — incluye golden path RNF-32.

## 7. Entorno / credenciales de prueba (no secretos reales)

- **Cisco DevNet:** requiere **cuenta Cisco.com (CCO)**, NO NetAcad (son identidades distintas; ambas
  gratis). Sandboxes always-on con **credenciales dinámicas por lanzamiento** (portal "Quick Access").
- **GNS3:** servidor en **GCP**; el usuario lo levanta (VM → túnel VPN → GUI) y me pasa endpoint API +
  versión + credenciales. La IP de gestión del equipo emulado debe caer en `CBS_ALLOWED_SCAN_CIDRS`.
- **Entorno local:** `docker-compose.dev.yml` (raíz backend) + `deploy/` (realm Keycloak sembrado).
  Roles: **Administrador, Operador, Auditor/Solo lectura**.

## 8. Preguntas abiertas para el usuario (al retomar)

- ¿Arrancamos por **(A)** capturas DevNet (Nivel 1), **(B)** Nivel 2a (multi-vendor + NOS libre), o
  **(C)** `compliance-audit-service`?
- Para Nivel 1: ¿capturas de **IOS-XE + NX-OS** vía DevNet? ¿Incluimos el fixture "oro" running≠startup
  (requiere un Reserved sandbox con VPN)?

---

> **Referencias maestras:** especificación (RF/RNF/RBAC/arquitectura) →
> `management/documentos/proyecto_microservicios_redsegura.md`. Plan de ejecución →
> `management/documentos/planificacion/plan_general_proyecto_redSegura.md`. Reglas de trabajo →
> `backend/CLAUDE.md`. QA consolidado → `management/documentos/qa/reporte_qa.md`. Preparación a
> producción y deuda → `management/documentos/arquitectura/preparacion_produccion.md`.
