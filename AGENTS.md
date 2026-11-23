# Contexto para agentes de IA

## Qué es el proyecto

Sistema personal de Oscar que busca apuestas de **ganador del partido (1X2)** con valor en
**Apuesta Total** (Perú). El precio justo de cada resultado sale de **Pinnacle y Betfair Exchange sin
margen**; si Apuesta Total paga más que ese precio, el sistema manda una alerta por WhatsApp.
Un modelo propio (Dixon-Coles + Elo) se muestra solo como dato informativo (ADR 0011).
**Oscar apuesta a mano**; el sistema nunca apuesta. Versión actual: **v1.4.2**.

- Código: repositorio `apuestas-sistema` (Python 3.12, FastAPI, Docker).
- Datos: Supabase (Postgres).
- Orquestación: timers de systemd en el VPS de Hostinger (`srv1864174`). n8n es opcional y no está instalado.
- Mensajes: la API los deja en `mensajes_salida`; `vps/apuestas_host.py` los entrega con `docker exec openclaw-wr93-openclaw-1 openclaw message send`. Más adelante, quizás una app Android.
- Base de datos: proyecto Supabase `BDapuestas` (esquema `public`).
- Ligas activas: Premier (E0), LaLiga (SP1), Serie A (I1), Bundesliga (D1), Ligue 1 (F1). 5 = máximo por consulta de OddsPapi.
- Zona horaria: America/Lima. Moneda: soles (PEN).

## Reglas que no se rompen

1. **El sistema no apuesta ni maneja dinero.** No agregar integraciones que hagan apuestas automáticas.
2. **Los números de una alerta salen del sistema, sin reescribir.** Gemini/OpenClaw puede dar formato
   o contexto, nunca cambiar una cuota, probabilidad o monto.
3. **Valor, no "alta probabilidad".** Un pick existe solo si `p_justa × cuota − 1 ≥ EV mínimo`
   (2 % sáb/dom, 3 % lun–vie), con `p_justa` de Pinnacle + Betfair sin margen y consenso entre ambas.
3b. **El modelo propio no decide** mientras un backtest no muestre que le gana a Pinnacle (ver
   `04-modelo/resultados-backtest-2026-11.md`). Volver atrás = `fuente_probabilidad: modelo`.
4. **Nada de umbrales "a ojo".** Cambios en `config/estrategia.yaml` se justifican con un backtest
   y se anotan en `bitacora.md`.
5. **Modo prueba primero.** `modo_prueba: true` hasta tener 4–8 semanas de alertas con CLV positivo.
6. **Límites de riesgo siempre activos:** tope por apuesta (2%), exposición semanal (10%),
   pausa automática al perder 15% del fondo en el mes.
7. **Presupuesto de API:** OddsPapi gratis = 250 consultas/mes. El cliente bloquea al llegar a 230.
   **1 casa por consulta** (la lista dio 400); `betfair-ex` además **1 liga por consulta** y solo bajo demanda.
   Hoy: ~122/mes + pocas de Betfair. Los comandos de
   prueba (`probar-oddspapi`, `listar-casas`) gastan consultas que `uso_api` no cuenta.
8. **Secretos solo en `.env`** (nunca en GitHub): `DATABASE_URL`, `ODDSPAPI_API_KEY`, `API_TOKEN`. En el VPS, además, `/etc/apuestas/host.env` (600).
9. **La API nunca envía mensajes ni recibe acceso a Docker.** Solo el mensajero del VPS toca WhatsApp.

## Dónde está cada cosa

| Necesito… | Ir a |
|---|---|
| Entender el flujo completo | `01-arquitectura/arquitectura.md` |
| Cómo decide una alerta (embudo) | `04-modelo/modelo-y-embudo.md` |
| Toda técnica/herramienta usada, probada o descartada | `04-modelo/tecnicas-y-herramientas.md` |
| Resultados del backtest real | `04-modelo/resultados-backtest-2026-11.md` |
| Saber por qué se eligió algo | `02-decisiones/` |
| Ver las tablas | `03-datos/modelo-de-datos.md` y `apuestas-sistema/supabase/migrations/` |
| Cambiar umbrales | `apuestas-sistema/config/estrategia.yaml` (+ backtest + bitácora) |
| Agregar una liga | `apuestas-sistema/config/ligas.yaml` + `05-operacion/runbook.md` |
| Conectar un servicio | `06-guias/conexiones.md` |
| Comandos (PC, VPS, OpenClaw, Supabase) | `05-operacion/comandos.md` |
| Estado actual del despliegue | `bitacora.md` (entrada más reciente) |
| Lo que falta decidir | `preguntas-abiertas.md` |

## Cómo trabajar

- Antes de cambiar código: correr `pytest` en `apuestas-sistema` (71 pruebas).
- Si algo de conexión falla: `apuestas verificar` antes de tocar código.
- Después de un cambio importante: anotar en `bitacora.md` (fecha, qué, por qué).
- Si una decisión cambia la arquitectura: crear un nuevo ADR en `02-decisiones/`.
