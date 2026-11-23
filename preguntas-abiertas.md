# Preguntas abiertas

Lo que falta definir. Al resolver una, moverla a "Resueltas" con la fecha y la respuesta.

## Pendientes

| # | Pregunta | Por qué importa | Valor provisional |
|---|---|---|---|
| 0 | **Contraseña de la base de `BDapuestas`**: la actual falla. Resetearla y actualizar `.env` (VPS y PC) | Sin base no hay datos ni alertas | — |
| 1 | ¿Número de WhatsApp que recibe las alertas? | Lo pide `vps/instalar.sh` (debe estar en `allowFrom` de OpenClaw con `+51`) | El tuyo |
| 2 | ¿Qué ligas te interesan? ¿Incluir Liga 1 de Perú? | Liga 1 no está en football-data: hace falta otra fuente | Premier, LaLiga, Serie A |
| 3 | ¿Con qué monto de fondo empezarías (soles) y cuál es la apuesta mínima en Apuesta Total? | Define los montos sugeridos | S/ 500 · mínimo S/ 2 |
| 4 | ¿Registrar apuestas desde WhatsApp (responder "aposte 10 a 2.08")? | Sin registro, el límite de pérdida no funciona | Por comando en el VPS |

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
