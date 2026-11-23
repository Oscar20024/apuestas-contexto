# Runbook: qué hacer cuando…

## Primer paso ante cualquier problema

```bash
docker compose exec apuestas-api python -m apuestas verificar     # en el VPS
apuestas verificar                                                # en tu PC
```
Revisa .env, Supabase, la bandeja de mensajes, football-data y el uso de OddsPapi, y dice cómo
arreglar cada error. `sudo systemctl start apuestas-tarea@prueba.service` hace lo mismo y te manda el
resultado por WhatsApp.

Registros del programador y el mensajero: `journalctl -u 'apuestas-*' -n 100 --no-pager`.

## …llega un mensaje "FALLO en Apuestas 0X"

1. En el VPS: `cd /docker/apuestas-sistema && docker compose logs --tail 100`
2. En Supabase: `select * from ejecuciones order by inicio desc limit 5;` (columna `detalle` tiene el error).
3. Errores típicos:

| Error | Causa | Solución |
|---|---|---|
| `429 Limite mensual de OddsPapi` | Se gastaron las consultas del mes | Esperar al día 1 o subir de plan. Revisar `uso_api` por si algo se repitió de más. |
| `401 Token invalido` | `API_TOKEN` de `/etc/apuestas/host.env` distinto al del `.env` | Igualarlos y `docker compose up -d` |
| `la API no responde` | El contenedor está caído | `docker compose ps` y `docker compose up -d` |
| `could not connect to server` / timeout | Supabase pausado o URL mal | Abrir el proyecto en Supabase (lo despierta); revisar `DATABASE_URL` |
| `Liga X sin modelo entrenado` | No corrió el entrenamiento | `docker compose exec apuestas-api python -m apuestas entrenar` |
| `Historial insuficiente` | Liga nueva sin historial | `... python -m apuestas historial --ligas X` y luego entrenar |
| `400 Bad Request` en `odds-by-tournaments` | Se pidieron varias casas juntas (`a,b`) o un parámetro inválido | Dejar `oddspapi.casas_por_consulta: 1`. El escaneo ya pide de a una si la lista da 400 |

## …no llegan mensajes a WhatsApp

1. `journalctl -u apuestas-mensajero -n 50 --no-pager`: dice qué mensaje falló y por qué.
2. Probar el envío directo: `python3 vps/apuestas_host.py probar-whatsapp`.
3. Si falla también: `docker logs --tail 50 openclaw-wr93-openclaw-1` (¿WhatsApp desconectado?).
   La sesión se reconecta sola cada ~2 h; los mensajes pendientes se reintentan cada 5 min, hasta 5 veces.
4. En Supabase: `select id, tipo, estado, intentos, ultimo_error from mensajes_salida order by id desc limit 10;`
   - `caducado`: era una alerta y el partido ya empezó (no sirve mandarla tarde).
   - `fallido`: 5 intentos sin éxito. Para reenviarlo: `update mensajes_salida set estado='pendiente', intentos=0 where id=...;`

## …OpenClaw responde "is not listed in the configured WhatsApp allowFrom policy"

OpenClaw solo envía WhatsApp a números de `channels.whatsapp.allowFrom`, y los compara en formato
internacional (E.164, con `+51`). Encontrado el 2026-11-22: la lista tenía el número sin código de país.

```bash
docker exec openclaw-wr93-openclaw-1 openclaw config get channels.whatsapp.allowFrom
docker exec openclaw-wr93-openclaw-1 openclaw config set channels.whatsapp.allowFrom '["XXXXXXXXX", "+51XXXXXXXXX"]' --strict-json
```
Se **agrega** el número en formato `+51` y se mantiene el que ya estaba, para no cambiar cómo
funciona hoy OpenClaw con sus otras funciones. El cambio se aplica en caliente.

Ojo: el arranque de la imagen de Hostinger reescribe la config de WhatsApp en cada reinicio, probablemente
a partir de `WHATSAPP_NUMBER` del `.env` de OpenClaw. Si después de un reinicio vuelve el error, revisar
`docker exec openclaw-wr93-openclaw-1 printenv WHATSAPP_NUMBER` y repetir el `config set`.

## …casi todo se descarta por `sin_consenso` o `referencia_vieja`

```sql
select motivo, count(*) from evaluaciones where evaluado_en > now() - interval '7 days' group by 1 order by 2 desc;
```
- **`referencia_vieja` en muchos partidos:** OddsPapi no está actualizando Pinnacle. Probar
  `probar-oddspapi --torneo 17 --casa pinnacle` y mirar las horas "sin cambios". Si es general y temporal,
  esperar; si dura, subir `antiguedad_maxima_referencia_horas` en `estrategia.yaml` (repo → VPS) y anotarlo.
- **`sin_consenso` en muchos partidos con ventaja:** Betfair ve distinto que Pinnacle. Es una protección;
  si quieres más alertas a cambio de más riesgo: `exigir_consenso_referencias: false` (decide el promedio).
