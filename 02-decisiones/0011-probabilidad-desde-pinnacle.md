# 0011 · La probabilidad "real" sale de Pinnacle, no de nuestro modelo

**Estado:** Aceptada · **Fecha:** 2026-11-22 · Ajusta [0003](0003-modelo-dixon-coles-elo.md) y [0005](0005-valor-no-probabilidad.md)

## Contexto
Primer backtest con datos reales (E0, SP1, I1; ver `04-modelo/resultados-backtest-2026-11.md`):
- El sistema original (0.6 Dixon-Coles + 0.4 Elo) habría perdido: ROI −5 % a −13 % y CLV −6 %
  en **todas** las temporadas. Las "ventajas" que veía eran errores del modelo.
- Pi-ratings, tiros a puerta y combinadores (stacking) no lo arreglan.
- Pinnacle sin margen predice mejor que cualquier versión del modelo, y darle peso al modelo
  sobre Pinnacle empeora la predicción.

## Decisión
1. `embudo.fuente_probabilidad: mercado`: la probabilidad justa = Pinnacle sin margen (método potencia).
2. Se alerta cuando **Apuesta Total paga más que ese precio justo** (EV ≥ 2 % fin de semana, 3 % entre semana),
   con local, empate y visita, cuota 1,50–4,00.
3. Si Apuesta Total y Pinnacle difieren más de 10 puntos, se descarta (cuota vieja o dato erróneo).
4. Nuestro modelo sigue entrenándose y aparece en la alerta como **dato informativo**.
5. Se puede volver al modelo con `fuente_probabilidad: modelo` (sin tocar código).

## Consecuencias
- Habrá **pocas alertas**. Con Bet365 como sustituto de Apuesta Total, el backtest dio 0–7 en 4,5 temporadas
  por liga: Bet365 es casi tan eficiente como Pinnacle. Apuesta Total puede comportarse distinto; solo se sabe en vivo.
- El modo prueba ahora mide eso: cuántas veces Apuesta Total paga más que Pinnacle y si el CLV es positivo.
- Si Apuesta Total limita la cuenta por ganar seguido (riesgo documentado por Kaunitz et al. 2017),
  la estrategia deja de servir.

## Alternativas descartadas
- **Seguir con el modelo:** el backtest muestra pérdida sistemática.
- **Correr ambos en paralelo:** duplica mensajes con una estrategia que ya sabemos que pierde.
