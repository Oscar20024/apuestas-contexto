# 0005 · Decidir por valor esperado, no por probabilidad alta

**Estado:** Aceptada · **Fecha:** 2026-11-22

## Contexto
La idea inicial era "predicciones con alta probabilidad". Pero un favorito al 80% paga ~1.20: si se pierde 1 de cada 5, se borra lo ganado en las otras 4.

## Decisión
Un partido es pick solo si `EV = p_modelo × cuota − 1` supera el mínimo (5% fin de semana, 8% entre semana), después de quitar el margen de la casa para comparar.

## Consecuencias
- Habrá semanas con 0 alertas, y está bien.
- La tasa de acierto será más baja que "apostar al favorito" (≈40% en cuotas de 2.0–3.0). Lo que importa es el ROI y el CLV, no el % de aciertos.
