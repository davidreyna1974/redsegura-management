# redSegura — Sistema (repositorio `management`)

> Plataforma de microservicios para automatización de red, monitoreo y gestión de
> vulnerabilidades, sobre un único inventario de activos.

![estado](https://img.shields.io/badge/estado-en%20desarrollo-informational)
![fase](https://img.shields.io/badge/fase-Fundaci%C3%B3n-blue)
![licencia](https://img.shields.io/badge/licencia-por%20definir-lightgrey)

Este es el repositorio **umbrella** del sistema: aloja la **documentación general** del
proyecto y (en fases posteriores) la **orquestación** — `docker-compose.yml`, manifiestos de
Kubernetes y Terraform. El código de los microservicios y del dashboard vive en sus propios
repositorios.

## 🧱 Capas del sistema

| Capa | Repositorio | Tecnología | Estado |
|---|---|---|---|
| **Orquestación + documentación** (este repo) | `codigo/management/` | Docs · Docker Compose · Terraform · Kubernetes | Fundación |
| **Backend** | `codigo/backend/` | Monorepo de 10 microservicios (Java/Spring Boot + Python/FastAPI) | Fundación |
| **Frontend** | `codigo/frontend/` | Angular SPA (dashboard) | Pendiente |

---

## 🎯 Descripción

redSegura unifica **inventario de activos + postura de configuración + postura de
vulnerabilidades** en una sola fuente de verdad, combinando automatización de red y gestión de
vulnerabilidades. Roles del sistema: **Administrador**, **Operador** y **Auditor/Solo lectura**.
Especificación completa: [`documentos/proyecto_microservicios_redsegura.md`](documentos/proyecto_microservicios_redsegura.md).

## 🏗️ Arquitectura

Microservicios por *bounded context* (DDD), *database-per-service* (PostgreSQL), comunicación
asíncrona por eventos (RabbitMQ), API Gateway + Keycloak (OAuth2/OIDC), y observabilidad de tres
pilares (Prometheus/Grafana, Loki, OpenTelemetry/Jaeger).
Ver [`documentos/arquitectura/diagrama_arquitectura.md`](documentos/arquitectura/diagrama_arquitectura.md)
y [`documentos/arquitectura/memoria_tecnica_global.md`](documentos/arquitectura/memoria_tecnica_global.md).

## 📁 Estructura del sistema

```
codigo/
├── management/     este repo — documentación general + orquestación
│   └── documentos/ arquitectura · planificación · qa · sesiones · plantillas
├── backend/        monorepo de microservicios (Java + Python)
└── frontend/       Angular SPA (pendiente)
```

## 🚀 Cómo ejecutar

> El sistema aún está en **Fundación**: no hay servicios ejecutables todavía. La orquestación
> local (`docker-compose.yml`) y la infraestructura (Terraform, Kubernetes) se añadirán en fases
> posteriores. Ver el plan de trabajo:
> [`documentos/planificacion/plan_general_proyecto_redSegura.md`](documentos/planificacion/plan_general_proyecto_redSegura.md).

## ✅ Calidad y QA

- Metodología: [Protocolo de verificación en 4 fases](documentos/qa/protocolo_verificacion_4_fases.md).
- Cobertura mínima objetivo: **≥ 70 % statements** por microservicio.
- Reporte consolidado de QA: *pendiente* (se genera al certificar los servicios).

## 📚 Documentación

Índice navegable en [`documentos/README.md`](documentos/README.md).

## 🔐 Seguridad

Política de reporte de vulnerabilidades en [`SECURITY.md`](SECURITY.md) (canal privado). Las
dependencias se vigilan con **Dependabot** y escaneo en CI (SCA).

## 📄 Licencia

Por definir.

---
<sub>David Reyna Pineda · 2026</sub>
