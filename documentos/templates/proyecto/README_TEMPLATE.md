# <NOMBRE DEL PROYECTO>

<!-- GUÍA: el README es la portada del repo. Que en 30 segundos se entienda QUÉ es, cómo se ejecuta
     y qué tan serio es el proyecto (calidad/QA). Borra secciones que no apliquen a tu tipo de proyecto. -->

> <Una línea: qué hace el sistema y para quién.>

![versión](https://img.shields.io/badge/versión-<X.Y.Z>-blue)
![estado](https://img.shields.io/badge/estado-<en%20desarrollo>-informational)
![<tech>](https://img.shields.io/badge/<tech>-<versión>-informational)
![tests](https://img.shields.io/badge/tests-<N>%20passing-success)
![CI](https://github.com/<owner>/<repo>/actions/workflows/ci.yml/badge.svg)
![CD](https://github.com/<owner>/<repo>/actions/workflows/cd.yml/badge.svg)
![cobertura](https://img.shields.io/badge/cobertura-<N>%25-success)
![licencia](https://img.shields.io/badge/licencia-<MIT>-lightgrey)
<!-- GUÍA: los badges de CI/CD son EN VIVO (GitHub Actions) — reflejan la última corrida real de
     cada workflow. El de tests/cobertura son estáticos (shields.io): actualízalos a mano.
     El badge de versión (`<X.Y.Z>`) debe coincidir con el ÚLTIMO tag/release publicado (SemVer):
     actualízalo en el MISMO commit de release en que actualizas el CHANGELOG, antes de tagear.
     El badge de `estado` refleja el ciclo de vida: en desarrollo → estable · mantenimiento (al cerrar
     el proyecto; ver planificacion/acta_cierre_proyecto_TEMPLATE.md). -->


[opcional: si es un sistema multi-repo, enlaza las otras capas]

| Capa | Repositorio | Tecnología |
|---|---|---|
| <este repo> | `<nombre>` | <stack> |

---

## 📋 Tabla de contenidos
- [Descripción](#-descripción) · [Stack](#-stack-tecnológico) · [Arquitectura](#-arquitectura)
- [Características](#-características) · [Cómo ejecutar](#-cómo-ejecutar) · [Calidad y QA](#-calidad-y-qa)
- [Estructura](#-estructura-del-proyecto) · [Documentación](#-documentación) · [Licencia](#-licencia)

## 🎯 Descripción
<2-4 líneas: problema, alcance, usuarios/roles.>

## 🛠️ Stack tecnológico
- Lenguaje/Framework: <...>
- Persistencia: <...>
- Tests: <unit> + <e2e>
- <otros: auth, build, CI>

## 🏗️ Arquitectura
<1 párrafo + enlace al diagrama.> Ver [`docs/arquitectura/diagrama_arquitectura.md`](docs/arquitectura/diagrama_arquitectura.md).

```
<árbol de carpetas resumido>
```

## ✨ Características
- <característica clave 1>
- <característica clave 2 — p.ej. RBAC de N roles>

## 🚀 Cómo ejecutar
### Requisitos
- <...>
### Pasos
```bash
<cmd install>
<cmd run>      # <URL local>
<cmd test>
<cmd build>
```
[opcional: usuarios de prueba]

## ✅ Calidad y QA
- **<N> tests** · 0 fallos · **<cobertura %>**.
- **<N> casos de prueba** documentados (campaña de QA de 4 fases).
- Reporte: [`docs/qa/reporte_qa.md`](docs/qa/reporte_qa.md).

## 📁 Estructura del proyecto
```
<repo>/
├── src/        <código>
├── docs/       documentación (ver docs/README.md)
├── CLAUDE.md   contexto para Claude Code / convenciones del repo
├── CHANGELOG.md
└── README.md
```

## 📚 Documentación
Indexada en [`docs/README.md`](docs/README.md).

## 🔐 Seguridad
Política de reporte de vulnerabilidades en [`SECURITY.md`](SECURITY.md) (canal privado). Las
dependencias se vigilan con **Dependabot** y escaneo en CI (SCA).

## 📄 Licencia
Distribuido bajo licencia **<MIT>**. Ver [`LICENSE`](LICENSE).

---
<sub><Autor> · <año></sub>
