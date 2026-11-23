# 0007 · Fuentes: football-data.co.uk + OddsPapi

**Estado:** Aceptada · **Fecha:** 2026-11-22

## Decisión
- **Historial**: football-data.co.uk (CSV gratis, resultados + cuotas previas y de cierre de Bet365 y Pinnacle).
- **Cuotas actuales**: OddsPapi (incluye Apuesta Total y Pinnacle; plan gratis de 250 consultas/mes).

## Actualización 2026-11-22 (v1.3–v1.4)
- **OddsPapi:** también se usa `betfair-ex` (Betfair Exchange) como segunda referencia. Cada precio trae
  `changedAt` (último cambio) y `bookmakerChangedAt`. La documentación dice que `odds-by-tournaments`
  acepta hasta 3 casas por consulta, pero con nuestra clave `apuestatotal,pinnacle` dio **400**: se pide
  **1 casa por consulta**. `/bookmakers` lista ~300 casas (1 consulta).
- **Casas probadas como referencia:** `betfair-ex` ✅ (19/21 partidos, < 1 h, margen ~1 %);
  `sbobet` ❌ (11/21, hasta 37 h sin cambios, margen ~10 %); `singbet` ❌ (sin cuotas).
- **football-data:** ahora se leen también los tiros (`HS`, `AS`, `HST`, `AST`). La cuota previa de
  Pinnacle (`PSH/PSD/PSA`) dejó de venir a mitad de 2025/26 (55 % en E0, 0 % en 2026/27).
- **xG:** FBref perdió sus datos avanzados (Opta) el 20-01-2026. Understat sigue gratis para las 5 grandes
  ligas (vía `soccerdata`), no se usa por ahora.

## Limitaciones conocidas
- **Liga 1 de Perú no está en football-data.** Para agregarla hace falta otra fuente de historial (pendiente).
- Apuesta Total no tiene historial público: en el backtest se usa **Bet365 como sustituto** de "casa blanda".
- Las ligas "extra" (Argentina, Brasil, México…) solo traen cuotas de cierre: su backtest simula apostar al cierre.
- Algunos detalles del formato de OddsPapi no están 100% documentados (ver `05-operacion/runbook.md`, "Primera conexión con OddsPapi").
