# Bitácora

Formato: fecha · qué cambió · por qué · evidencia (backtest, CLV, etc.).

## 2026-11-23 · v1.4.2 — la tarea espera a que la API arranque

- v1.4.1 desplegada (imagen `apuestas-api:1.4.1`). Al lanzar `apuestas-tarea@previa` justo después de
  `docker compose up --force-recreate` (18:34 Lima), falló con `ConnectionResetError: [Errno 104] Connection
  reset by peer`: la API todavía estaba arrancando. No es un error del escaneo.
- **Corrección:** `vps/apuestas_host.py` ahora llama a `/salud` antes de la tarea y, si la conexión falla,
  reintenta hasta 6 veces cada 10 s (1 minuto). Solo reintenta errores de conexión; nunca repite la tarea.
  Evita el mismo problema si un timer coincide con un despliegue.
- El script del host se lee directo del repo: basta `git pull` (no hace falta reconstruir la imagen para esto).
- 71 pruebas.
- 18:39 Lima: `previa` relanzada a mano con v1.4.1 → **OK** (`partidos_vistos: 0`, `avisos: []`,
  `consultas_referencia_extra: 0`). Cero partidos porque en las 8 h siguientes no había partidos de liga.
  Primera ejecución completa sin errores de la estrategia de mercado.

## 2026-11-23 · v1.4.1 — falla de la primera `previa` y corrección

**Qué pasó:** a las 12:30 la tarea `previa` falló y llegó el aviso por WhatsApp:
`400 Bad Request` en `odds-by-tournaments?tournamentIds=17,8,23,35,34&bookmakers=betfair-ex`.
Apuesta Total y Pinnacle con las 5 ligas sí funcionaron; **Betfair Exchange no acepta varias ligas en una
consulta** (con 1 liga, `probar-oddspapi --torneo 17`, sí funcionó). Como la falla cortó el escaneo,
no se evaluó ningún partido. Gastó 3 consultas.

**Corrección (v1.4.1):**
- Betfair se pide **solo para las ligas donde Pinnacle ya ve una alerta posible**, de a **1 liga por consulta**.
  Casi nunca hay alerta posible, así que casi nunca se gasta una consulta en Betfair.
- Si Betfair (o cualquier referencia extra) falla, el escaneo **sigue** con Pinnacle sola y lo anota en
  `resumen.avisos`.
- El cliente se niega a pedir `betfair-ex` con más de 1 liga (no gasta la consulta en un 400 seguro).
- Presupuesto: ~122 consultas/mes (Apuesta Total + Pinnacle) + unas pocas de Betfair.
- Nueva prueba de punta a punta en modo mercado con Betfair (70 pruebas).

## 2026-11-22 · Bundesliga y Ligue 1 activadas

- `config/ligas.yaml`: D1 (OddsPapi 35) y F1 (OddsPapi 34) con `activa: true`. Los IDs son los de Sportradar,
  igual que 17/8/23 (ya verificados); se confirman con `verificar --con-oddspapi` (1 consulta).
- **Sin consultas extra:** una consulta cubre hasta 5 ligas por casa. Ahora son 5 ligas activas: es el máximo
  sin duplicar consultas.
- ~48 partidos por jornada en vez de ~30: más oportunidades y datos del modo prueba más rápido.
- Hecho en el VPS (23:00 Lima): `historial` y `entrenar` de D1 y F1. `verificar --con-oddspapi`: **24 ok, 0 avisos,
  0 errores**. IDs confirmados: 35 = Germany - Bundesliga, 34 = France - Ligue 1. Datos: D1 2223 y F1 2427 partidos
  con modelo activo. Base: 27 MB de 500.
- Sin backtest de D1/F1: la estrategia de mercado no se puede backtestear bien (no hay historial de Apuesta Total).

## 2026-11-22 · Documentación al día con v1.4.0

Revisión completa del repo de contexto contra lo que corre en el VPS:
- Nuevo `04-modelo/tecnicas-y-herramientas.md`: catálogo de cada técnica y herramienta (en uso, probada,
  investigada o descartada), con dónde está en el código, parámetros y resultado.
- `04-modelo/modelo-y-embudo.md` reescrito: embudo actual (10 motivos), ejemplo con cuotas reales del 27-11,
  formato de la alerta y limitaciones del backtest.
- Nuevo ADR 0012 (refuerzos del descarte). ADR 0003, 0005 y 0007 marcados/actualizados.
- `vision.md`, `arquitectura.md` (diagramas), `modelo-de-datos.md` (motivos, columnas, `detalles`, consulta de
  oportunidades), `runbook.md` (400, `sin_consenso`, `referencia_vieja`, cambiar referencia, volver al modelo),
  `conexiones.md`, `instalacion-paso-a-paso.md`, `preguntas-abiertas.md`, `plan-de-mejoras.md`, `AGENTS.md`, `README.md`.

