# 0012 · Refuerzos del descarte: frescura, consenso y revisión cerca del partido

**Estado:** Aceptada · **Fecha:** 2026-11-22 · Complementa [0011](0011-probabilidad-desde-pinnacle.md)

## Contexto
Con la probabilidad tomada del mercado (ADR 0011), los errores más probables ya no son del modelo sino
de los **datos de cuotas**: una cuota de Pinnacle vieja, una sola referencia que se equivoca, o una
cuota de Apuesta Total que cambia antes de que llegues a apostar. Además, entre semana solo había un
escaneo a las 09:00, lejos de los partidos de la tarde.

## Decisión
1. **Frescura (`referencia_vieja`):** se lee `changedAt` de cada precio. Si la cuota de Pinnacle no cambia
   hace más de 12 h, se descarta. La antigüedad de la cuota de Apuesta Total se muestra en la alerta (no se
   descarta: que Apuesta Total no se actualice es justamente la oportunidad).
2. **Consenso (`sin_consenso`):** segunda referencia `betfair-ex`. Precio justo = promedio sin margen de
   Pinnacle y Betfair; si las dos tienen el partido, **las dos** deben ver la ventaja mínima. Si Betfair no
   lo tiene, decide Pinnacle sola.
3. **Revisión `previa`:** escaneo de 8 h a las 12:30 (Lima) todos los días menos viernes (el viernes ya
   está el de las 12:00). Timer `apuestas-previa.timer`.
4. **Una casa por consulta** a OddsPapi (la lista dio 400). Si se configura `casas_por_consulta > 1` y la API
   responde 400, el sistema pide de a una sin fallar.

## Consecuencias
- ~61 escaneos/mes × 3 casas ≈ **183 consultas de 250** (reserva de 20). No alcanza para una cuarta casa
  ni para más de 5 ligas sin quitar escaneos.
- Menos alertas falsas por datos viejos o por una sola casa equivocada; algunas alertas buenas también
  se pierden cuando Betfair no coincide.
- Estos filtros no se pueden backtestear (no hay historial de `changedAt` ni de Betfair): se evalúan en vivo
  mirando los motivos de descarte en `evaluaciones`.

## Alternativas descartadas
- **sbobet / singbet como referencia:** cuotas viejas y caras, o sin datos.
- **Copias de Pinnacle (ps3838, pin88, pinnacle+5):** no son una opinión independiente.
- **Escanear cada hora:** no alcanzan las 250 consultas del plan gratis.