- **Betfair no trae un partido:** decide Pinnacle sola (no es error).

## …quiero cambiar o quitar la casa de referencia extra

1. `listar-casas --buscar <texto>` para ver nombres (1 consulta).
2. `probar-oddspapi --torneo 17 --casa <nombre>` (1 consulta): mirar cuántos partidos trae, horas sin cambios y margen.
3. En `config/estrategia.yaml` → `oddspapi.casas_referencia_extra: [<nombre>]` (o `[]` para quitarla).
4. Repo → VPS: `git pull` + `docker compose up -d --build --force-recreate`. Anotar en `bitacora.md`.
5. Ojo con el presupuesto: cada casa = 1 consulta por escaneo (`calendario-y-consultas.md`).

## …quiero volver a decidir con nuestro modelo

`embudo.fuente_probabilidad: modelo` en `estrategia.yaml`. Antes, un backtest (`apuestas comparar`) que muestre
que el modelo le gana a Pinnacle previa en log-loss; si no, se repite la pérdida de ADR 0011.

## …el resumen dice "nombres de equipos por revisar"

OddsPapi llamó a un equipo de una forma que el sistema no reconoció. Mientras no se resuelva,
**ese partido no se evalúa**. Ver consultas en `03-datos/modelo-de-datos.md` → "Nombres por revisar".

## …llega "ALERTAS PAUSADAS"

Se perdió el 15% del fondo en el mes. Es intencional. Se reactiva solo el día 1.
Antes de forzarlo, revisar: ¿el CLV de las alertas era positivo? Si era negativo, el problema es el modelo, no la mala suerte.

## …quiero agregar una liga

1. `apuestas listar-torneos --buscar "<país>"` → anotar el ID.
2. En `config/ligas.yaml`: poner `oddspapi_torneo_id` y `activa: true`.
3. Correr el backtest de esa liga y anotar el resultado en `bitacora.md`.
4. En el VPS: `git pull`, `docker compose up -d --build`, luego `historial --ligas X` y `entrenar --ligas X`.
5. Revisar el presupuesto de consultas (`calendario-y-consultas.md`).

## …quiero cambiar un umbral

1. Cambiar en `config/estrategia.yaml` **en tu PC**.
2. Medir: en modo mercado, con los datos en vivo de `evaluaciones` (ver consultas en `03-datos/modelo-de-datos.md`);
   el backtest solo sirve como referencia porque usa Bet365 en vez de Apuesta Total.
   En modo modelo: `apuestas comparar --liga E0 --reusar` o `apuestas backtest --liga E0 --sin-busqueda`.
3. Subir a GitHub, anotar en `bitacora.md`, y en el VPS `git pull` + `docker compose up -d --build --force-recreate`
   (el `.env` y la configuración se leen al crear el contenedor).

## …hice una apuesta real

Registrarla para que el sistema calcule el fondo y el límite de pérdida (en el VPS, dentro de `/docker/apuestas-sistema`):

```bash
docker compose exec apuestas-api python -m apuestas registrar-apuesta --evaluacion <ID de la alerta> --monto 10 --cuota 2.08
```

(`--cuota` solo si la cuota que tomaste fue distinta a la de la alerta.) El resultado se liquida solo el lunes.

## Primera conexión con OddsPapi

La documentación no deja claros algunos detalles. Antes de activar los escaneos:

```bash
apuestas probar-oddspapi --torneo 17 --casa apuestatotal
apuestas probar-oddspapi --torneo 17 --casa pinnacle
```

Revisar `datos/oddspapi_muestra.json` y confirmar:
- [x] (2026-11-22) El nombre de la casa en `bookmakerOdds` es `apuestatotal` y `pinnacle` (si no, cambiar `casa_blanda` / `casa_referencia` en `estrategia.yaml`).
- [x] (2026-11-22) Existe el mercado `101` con resultados `101`, `102`, `103`.
- [ ] `participant1` es el local (se confirma en el primer escaneo: los nombres se emparejan con el historial).
- [x] (2026-11-22) Los nombres no vienen en la respuesta (`None vs None`): el escaneo los busca con `/participants`.
- [x] (2026-11-22) Cada precio trae `changedAt` (el comando muestra "h sin cambios").
- [x] (2026-11-22) Varias casas en una consulta (`apuestatotal,pinnacle`) → **400**. Se usa 1 casa por consulta.
- [x] (2026-11-22) `betfair-ex` trae 1X2 de la Premier (19/21, < 1 h). `sbobet` viejo y caro; `singbet` vacío.
- [x] (2026-11-22) Apuesta Total tiene cuotas para la Premier (torneo 17: 21 partidos, 8 de 9 mostrados con cuotas). LaLiga y Serie A se confirman en el primer escaneo.
