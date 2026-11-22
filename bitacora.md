# Bitácora

Formato: fecha · qué cambió · por qué · evidencia (backtest, CLV, etc.).

## 2026-11-22 · v1.1.0 — listo para conectar los servicios

**Por qué:** la v1.0 tenía toda la lógica, pero conectar cada servicio dependía de editar a mano y
no había forma de saber qué fallaba.

**Sistema**
- `apuestas configurar`: asistente que arma el `.env`, prueba la base de datos al momento y genera el `API_TOKEN`.
- `apuestas verificar [--con-oddspapi]` y `GET /diagnostico`: revisan .env, Supabase (conexión, esquema,
  tablas, migraciones, datos, espacio), football-data y OddsPapi (key + IDs de ligas), con el arreglo de cada error.
- `DB_SCHEMA`: el sistema puede vivir en un esquema propio dentro de un proyecto de Supabase existente
  (la cuenta gratis ya tiene 2 proyectos activos). Ver ADR 0008.
- `apuestas migrar` lleva registro en `migraciones_aplicadas` y solo aplica lo que falte.
- Errores de conexión con mensaje claro en vez de traza de Python.
- n8n: flujo **98 - Enviar mensaje** (modos `comando`, `webhook_openclaw`, `webhook`) y flujo
  **00 - Prueba de conexiones**. Todos los flujos envían a través del 98. Ver ADR 0009.
- 51 pruebas (antes 35), incluidas migraciones en esquema propio y el asistente.

**Probado:** migraciones en un Postgres con una tabla `public.ligas` ajena (no se tocó); texto con
comillas, tildes y apóstrofes llega intacto al comando de OpenClaw; código JavaScript de los flujos compila.

**No probado aún:** importación real de los flujos en tu n8n, OpenClaw real, Supabase real.

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
