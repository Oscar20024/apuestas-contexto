# Preguntas abiertas

Lo que falta definir. Al resolver una, moverla a "Resueltas" con la fecha y la respuesta.

## Pendientes

| # | Pregunta | Por qué importa | Valor provisional |
|---|---|---|---|
| 1 | **Supabase:** ¿pausar un proyecto y crear `apuestas` (A), usar uno existente con esquema propio (B) o pasar a Pro (C)? | El plan gratis ya tiene 2 proyectos activos (el máximo) | B: `DB_SCHEMA=apuestas` |
| 2 | **n8n:** ¿está instalado directo en el VPS o en Docker? | Define `api_url` y qué modo de envío se puede usar | Directo: `http://127.0.0.1:8010` |
| 3 | **OpenClaw:** ¿el comando `openclaw message send` funciona en el VPS, o hay que usar su webhook (`/hooks/agent`)? | Modo `comando` entrega el texto exacto; el webhook pasa por Gemini | `comando` |
| 4 | ¿Número de WhatsApp que recibe las alertas? | Va en el flujo 98 | — |
| 5 | ¿Qué ligas te interesan? ¿Incluir Liga 1 de Perú? | Liga 1 no está en football-data: hace falta otra fuente | Premier, LaLiga, Serie A |
| 6 | ¿Con qué monto de fondo empezarías (soles) y cuál es la apuesta mínima en Apuesta Total? | Define los montos sugeridos | S/ 500 · mínimo S/ 2 |
| 7 | ¿Qué plan de VPS tienes (KVM 1/2/4)? | Memoria para Docker + n8n + OpenClaw | — |
| 8 | ¿Registrar apuestas desde WhatsApp (responder "aposte 10 a 2.08")? | Sin registro, el límite de pérdida no funciona | Por comando en el VPS |

## Resueltas

| Fecha | Pregunta | Respuesta |
|---|---|---|
| 2026-11-22 | ¿Qué tipo de apuesta? | Ganador del partido (local o visita) |
| 2026-11-22 | ¿Qué días? | Sábado y domingo; entre semana solo si la ventaja es alta |
| 2026-11-22 | ¿Canal de alertas? | WhatsApp vía OpenClaw primero; app Android después |
| 2026-11-22 | ¿Dónde vive el código? | GitHub (Oscar20024), dos repos privados: `apuestas-sistema` y `apuestas-contexto` |
