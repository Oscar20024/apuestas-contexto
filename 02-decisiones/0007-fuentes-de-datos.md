# 0007 · Fuentes: football-data.co.uk + OddsPapi

**Estado:** Aceptada · **Fecha:** 2026-11-22

## Decisión
- **Historial**: football-data.co.uk (CSV gratis, resultados + cuotas previas y de cierre de Bet365 y Pinnacle).
- **Cuotas actuales**: OddsPapi (incluye Apuesta Total y Pinnacle; plan gratis de 250 consultas/mes).

## Limitaciones conocidas
- **Liga 1 de Perú no está en football-data.** Para agregarla hace falta otra fuente de historial (pendiente).
- Apuesta Total no tiene historial público: en el backtest se usa **Bet365 como sustituto** de "casa blanda".
- Las ligas "extra" (Argentina, Brasil, México…) solo traen cuotas de cierre: su backtest simula apostar al cierre.
- Algunos detalles del formato de OddsPapi no están 100% documentados (ver `05-operacion/runbook.md`, "Primera conexión con OddsPapi").
