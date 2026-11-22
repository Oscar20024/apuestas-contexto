# Bitácora

Formato: fecha · qué cambió · por qué · evidencia (backtest, CLV, etc.).

## 2026-11-22 · v1.2.0 — adaptado al VPS real (sin n8n) y tablas creadas en Supabase

**Por qué:** n8n no está instalado, y OpenClaw corre en Docker y se maneja con `docker exec`
(según el inventario del VPS y el manual técnico v1.1 de memory-api). Gemini falla seguido (503, 429, 404).

**Supabase**
- Proyecto `BDapuestas` (us-west-2): aplicadas las migraciones 0001 y 0002 (13 tablas, RLS en todas).
  Asesor de seguridad: solo el aviso informativo "RLS sin políticas", que es lo esperado.

**Sistema**
- Bandeja de salida `mensajes_salida` (migración 0002): la API deja los mensajes, nunca los envía.
  Las alertas caducan al empezar el partido; hasta 5 reintentos.
- `POST /tareas/{lunes|entre_semana|viernes|fin_de_semana|prueba}`, `GET /mensajes/pendientes`,
  `POST /mensajes/{id}/resultado`. Comando `apuestas tarea <nombre>`.
- `vps/apuestas_host.py` (solo biblioteca estándar): programador y mensajero. Envía con
  `docker exec openclaw-wr93-openclaw-1 openclaw message send` sin shell. Si la API cae, avisa directo.
- Timers de systemd con hora de Lima (`vps/systemd/`) e instalador `vps/instalar.sh`.
- Docker endurecido para el VPS: solo `127.0.0.1`, UID 10001, solo lectura, sin capacidades, 768 MB / 0.5 CPU.
- n8n pasa a ser opcional (flujos que solo llaman a `/tareas/...`). Se quitan los flujos 98 y 99.
- 58 pruebas (antes 51).

**Probado:** de punta a punta con la API real, Postgres y un `docker` simulado: WhatsApp caído →
reintento → enviado; API caída → aviso directo. Expresiones de calendario validadas con `systemd-analyze`.

**No probado aún:** `docker exec ... message send` en el VPS real, timers en el VPS real, conexión a Supabase con tu contraseña.

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
