# Bitácora

Formato: fecha · qué cambió · por qué · evidencia (backtest, CLV, etc.).

## 2026-11-22 · v1.0.0 — primera versión completa (sin desplegar)

**Sistema (`apuestas-sistema`)**
- Modelo: Dixon-Coles (ventana 3 años, vida media 1 año) + Elo con regresión logística, mezcla 60/40.
- Embudo de 5 pasos con motivos de descarte guardados.
- Límites: 3 alertas/día, 10% de exposición semanal, pausa al −15% mensual, tope 2% por apuesta.
- Backtest walk-forward con búsqueda de umbrales (calibración vs. validación) y comparación contra el cierre de Pinnacle.
- API FastAPI para n8n (token), Docker, esquema SQL para Supabase con RLS, 5 flujos de n8n.
- Modo prueba activado por defecto.
- 35 pruebas, incluida una de punta a punta con Postgres real (historial → entrenar → escanear → apuesta → liquidar → resumen).

**Probado con:** liga simulada (5 temporadas). Resultado esperado: el modelo queda un poco por debajo del
cierre de Pinnacle y el CLV sale negativo; confirma que el reporte no "inventa" ganancias.

**No probado aún:** datos reales de football-data (bloqueados en el entorno de desarrollo),
respuesta real de OddsPapi, envío por OpenClaw.

**Siguiente:** backtest real en tu PC (E0, SP1, I1) → Supabase → OddsPapi → VPS → n8n.
