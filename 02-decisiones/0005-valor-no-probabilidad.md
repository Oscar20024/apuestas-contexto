# 0005 · Decidir por valor esperado, no por probabilidad alta

**Estado:** Aceptada (umbrales actualizados por [0011](0011-probabilidad-desde-pinnacle.md)) · **Fecha:** 2026-11-22

## Contexto
La idea inicial era "predicciones con alta probabilidad". Pero un favorito al 80% paga ~1.20: si se pierde 1 de cada 5, se borra lo ganado en las otras 4.

## Decisión
Un partido es pick solo si `EV = p × cuota − 1` supera el mínimo, después de quitar el margen de la casa para comparar.

- Original: `p` = nuestro modelo; mínimo 5 % fin de semana, 8 % entre semana; solo local o visita.
- **Desde v1.3.0 (ADR 0011):** `p` = precio justo de Pinnacle (y Betfair desde v1.4.0) sin margen;
  mínimo **2 %** fin de semana y **3 %** entre semana; local, empate o visita. Los umbrales bajan porque
  ahora `p` es mucho más confiable: una ventaja de 2 % sobre el mercado sharp es real, mientras que un
  "+14 %" del modelo resultó ser error del modelo.

## Consecuencias
- Habrá semanas con 0 alertas, y está bien.
- La tasa de acierto será más baja que "apostar al favorito" (≈40% en cuotas de 2.0–3.0). Lo que importa es el ROI y el CLV, no el % de aciertos.
