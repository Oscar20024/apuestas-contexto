# Preguntas abiertas

Lo que falta definir. Al resolver una, moverla a "Resueltas" con la fecha y la respuesta.

## Pendientes

| # | Pregunta | Por qué importa | Valor provisional |
|---|---|---|---|
| 1 | ¿Qué ligas te interesan? ¿Incluir Liga 1 de Perú? | Liga 1 no está en football-data: hace falta otra fuente de historial | Premier, LaLiga, Serie A |
| 2 | ¿Qué plan de VPS tienes (KVM 1/2/4) y cómo está instalado n8n (Docker o directo)? | Cambia la URL que usa n8n y la red de Docker | `127.0.0.1:8010` |
| 3 | ¿Cómo recibe OpenClaw un mensaje para mandarlo a tu WhatsApp? (webhook, API, comando) | El último nodo de n8n depende de esto | POST `{"mensaje": "..."}` |
| 4 | ¿Con qué monto de fondo empezarías (en soles)? | Define los montos sugeridos | S/ 500 |
| 5 | ¿Cuál es la apuesta mínima en Apuesta Total? | Montos muy chicos pueden no aceptarse | S/ 2 |
| 6 | ¿Supabase gratis o Pro? ¿Proyecto nuevo solo para esto? | Espacio y pausas por inactividad | Gratis, proyecto nuevo |
| 7 | ¿Nombres de los repositorios en GitHub? | Para la guía y el `git clone` del VPS | `apuestas-sistema` y `apuestas-contexto` |
| 8 | ¿Quieres poder registrar tus apuestas desde WhatsApp (responder "aposte 10 a 2.08")? | Sin registro, el límite de pérdida no funciona | Por comando en el VPS |

## Resueltas

| Fecha | Pregunta | Respuesta |
|---|---|---|
| 2026-11-22 | ¿Qué tipo de apuesta? | Ganador del partido (local o visita) |
| 2026-11-22 | ¿Qué días? | Sábado y domingo; entre semana solo si la ventaja es alta |
| 2026-11-22 | ¿Canal de alertas? | WhatsApp vía OpenClaw primero; app Android después |
| 2026-11-22 | ¿Dónde vive el código? | GitHub, dos repos: sistema y contexto |
