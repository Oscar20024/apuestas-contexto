# Resultados del backtest — 2026-11-22

Datos: CSV de football-data guardados en el VPS (8 temporadas, 2019/20 a 2026/27) de E0, SP1 e I1.
Comando: `apuestas comparar --liga <L> --csv datos/football_data/<L>_*.csv`.
Walk-forward: se reentrena cada semana con lo anterior; los combinadores se entrenan solo con
temporadas previas. Se evalúan ~1.620 partidos por liga (2022/23 en adelante).
Casa "blanda" del backtest: Bet365 previa (Apuesta Total no tiene historial público).
CLV medido contra el cierre de Pinnacle.

## 1. El sistema actual (mezcla fija 0.6 Dixon-Coles + 0.4 Elo) pierde

| Liga | Picks | ROI | CLV medio | Picks con CLV > 0 | Ventaja que el modelo CREÍA tener |
|---|---|---|---|---|---|
| Premier (E0) | 402 | −12,8 % | −5,8 % | 20 % | ~+14 % |
| LaLiga (SP1) | 362 | −12,6 % | −6,5 % | 19 % | ~+14 % |
| Serie A (I1) | 373 | −4,7 % | −6,3 % | 15 % | ~+14 % |

Pasa en **todas** las temporadas (CLV entre −3,5 % y −7,3 % cada una). No es mala suerte:
las "ventajas" grandes son errores del modelo, no del mercado.

## 2. Las mejoras de la fase 1 no lo arreglan

Log-loss (menor es mejor). Pinnacle previa sin margen, en los partidos donde hay dato:

| | E0 | SP1 | I1 |
|---|---|---|---|
| Mercado (Pinnacle previa) | **0,951** | **0,966** | **0,973** |
| Nuestro modelo (fija) | 0,968 | 0,983 | 0,989 |
| Mezcla 90 % mercado + 10 % modelo | 0,952 | 0,967 | 0,973 |

- Pi-ratings, tiros a puerta y combinadores (stacking) quedan en el rango 0,985–0,992: **no mejoran**.
- Cualquier peso que se le dé a nuestro modelo sobre Pinnacle **empeora** la predicción.
- Conclusión: con solo goles y tiros, el modelo no aporta información que Pinnacle no tenga ya.

## 3. Estrategia "precio vs Pinnacle" (Kaunitz) con Bet365 como casa blanda

Probabilidad = Pinnacle previa sin margen; apostar si Bet365 paga por encima. Resultado:
pocas oportunidades (≈3–16 por temporada) y CLV ≈ 0. Bet365 es casi tan eficiente como Pinnacle.
Apuesta Total es otra casa (margen ~6 %, mercado local); **su comportamiento solo se puede medir en vivo**.

## Archivos

- Predicciones por componente: `datos/comparar/componentes_<L>.csv` (fuera del repo).
- Tablas: `datos/comparar/comparacion_<L>.csv`.
