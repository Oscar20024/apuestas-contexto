# Runbook: qué hacer cuando…

## …llega un mensaje "FALLO en Apuestas 0X"

1. En el VPS: `cd ~/apuestas-sistema && docker compose logs --tail 100`
2. En Supabase: `select * from ejecuciones order by inicio desc limit 5;` (columna `detalle` tiene el error).
3. Errores típicos:

| Error | Causa | Solución |
|---|---|---|
| `429 Limite mensual de OddsPapi` | Se gastaron las consultas del mes | Esperar al día 1 o subir de plan. Revisar `uso_api` por si algo se repitió de más. |
| `401 Token invalido` | La credencial de n8n no coincide con `API_TOKEN` | Corregir la credencial "Apuestas API" en n8n |
| `could not connect to server` / timeout | Supabase pausado o URL mal | Abrir el proyecto en Supabase (lo despierta); revisar `DATABASE_URL` |
| `Liga X sin modelo entrenado` | No corrió el entrenamiento | `docker compose exec apuestas-api python -m apuestas entrenar` |
| `Historial insuficiente` | Liga nueva sin historial | `... python -m apuestas historial --ligas X` y luego entrenar |

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
2. `apuestas backtest --liga E0 --sin-busqueda` y comparar con el reporte anterior.
3. Si mejora en validación, subir a GitHub, anotar en `bitacora.md`, y en el VPS `git pull` + `docker compose restart`.

## …hice una apuesta real

Registrarla para que el sistema calcule el fondo y el límite de pérdida:

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
- [ ] El nombre de la casa en `bookmakerOdds` es `apuestatotal` y `pinnacle` (si no, cambiar `casa_blanda` / `casa_referencia` en `estrategia.yaml`).
- [ ] Existe el mercado `101` con resultados `101`, `102`, `103`.
- [ ] `participant1` es el local.
- [ ] Si los nombres de equipos no vienen en la respuesta, que el comando muestre `None` es normal: el escaneo los busca con `/participants`.
- [ ] Apuesta Total tiene cuotas para las ligas activas.