Cambios de código menores (mismo despliegue):
- `evaluaciones.p_local/p_empate/p_visita` ahora guardan la probabilidad que **decidió** (precio justo en modo
  mercado); nuestro modelo queda en `detalles`. `detalles.p_justa` nuevo.
- La alerta dice "Prob. justa (Pinnacle + Betfair Exchange)" en vez de una línea aparte de confirmación.

## 2026-11-22 · Referencia extra: Betfair Exchange

Prueba con `probar-oddspapi --torneo 17` (Premier), una casa por comando:

| Casa | Partidos con 1X2 | Tiempo sin cambios | Margen aprox. | Veredicto |
|---|---|---|---|---|
| `betfair-ex` | 19 de 21 | 0–0,5 h | ~1 % | ✅ elegida |
| `sbobet` | 11 de 21 | 5–37 h | ~10 % | ❌ cuotas viejas y caras |
| `singbet` | 0 | — | — | ❌ sin cuotas en nuestro plan |

`estrategia.yaml` → `oddspapi.casas_referencia_extra: [betfair-ex]`. Precio justo = promedio Pinnacle + Betfair
sin margen; las dos deben ver la ventaja. Si Betfair no tiene el partido, decide solo Pinnacle.
Costo: 3 consultas por escaneo (~183 al mes de 250).

## 2026-11-22 · v1.4.0 — refuerzos del descarte

- **Cuotas recientes:** se lee `changedAt` de cada precio. Si Pinnacle no cambia hace >12 h → `referencia_vieja`.
  La alerta muestra hace cuánto no cambia la cuota de Apuesta Total.
- **Precio justo más sólido:** `oddspapi.casas_referencia_extra` (vacío por ahora). Con otra casa sharp,
  el precio justo es el promedio y todas deben ver la ventaja (`sin_consenso`). Falta elegir la casa con
  `apuestas listar-casas` (1 consulta).
- **Revisión cerca del partido:** tarea `previa` a las 12:30 todos los días menos viernes (8 h). Nuevo timer
  `apuestas-previa.timer` → hay que volver a correr `sudo bash vps/instalar.sh`.
- **Varias casas por consulta: NO funciona con nuestra clave.** La documentación dice hasta 3, pero
  `probar-oddspapi --casa apuestatotal,pinnacle` dio **400 Bad Request**. Corregido (v1.4.0 final):
  `oddspapi.casas_por_consulta: 1` y, si se sube y la API da 400, se piden de a una. ~122 consultas/mes de 250
  (~183 con una referencia extra).
- `listar-casas` funcionó: OddsPapi lista ~300 casas (guardado en `datos/oddspapi_casas.json` del VPS).
  Candidatas a referencia extra: `sbobet`, `singbet`, `betfair-ex`, `matchbook` (falta probar cuál trae 1X2 de E0/SP1/I1).
- `probar-oddspapi --casa a,b,c` muestra varias casas y la antigüedad de cada cuota.
- 69 pruebas (antes 66).
- Desplegado en el VPS (22:41 Lima, commit `93c685d`): 6 timers activos; primera `previa` el lunes 2026-11-23 12:30.

## 2026-11-22 · v1.3.0 — backtest real y cambio de estrategia (ADR 0011)

- CSV de football-data copiados del VPS a la PC (`tar` en el VPS + `scp mivps:...`; root no entra por llave).
- Fase 1 programada: pi-ratings (`modelos/pi_ratings.py`), tiros a puerta en el normalizador,
  combinador/stacking (`modelos/combinador.py`) y `apuestas comparar` (`backtest/comparar.py`).
- **Resultado:** el sistema original perdía (ROI −5 % a −13 %, CLV −6 % en todas las temporadas) y ninguna
  mejora lo arregla. Pinnacle sin margen predice mejor. Detalle en `04-modelo/resultados-backtest-2026-11.md`.
- **Decisión de Oscar:** cambiar a "precio vs Pinnacle". `estrategia.yaml`: `fuente_probabilidad: mercado`,
  margen `potencia`, selecciones H/D/A, EV mínimo 2 % (finde) y 3 % (entre semana), cuota 1,50–4,00,
  diferencia máxima AT vs Pinnacle 10 pts. El modelo queda como dato informativo en la alerta.
- Se esperan pocas alertas; el modo prueba medirá si Apuesta Total paga más que Pinnacle y con qué CLV.
- 66 pruebas (antes 59).
- Desplegado en el VPS (22:13 Lima): imagen `apuestas-api:1.3.0`, `verificar` 16 ok / 0 avisos / 0 errores.

## 2026-11-22 · Investigación: cómo mejorar el modelo

