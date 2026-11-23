# Plan de mejoras del modelo (investigación 2026-11-22)

Objetivo: que las alertas y los descartes sean más finos y se adapten a distintos escenarios,
**sin** subir la complejidad a ciegas. Cada mejora entra solo si mejora el backtest
(log-loss/RPS, calibración, ROI) y después el CLV real en modo prueba.

## Regla de oro

1. Primero medir la base actual (backtest de E0, SP1, I1). Sin eso no se sabe si algo "mejora".
2. Una mejora a la vez, con la misma partición calibración/validación.
3. Elegir modelos por **calibración** (log-loss, RPS), no por % de aciertos.
   Walsh & Joshi (2024): seleccionar por calibración dio mucho más retorno que por accuracy.
4. Desconfiar de mejoras que solo aparecen tras probar muchas combinaciones de umbrales (sobreajuste).

## Fase 1 — sin datos nuevos (lo más barato)

| # | Mejora | Por qué | Esfuerzo |
|---|---|---|---|
| 1.1 | **Pi-ratings** como tercer modelo/feature (librería `penaltyblog`) | Rating específico de fútbol: usa diferencia de goles, separa local/visita y pesa más lo reciente. En los papers supera a Elo | Bajo |
| 1.2 | **Calibración del ensamble** (Platt / isotónica) y pesos elegidos por log-loss | Hoy el 60/40 es fijo. Un valor de "+8 %" solo es real si las probabilidades están bien calibradas | Bajo |
| 1.3 | **Mezcla modelo + mercado** (Pinnacle sin margen) con regresión logística | El mercado sharp ya resume casi toda la información. Lo que importa es dónde Apuesta Total se queda atrás de Pinnacle. Los CSV de football-data traen `PSH/PSCH` para entrenarlo | Medio |
| 1.4 | **Tiros y tiros a puerta** (`HS, AS, HST, AST` del mismo CSV) como ratings de ataque | Proxy gratis de xG; los tiros a puerta tienen valor predictivo (Wheatcroft) | Bajo |
| 1.5 | **Meta-modelo de árboles** (CatBoost/LightGBM) sobre los ratings anteriores | Enfoque ganador del 2017 Soccer Prediction Challenge (pi-ratings + gradient boosting). Cabe en el VPS | Medio |
| 1.6 | Ajustar vida media y ventana **por liga** en el backtest | Cada liga cambia a distinto ritmo | Bajo |

## Fase 2 — datos nuevos gratuitos

| # | Mejora | Fuente | Notas |
|---|---|---|---|
| 2.1 | **xG por partido** → Dixon-Coles con xG o ratings de xG | Understat (EPL, La Liga, Serie A desde 2014/15) vía `soccerdata` | FBref ya no sirve para xG: perdió los datos avanzados de Opta el 20-01-2026 |
| 2.2 | **Bajas y alineaciones** antes del partido | API-Football, plan gratis 100 consultas/día | Útil como filtro de última hora (si falta una figura, descartar o recalcular) |

## Fase 3 — escenarios (descartes especializados)

Analizar el backtest **por segmentos** y apagar los segmentos donde el modelo pierde:

- Inicio de temporada (primeras ~5 jornadas) y equipos recién ascendidos (pocos datos).
- Final de temporada: partidos sin nada en juego vs. lucha por descenso/Europa.
- Semanas con copas europeas (rotaciones) y días de descanso desiguales.
- Tipo de selección: local / empate / visita y rango de cuota. La literatura encuentra sesgo
  favorito-sorpresa en el mercado 1X2 de casas tradicionales: las apuestas a sorpresas pierden más.
- Derbis.

Cada segmento queda como regla en `embudo` (descarte o umbral de EV más alto), con su motivo visible.

## Qué NO hacer (por ahora)

- Redes neuronales / LLM que "opinen" sobre el partido: no mejoran calibración con estos datos.
- Agregar muchas variables sin validar.
- Skills de Claude de apuestas (p. ej. `machina-sports/sports-skills` → `betting`, `sports-betting-analyzer`):
  hacen de-vig, Kelly y edge, que el sistema ya tiene. No entrenan modelos.

## Riesgo a tener presente

Kaunitz et al. (2017) ganaron apostando donde una casa pagaba más que el consenso del mercado,
pero las casas les **limitaron las cuentas**. Si el sistema funciona, Apuesta Total puede hacer lo mismo.

## Fuentes

- penaltyblog (Poisson, Bivariate, Dixon-Coles, Elo, Pi ratings, de-vig): https://penaltyblog.readthedocs.io/
- Pi ratings en penaltyblog: https://penaltyblog.readthedocs.io/en/master/ratings/pi.html
- soccerdata (Understat, ClubElo, football-data, etc.): https://soccerdata.readthedocs.io/en/stable/
- Hubáček et al. (2019), ganadores 2017 Soccer Prediction Challenge: https://ida.fel.cvut.cz/papers/hubacek2019learning.html
- Yeung et al., CatBoost + pi-ratings: https://arxiv.org/abs/2309.14807v1
- Walsh & Joshi (2024), calibración vs accuracy: https://doaj.org/article/2e36e6f3ec6a46b7b54308dee2d11c23
- Wheatcroft, predecir estadísticas del partido: https://arxiv.org/pdf/2001.09097
- Kaunitz et al. (2017), BeatTheBookie: https://github.com/Lisandro79/BeatTheBookie
- Hegarty & Whelan, sesgo favorito-sorpresa en 1X2: https://cepr.org/voxeu/columns/forecasting-soccer-matches-betting-odds-tale-two-markets
- FBref pierde datos avanzados (enero 2026): https://www.thestatsapi.com/blog/best-football-statistics-websites
- API-Football, plan gratis: https://www.api-football.com/pricing
