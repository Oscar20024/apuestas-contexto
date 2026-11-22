# Arquitectura

## Vista general

```mermaid
flowchart TB
    subgraph Fuentes externas
        FD[football-data.co.uk<br/>historial CSV]
        OP[OddsPapi<br/>Apuesta Total + Pinnacle]
    end

    subgraph VPS Hostinger
        N8N[n8n<br/>horarios y envío]
        API[apuestas-api<br/>Docker · FastAPI · Python]
        OC[OpenClaw + Gemini]
    end

    SB[(Supabase<br/>Postgres)]
    GH[GitHub<br/>apuestas-sistema · apuestas-contexto]
    WA[WhatsApp de Oscar]
    APP[App Android<br/>fase 4]

    N8N -- HTTP + token --> API
    API --> FD
    API --> OP
    API <--> SB
    N8N -- mensajes --> OC --> WA
    SB -.-> APP
    GH -. git pull .-> API
```

## Componentes

| Componente | Responsabilidad | Dónde |
|---|---|---|
| `apuestas-api` | Todo el cálculo: descarga, modelo, embudo, mensajes | Docker en el VPS, solo en `127.0.0.1:8010` |
| n8n | Horarios (cron), llamar a la API, reenviar mensajes, avisar errores | VPS |
| Supabase | Partidos, cuotas, modelos, evaluaciones, apuestas reales, uso de API | Nube (proyecto propio o esquema `apuestas` en uno existente) |
| OpenClaw + Gemini | Entregar el mensaje por WhatsApp | VPS |
| GitHub | Código versionado (`apuestas-sistema`) y contexto (`apuestas-contexto`) | Nube |

n8n no calcula nada: si mañana se cambia n8n por otra cosa, el sistema sigue igual.
Ver [ADR 0004](../02-decisiones/0004-n8n-solo-orquesta.md).

Todos los mensajes salen por un único flujo de n8n (**98 - Enviar mensaje**), que entrega a
WhatsApp por comando de OpenClaw, por su webhook o por otro webhook.
Ver [ADR 0009](../02-decisiones/0009-envio-centralizado.md).

Puntos de conexión y cómo probarlos: [06-guias/conexiones.md](../06-guias/conexiones.md).

## Flujo semanal

```mermaid
sequenceDiagram
    participant N as n8n
    participant A as apuestas-api
    participant S as Supabase
    participant O as OddsPapi
    participant W as WhatsApp

    Note over N: Lunes 7:00
    N->>A: POST /historial/actualizar
    A->>S: resultados + cuotas de cierre
    N->>A: POST /modelo/entrenar
    A->>S: modelo nuevo (activo)
    N->>A: GET /resumen/semanal
    N->>W: resumen

    Note over N: Mar-jue 9:00 · Vie 12:00 · Sáb-dom 6:00 y 10:00
    N->>A: POST /escanear {horizonte_horas}
    A->>O: cuotas (2 consultas)
    A->>S: partidos, cuotas, evaluaciones
    A-->>N: alertas + mensajes
    N->>W: una alerta por mensaje
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

- La API no se publica a internet: escucha en `127.0.0.1`. Solo n8n (en el mismo VPS) la llama.
- Todas las rutas (menos `/salud`) piden la cabecera `X-API-Token`.
- RLS activado en todas las tablas de Supabase. El backend entra con el usuario `postgres`;
  la futura app usará políticas de solo lectura.
- Secretos únicamente en `.env` del VPS. `.gitignore` lo excluye.
