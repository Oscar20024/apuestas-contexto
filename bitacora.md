# Bitácora

Formato: fecha · qué cambió · por qué · evidencia (backtest, CLV, etc.).

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
