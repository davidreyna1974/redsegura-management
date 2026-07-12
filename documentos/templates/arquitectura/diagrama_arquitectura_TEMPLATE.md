# Diagrama de arquitectura — <NOMBRE DEL PROYECTO>

<!-- GUÍA: usa Mermaid (se renderiza en GitHub/GitLab; no requiere imágenes). Mantén 3-5 diagramas
     máximo: capas, estructura interna, un flujo clave y (si aplica) máquinas de estado.
     Borra los que no apliquen a tu tipo de proyecto. -->

> Diagramas en [Mermaid](https://mermaid.js.org/).

## Vista de capas / componentes

```mermaid
flowchart TB
    subgraph Cliente["<Cliente / Frontend / Consumidor>"]
        UI["<UI / Caller>"]
    end
    subgraph Servicio["<Backend / Servicio / Aplicación>"]
        API["<Puntos de entrada (Controllers/Handlers/CLI)>"]
        SEC["<Seguridad: Auth + Autorización>"]
        BIZ["<Lógica de negocio>"]
        DAT["<Acceso a datos>"]
        API --> SEC --> BIZ --> DAT
    end
    DB[("<Persistencia / Almacén>")]
    UI -- "<protocolo, p.ej. HTTPS + token>" --> API
    DAT --> DB
```

## Estructura interna de módulos

```mermaid
flowchart LR
    subgraph nucleo["núcleo / core"]
        A["<auth / seguridad>"]
        B["<config>"]
        C["<utilidades compartidas>"]
    end
    subgraph modulos
        M1["<módulo 1>"]
        M2["<módulo 2>"]
        M3["<módulo 3>"]
    end
    modulos --> nucleo
```

## Flujo clave: <p.ej. autenticación / petición principal / pipeline>

```mermaid
sequenceDiagram
    participant U as <Actor>
    participant S as <Sistema>
    participant D as <Dependencia/BD>
    U->>S: <acción>
    S->>D: <consulta/comando>
    D-->>S: <resultado>
    alt <caso autorizado/feliz>
        S-->>U: <respuesta 200>
    else <caso no autorizado/error>
        S-->>U: <respuesta 4xx>
    end
```

## Máquina(s) de estado [opcional, si hay entidades con ciclo de vida]

```mermaid
flowchart LR
    E1[<ESTADO_INICIAL>] --> E2[<ESTADO_INTERMEDIO>] --> E3[<ESTADO_FINAL>]
    E1 --> EX[<CANCELADO/ERROR>]
    E2 --> EX
```
