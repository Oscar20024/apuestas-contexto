# Visión

## Problema

Elegir apuestas "por intuición" o por el favorito no da ganancia a largo plazo: la casa pone cuotas
con un margen a su favor (Apuesta Total ≈ 6 %). Para ganar hay que encontrar **cuotas mal puestas**,
y eso requiere comparar muchos partidos con un criterio fijo.

## Qué hace el sistema (v1.4.0)

1. Trae las cuotas actuales de Apuesta Total, Pinnacle y Betfair Exchange (OddsPapi).
2. Calcula el **precio justo** de cada resultado: lo que dicen Pinnacle y Betfair, sin su margen.
   Son los mercados que mejor predicen el fútbol (el backtest lo confirmó: ver ADR 0011).
3. Pasa cada partido por un **embudo** que descarta lo que no tiene valor o es dudoso
   (cuota vieja, referencias que no coinciden, mercado en contra…).
4. Avisa por WhatsApp solo cuando **Apuesta Total paga más que el precio justo**, con el monto sugerido.
5. Registra todo (también los descartes) para medir si de verdad funciona.
6. Mantiene un modelo propio (Dixon-Coles + Elo) como dato informativo en la alerta.

## Qué NO hace

- No apuesta solo ni se conecta a la cuenta de Apuesta Total.
- No promete ganar: es una herramienta para decidir con datos.
- No mira hándicaps ni goles: solo 1X2 (local, empate o visita).

## Ritmo

- Apuestas normales: **sábado y domingo** (ventaja mínima 2 %).
- Entre semana: ventaja mínima más alta (3 %).
- Escaneos: lun–dom según el [calendario](../05-operacion/calendario-y-consultas.md).
- Se esperan **pocas alertas**: Apuesta Total suele pagar menos que Pinnacle.

## Criterios de éxito (en este orden)

1. **Oportunidades:** que Apuesta Total pague más que el precio justo con cierta frecuencia
   (se puede medir en 1–2 semanas con los datos guardados).
2. **CLV positivo** durante el modo prueba: las cuotas de las alertas mejores que el cierre de Pinnacle.
3. Recién entonces: dinero real con montos pequeños y los límites activos.

Si después del modo prueba no hay oportunidades o el CLV es negativo, la estrategia no sirve con Apuesta Total.

## Fases

| Fase | Contenido | Estado |
|---|---|---|
| 1 | Sistema base: datos, modelo, embudo, backtest, API, programador, mensajero, WhatsApp | ✅ |
| 2 | Despliegue en VPS + Supabase | ✅ 2026-11-22 |
| 2b | Backtest real y cambio a "precio vs Pinnacle" + refuerzos (v1.3–v1.4) | ✅ 2026-11-22 |
| 3 | Modo prueba: medir oportunidades y CLV en vivo | En curso desde 2026-11-23 |
| 4 | Ajuste de umbrales con datos reales, más ligas | Pendiente |
| 5 | App Android (panel de rendimiento + notificaciones) | Idea |
