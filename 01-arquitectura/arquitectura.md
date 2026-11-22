# Arquitectura

## Vista general

```mermaid
flowchart TB
    subgraph Fuentes externas
        FD[football-data.co.uk<br/>historial CSV]
        OP[OddsPapi<br/>Apuesta Total + Pinnacle]
    end

    subgraph VPS["VPS Hostinger srv1864174 (1 vCPU, 3.8 GB)"]
        T[systemd timers<br/>hora de Lima]
        H[apuestas_host.py<br/>programador + mensajero]
        API[apuestas-api<br/>Docker · FastAPI · 127.0.0.1:8010]
        OC[OpenClaw<br/>openclaw-wr93-openclaw-1]
    end

    SB[(Supabase BDapuestas<br/>Postgres)]
    GH[GitHub<br/>apuestas-sistema · apuestas-contexto]
    WA[WhatsApp de Oscar]
    APP[App Android<br/>fase 4]

    T --> H
    H -- "POST /tareas/* (token)" --> API
    API --> FD
    API --> OP
    API <--> SB
    H -- "GET /mensajes/pendientes" --> API
    H -- "docker exec ... message send" --> OC --> WA
    SB -.-> APP
    GH -. git pull .-> API
```

## Componentes

| Componente | Responsabilidad | Dónde |
|---|---|---|
| `apuestas-api` | Todo el cálculo: descarga, modelo, embudo, mensajes (los deja en la bandeja) | Docker en el VPS, solo en `127.0.0.1:8010`, UID 10001, 768 MB / 0.5 CPU |
| Programador | A cada hora fija pide una tarea a la API | Timers de systemd en el VPS |
| Mensajero | Entrega la bandeja por WhatsApp con `docker exec` a OpenClaw; reintenta | `vps/apuestas_host.py` en el VPS (root) |
| Supabase | Partidos, cuotas, modelos, evaluaciones, bandeja de mensajes, apuestas reales, uso de API | Proyecto `BDapuestas` (us-west-2) |
| OpenClaw | Sesión de WhatsApp; entrega el texto tal cual | Contenedor existente del VPS |
| GitHub | Código versionado (`apuestas-sistema`) y contexto (`apuestas-contexto`) | Nube |
| n8n (opcional) | Podría reemplazar a los timers; no está instalado | — |

El orquestador no calcula: si mañana se cambian los timers por n8n u otra cosa, el sistema sigue igual.
Ver [ADR 0004](../02-decisiones/0004-n8n-solo-orquesta.md) y [ADR 0010](../02-decisiones/0010-programador-systemd-y-mensajero.md).

Puntos de conexión y cómo probarlos: [06-guias/conexiones.md](../06-guias/conexiones.md).

## Flujo semanal

```mermaid
sequenceDiagram
    participant T as systemd
    participant H as apuestas_host.py
    participant A as apuestas-api
    participant S as Supabase
    participant O as OddsPapi
    participant C as OpenClaw
    participant W as WhatsApp

    Note over T: Lunes 7:00
    T->>H: tarea lunes
    H->>A: POST /tareas/lunes
    A->>S: resultados, modelo nuevo, resumen a la bandeja

    Note over T: Mar-jue 9:00 · Vie 12:00 · Sáb-dom 6:00 y 10:00
    T->>H: tarea viernes (por ejemplo)
    H->>A: POST /tareas/viernes
    A->>O: cuotas (2 consultas)
    A->>S: partidos, cuotas, evaluaciones, alertas a la bandeja
    H->>A: GET /mensajes/pendientes
    H->>C: docker exec openclaw message send
    C->>W: alerta (texto exacto)
    H->>A: POST /mensajes/{id}/resultado

    Note over T: Cada 5 min
    T->>H: enviar (reintentos de lo que quedó pendiente)
```

## Embudo de decisión (dentro de `apuestas-api`)

```mermaid
flowchart TB
    E[Partido + cuotas] --> P1{1. ¿Datos suficientes?<br/>≥10 partidos por equipo}
    P1 -- no --> D1[Descarte: pocos_datos]
    P1 -- sí --> P2[2. Probabilidad del modelo<br/>60% Dixon-Coles + 40% Elo]
    P2 --> P3{3. ¿Hay valor?<br/>EV ≥ 5% finde / 8% entre semana}
    P3 -- no --> D3[Descarte: sin_valor]
    P3 -- sí --> P4{4. Riesgo<br/>cuota 1.50–3.50 · diferencia ≤15 pts<br/>Pinnacle no se movió en contra}
    P4 -- no --> D4[Descarte: cuota_fuera_rango /<br/>ventaja_sospechosa / mercado_en_contra]
    P4 -- sí --> P5[5. Monto: Kelly 25%, tope 2%]
    P5 --> L{Límites globales<br/>3 alertas/día · 10% semanal<br/>pausa si −15% en el mes}
    L --> AL[Alerta]
```

Detalle de cada paso: [04-modelo/modelo-y-embudo.md](../04-modelo/modelo-y-embudo.md).

## Seguridad

- La API no se publica a internet: escucha en `127.0.0.1` (el VPS tiene UFW desactivado, así que nada en `0.0.0.0`).
- El mensajero es el único que toca WhatsApp, y solo ejecuta un comando fijo sin shell.
- La API corre con UID 10001, distinto del 1000 que comparten OpenClaw y memory-api.
- Todas las rutas (menos `/salud`) piden la cabecera `X-API-Token`.
- RLS activado en todas las tablas de Supabase. El backend entra con el usuario `postgres`;
  la futura app usará políticas de solo lectura.
- Secretos únicamente en `.env` del VPS. `.gitignore` lo excluye.
