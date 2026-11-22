# Visión

## Problema

Elegir apuestas "por intuición" o por el favorito no da ganancia a largo plazo: la casa ya pone
la cuota del favorito baja y se queda con un margen (≈5–8%). Para ganar hay que encontrar
**cuotas mal puestas**, y eso requiere comparar muchos partidos con un criterio fijo.

## Qué hace el sistema

1. Junta historial de partidos y cuotas (football-data.co.uk) y cuotas actuales (OddsPapi:
   Apuesta Total y Pinnacle).
2. Estima la probabilidad de cada resultado con un modelo propio (Dixon-Coles + Elo).
3. Pasa cada partido por un **embudo de 5 pasos** que descarta lo que no tiene valor o es riesgoso.
4. Avisa por WhatsApp solo los partidos que pasan, con el monto sugerido.
5. Registra todo (también los descartes) para medir si el sistema realmente funciona.

## Qué NO hace

- No apuesta solo ni se conecta a la cuenta de Apuesta Total.
- No promete ganar: es una herramienta para decidir con datos.
- No apuesta a empates, hándicaps ni goles (por ahora solo ganador local o visita).

## Ritmo

- Apuestas normales: **sábado y domingo**.
- Entre semana: solo si aparece algo con ventaja **más alta** que la normal (8% vs 5%).

## Criterios de éxito (en este orden)

1. **CLV positivo** durante 4–8 semanas en modo prueba: las cuotas que el sistema elige son mejores
   que la cuota de cierre de Pinnacle. Es la señal más rápida y confiable de ventaja real.
2. **Backtest de validación** con ROI por encima del margen de azar.
3. Recién entonces: dinero real con montos pequeños y los límites activos.

Si después de 8 semanas el CLV es negativo, el modelo no tiene ventaja: se ajusta o se detiene.

## Fases

| Fase | Contenido | Estado |
|---|---|---|
| 1 | Sistema base: datos, modelo, embudo, backtest, API, programador, mensajero, WhatsApp | v1.2 lista; tablas creadas en Supabase |
| 2 | Despliegue en VPS + Supabase + modo prueba | Pendiente |
| 3 | Ajuste de umbrales con datos reales, más ligas | Pendiente |
| 4 | App Android (panel de rendimiento + notificaciones) | Idea |
