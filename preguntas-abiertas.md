# Preguntas abiertas

Lo que falta definir. Al resolver una, moverla a "Resueltas" con la fecha y la respuesta.

## Pendientes

| # | Pregunta | Por qué importa | Valor provisional |
|---|---|---|---|
| 2 | ¿Incluir Liga 1 de Perú u otras ligas? | Ya hay 5 ligas activas = el máximo que cabe en 1 consulta por casa; una 6.ª liga duplica las consultas y no alcanza el plan gratis. Además el escaneo exige historial en football-data (nombres de equipos y modelo): Liga 1 de Perú no está ahí | E0, SP1, I1, D1, F1 |
| 3 | ¿Con qué monto de fondo empezarías (soles) y cuál es la apuesta mínima en Apuesta Total? | Define los montos sugeridos | S/ 500 · mínimo S/ 2 |
| 4 | ¿Registrar apuestas desde WhatsApp (responder "aposte 10 a 2.08")? | Sin registro, el límite de pérdida no funciona | Por comando en el VPS |
| 5 | ¿Agregar al resumen del lunes cuántas veces Apuesta Total pagó más que el precio justo? | Saber en 1–2 semanas si la estrategia tiene futuro, aunque no haya alertas | Propuesto; mientras, la consulta SQL de `03-datos` |
| 6 | Cambiar la contraseña de la base (apareció en el chat) | Seguridad | Pendiente: actualizar `.env` del VPS y de la PC y recrear el contenedor |

## Resueltas

| Fecha | Pregunta | Respuesta |
|---|---|---|
| 2026-11-22 | ¿Qué tipo de apuesta? | Ganador del partido (local o visita) |
| 2026-11-22 | ¿Qué días? | Sábado y domingo; entre semana solo si la ventaja es alta |
| 2026-11-22 | ¿Canal de alertas? | WhatsApp vía OpenClaw primero; app Android después |
| 2026-11-22 | ¿Dónde vive el código? | GitHub (Oscar20024), dos repos privados: `apuestas-sistema` y `apuestas-contexto` |
| 2026-11-22 | ¿Qué proyecto de Supabase? | Proyecto nuevo `BDapuestas` en otra cuenta, esquema `public`. Tablas creadas |
| 2026-11-22 | ¿n8n? | No está instalado. Horarios con systemd y envío con el mensajero (ADR 0010). n8n queda opcional |
| 2026-11-22 | ¿Cómo es el VPS? | `srv1864174`: 1 vCPU, 3.8 GB RAM + 2 GB swap, Docker; OpenClaw, memory-api, riachuelo y Traefik ya corren ahí |
| 2026-11-22 | ¿El envío por `docker exec ... openclaw message send` funciona? | Sí, después de agregar el número con `+51` a `channels.whatsapp.allowFrom` (se mantuvo el que ya estaba). Ver runbook |
| 2026-11-22 | ¿Contraseña de la base? | Reseteada en Database → Settings; conexión verificada desde el VPS |
| 2026-11-22 | ¿Número de WhatsApp? | Configurado en `/etc/apuestas/host.env` por `instalar.sh`; envío probado |
| 2026-11-22 | ¿El modelo propio sirve para decidir? | No: el backtest real pierde frente a Pinnacle. Se decide con el precio justo de Pinnacle + Betfair (ADR 0011, 0012) |
| 2026-11-22 | ¿Qué casa usar como segunda referencia? | `betfair-ex` (sbobet viejo y caro; singbet sin datos) |
| 2026-11-22 | ¿Activar más ligas? | Sí: Bundesliga (D1, torneo 35) y Ligue 1 (F1, torneo 34). Sin consultas extra (5 ligas por consulta). ~48 partidos por jornada en vez de ~30 |