- Nuevo `04-modelo/plan-de-mejoras.md`: 3 fases (sin datos nuevos → xG y bajas → descartes por escenario).
- Primer paso acordado en el plan: backtest base de E0, SP1, I1 antes de tocar el modelo.

## 2026-11-22 · Tarea `simulacro`: una alerta inventada por WhatsApp

- Nueva tarea `simulacro` (`apuestas-tarea@simulacro.service` o `apuestas tarea simulacro`): encola UNA alerta
  con la plantilla real, marcada `[SIMULACRO ...]` y `[MODO PRUEBA ...]`. Partido y números inventados.
- Se guarda en `mensajes_salida` como tipo `prueba`, caduca en 6 h y **no** crea evaluaciones: no ensucia las estadísticas ni el CLV.
- 59 pruebas (antes 58).
- Probado en el VPS (21:41 Lima): `git pull`, rebuild y `apuestas-tarea@simulacro.service` → la alerta llegó al WhatsApp con el formato correcto.

## 2026-11-22 · Estado al cierre del día — todas las conexiones verificadas

| Pieza | Estado |
|---|---|
| Repos en GitHub (`Oscar20024`) | ✅ v1.2.0 en ambos |
| Código en el VPS (`/docker/apuestas-sistema`) | ✅ clonado con deploy key de solo lectura; `git pull` funciona |
| API en Docker | ✅ `Up (healthy)` en `127.0.0.1:8010` |
| OddsPapi | ✅ key válida; IDs verificados: 17 = Premier, 8 = LaLiga, 23 = Serie A |
| football-data | ✅ descarga desde el VPS |
| Envío por WhatsApp (OpenClaw) | ✅ `openclaw message send` entrega (tras agregar `+51` a `allowFrom`) |
| Supabase (`BDapuestas`) | ✅ conectado (Postgres 17.12, 12 tablas, migraciones 0001 y 0002). `verificar`: 13 ok, 0 errores |
| Datos (`historial`, `entrenar`) | ⏳ después de Supabase |
| Programador y mensajero (`vps/instalar.sh`) | ⏳ después de los datos |

**Siguiente paso:** dejar correr el modo prueba. Primera tarea programada: lunes 2026-11-23 07:00 (Lima).

### Instalación completada (2026-11-22, noche)
- `historial` y `entrenar` hechos; `verificar`: **16 ok, 0 avisos, 0 errores**.
- `sudo bash vps/instalar.sh`: `/etc/apuestas/host.env` creado, permisos de `datos/`, unidades en `/etc/systemd/system`, 5 timers activos.
- Horarios confirmados (systemd los muestra en UTC; Lima = UTC−5):
  lunes 12:00 UTC = 07:00 Lima · martes a jueves 14:00 UTC = 09:00 · viernes 17:00 UTC = 12:00 · sábado y domingo 11:00 UTC = 06:00 (y 10:00) · mensajero cada 5 min.
- `probar-whatsapp` y `apuestas-tarea@prueba.service`: **los dos mensajes llegaron al WhatsApp** (20:02 Lima).
  Cadena completa probada: systemd → API → Supabase → bandeja → mensajero → OpenClaw → WhatsApp.
  Datos cargados: E0 2760, SP1 2779, I1 2770 partidos, con modelo activo en las 3 ligas; 22 MB usados.
- `probar-oddspapi --torneo 17`: **Apuesta Total y Pinnacle devuelven cuotas 1X2** (21 partidos de Premier).
  Margen aproximado: Apuesta Total ~6 %, Pinnacle ~3 %. Los nombres de equipos salen `None` en esa prueba
  (normal: el escaneo los busca con `/participants`). Estas 2 consultas no quedaron en `uso_api` (comando sin base).

Cómo se resolvió Supabase: la URL tenía la API key `sb_secret_...` en lugar de la contraseña de la base.
Se reseteó la contraseña (Database → Settings) y, tras editar el `.env`, hubo que **recrear el contenedor**
(`docker compose up -d --force-recreate`): Docker solo lee el `.env` al crear el contenedor.

Lecciones del día:
- Las credenciales se habían escrito en `.env.example` (que se sube a GitHub). Se movieron a `.env` antes de cualquier commit.
- En el VPS se pegaron comandos dentro de `.env`. El `.env` lleva solo 4 líneas `CLAVE=valor`; los comandos van en la terminal.
- Comandos de referencia: `05-operacion/comandos.md`.

## 2026-11-22 · Prueba de envío en el VPS real

- `docker exec openclaw-wr93-openclaw-1 openclaw message send --channel whatsapp` falló primero:
  `allowFrom` tenía el número sin `+51`. Se agregó en formato internacional sin quitar el existente.
  Después: "Sent via gateway (whatsapp)". OpenClaw no se reinició (el envío funcionó sin reiniciar).
- Credenciales movidas de `.env.example` a `.env` en la PC antes de cualquier commit (no se filtró nada).

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
