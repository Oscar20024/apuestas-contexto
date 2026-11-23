# 0003 · Modelo: mezcla Dixon-Coles + Elo

**Estado:** Ajustada por [0011](0011-probabilidad-desde-pinnacle.md) · **Fecha:** 2026-11-22

> Desde 2026-11-22 el modelo **no decide** las alertas: el backtest mostró que pierde frente a Pinnacle.
> Sigue entrenándose y aparece en la alerta como dato informativo. Pi-ratings y combinador se probaron y no mejoraron.

## Contexto
Hace falta una probabilidad para local, empate y visita que sea razonable, rápida de entrenar en el VPS y explicable.

## Decisión
- **Dixon-Coles**: ataque y defensa por equipo, ventaja de local, corrección de marcadores bajos, partidos viejos pesan menos (vida media 1 año), ventana de 3 años.
- **Elo + regresión logística**: fuerza relativa con todo el historial; la logística traduce la diferencia de Elo en las 3 probabilidades.
- Mezcla 60% / 40% (configurable).

## Alternativas descartadas por ahora
- Redes neuronales / XGBoost con muchas variables: más datos y más riesgo de sobreajuste; sin ventaja clara con solo resultados y cuotas.

## Consecuencias
- Entrena en segundos (un backtest de 5 temporadas tarda menos de un minuto).
- No conoce lesiones, rotaciones ni noticias. Por eso existe el filtro "mercado en contra" (Pinnacle sí las refleja).
