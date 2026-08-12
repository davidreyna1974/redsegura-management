# Estado de sesión activa — redSegura

<!-- Tablero VIVO (instancia de templates/sesiones/estado_sesion_activa_TEMPLATE.md). Se actualiza al
     completar cada módulo/fase y ANTES de cerrar sesión. El handoff ejecutivo está en
     `contexto_sesion_siguiente.md` (misma carpeta). -->

**Última actualización:** 2026-08-11 · **Estado:** ⏸️ **PAUSA PROLONGADA** ·
**Ramas activas:** backend `develop`, management `develop` (ambas limpias, pusheadas).

## 1. ¿Dónde vamos?
- **Módulo/tarea en curso:** validación de fidelidad con dispositivos de red (config-backup) — Fase 0
  (plan) completa; **ejecución de niveles pendiente**.
- **Fase del proyecto:** Fundación completa · **Fase A en curso** (2 de 5 servicios certificados).
- **Objetivo al retomar:** ejecutar el **Nivel 1** (fixtures) o avanzar el **Nivel 2a** (ver
  `contexto_sesion_siguiente.md §3`).

## 2. Hecho (acumulado hasta la pausa)
- [x] **Fundación:** arquitectura global, ADR-01..17, POM padre (Java 21, Spring Boot 3.5.16),
  5 contratos OpenAPI Fase A (gobernados con Spectral), CI esqueleto, repos GitHub con branch protection.
- [x] **`asset-inventory-service`** ✅ certificado (R1.3): CRUD+búsqueda, RBAC, ETag/If-Match,
  idempotencia, dual-stack IPv4/IPv6, redacción por rol, observabilidad 3 pilares, outbox→RabbitMQ,
  import masivo async. 112 tests, CI verde (Trivy).
- [x] **`config-backup-service`** ✅ certificado (4 fases, R-C1/2/3): respaldo/versionado SSH (Netmiko),
  Git interno, drift-check, jobs async + cron, consumidor idempotente `asset.*` + Pact consumidor,
  outbox→relay, idempotencia de escritura, filtros de listado, eventos `config.*` con envelope+JSON
  Schema. 105 tests, cobertura 95%, CI con SCA/Trivy.
- [x] **RNF-32** integración cross-service obligatoria: `matriz_interaccion.md` + gate
  `check_golden_paths.py` + DoD #8 + template + L-QA-09. Golden path `asset→config-backup` ejecutado (6/6).
- [x] **Plan de validación de dispositivos (Fase 0):** `plan_validacion_dispositivos.md` +
  `guia_captura_fixtures.md` + `guia_captura_devnet.md`.
- [x] **Checkpoint de pausa** (esta sesión): documentos de sesión + actualización de CLAUDE.md,
  CHANGELOGs, reporte_qa, plan_general + memoria de Claude Code.

## 3. Próximo paso concreto (lo primero al retomar)
1. Leer `contexto_sesion_siguiente.md` (handoff) — sección §3 (punto de retomar).
2. Confirmar con el usuario la bifurcación **(A) Nivel 1 / (B) Nivel 2a / (C) compliance-audit-service**.
3. Si (A): pedir/recibir capturas DevNet redactadas → crear `fixtures/` + tests de replay +
   `procedimiento_nivel1_fixtures.md` + `matriz_compatibilidad.md`.

## 4. Estado del gatekeeper (último conocido)
| Servicio | Comando | Último resultado |
|---|---|---|
| `asset-inventory-service` | `mvn -pl asset-inventory-service -am clean verify` | ✅ 112 tests, cobertura ≥70%, 0 lint (CI verde) |
| `config-backup-service` | `ruff check . && mypy . && pytest --cov` | ✅ 105 tests, cobertura 95%, 0 lint (CI verde) |
| Gate de release | `scripts/check_golden_paths.py` + `check_casos_completos.py` | ✅ 1 vector EJECUTADO, 4 DIFERIDOS; 0 casos ⏳ |

> **Nota:** no se re-ejecutó el gatekeeper en esta sesión (solo cambios documentales, sin tocar código).

## 5. Estado de git / repos
- **backend** (`redsegura-backend`): rama `develop`, sincronizada con origin, working tree **limpio**.
- **management** (`redsegura-management`): rama `develop`, working tree **limpio** tras este checkpoint.
- **frontend**: directorio **vacío** (aún no iniciado; E2E Playwright vivirá ahí).

## 6. Decisiones pendientes / bloqueos
- **Bifurcación de arranque** (A/B/C) — decisión del usuario.
- **Nivel 1 bloqueado por insumo externo:** capturas reales del usuario (Cisco DevNet).
- **Nivel 2b bloqueado por coordinación:** el usuario debe levantar GNS3 (VM GCP → VPN → GUI).
- **Multi-vendor del conector:** decisión de diseño (device_type por dispositivo vs. NAPALM) —
  prerrequisito del Nivel 2a.

## 7. Notas de contexto que NO están en el código
- **Mandato de producto:** redSegura debe ser **productivo real, sin atajos "MVP"**, y **abierto/
  multi-vendor** (alcance = dispositivos más usados de la industria; el inventario del cliente quedó
  **descartado** como criterio de alcance, decisión 2026-07-25).
- **Cuenta Cisco DevNet:** usar **Cisco.com (CCO)**, no NetAcad (el usuario tiene NetAcad).
- **GNS3 en GCP:** handshake usuario→Claude documentado en `contexto_sesion_siguiente.md §7`.
- **Deuda de producción registrada:** ver `preparacion_produccion.md` y memorias técnicas de cada
  módulo (JWT iss/aud ya cerrado en asset; exportadores de observabilidad por entorno; publisher
  confirms; búsqueda insensible a acentos BSRCH-02; etc.).
