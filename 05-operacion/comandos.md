# Comandos de uso diario

Hoja de referencia con los comandos que usamos. Rutas reales:
- PC (Windows): `C:\Users\lenovo\Downloads\apuestas-sistema` y `...\apuestas-contexto`
- VPS (`srv1864174`, como root): `/docker/apuestas-sistema`

## Regla de oro: qué viaja por GitHub y qué no

| Qué cambia | Cómo llega al VPS |
|---|---|
| Código, `config/*.yaml`, migraciones, scripts, documentación | PC → `git push` → VPS `git pull` |
| `.env` (contraseña de Supabase, keys, token) | **A mano** con `nano .env` en el VPS. Nunca por GitHub |

- Las credenciales van en `.env`, **nunca** en `.env.example` (ese sí se sube a GitHub).
- En la PC, `.env.vps` es una copia de `.env` con solo las 4 líneas `CLAVE=valor`, para pegar en el VPS.
  Git ignora tanto `.env` como `.env.vps`.
- El `.env` del VPS debe tener **solo** estas 4 líneas: `DATABASE_URL`, `DB_SCHEMA`, `ODDSPAPI_API_KEY`, `API_TOKEN`.
  Nunca pegar comandos dentro de nano.

---

## PC (Windows)

### Subir cambios a GitHub
```bash
cd C:\Users\lenovo\Downloads\apuestas-sistema
git status                 # .env y .env.vps NO deben aparecer
git add .
git commit -m "descripcion del cambio"
git push

cd C:\Users\lenovo\Downloads\apuestas-contexto
git add .
git commit -m "descripcion del cambio"
git push
```

### Borrar un archivo que ya estaba en el repo
```bash
git rm ruta/del/archivo
git commit -m "Quitar archivo"
git push
```

### Python en la PC (opcional; todo puede correr en el VPS)
```bash
python -m venv .venv
.venv\Scripts\activate
pip install -e ".[dev]"
pytest
apuestas backtest --liga E0
```

---

## VPS

### Entrar a la carpeta
```bash
cd /docker/apuestas-sistema
```

### Bajar cambios de GitHub y aplicarlos
```bash
git pull
docker compose up -d --build --force-recreate
sudo bash vps/instalar.sh          # solo si cambiaron archivos de vps/
```
`--force-recreate` también hace que el contenedor vuelva a leer el `.env`.

### Editar credenciales
```bash
nano .env                          # Ctrl+O + Enter guarda · Ctrl+X sale
cut -d= -f1 .env                   # muestra solo los nombres: deben ser los 4
chmod 600 .env
docker compose up -d --force-recreate
```

### Limpiar un `.env` dañado (deja solo las líneas CLAVE=valor)
```bash
grep -E '^[[:space:]]*[A-Z_]+=' .env | sed 's/^[[:space:]]*//; s/\r$//' > /tmp/env.limpio && mv /tmp/env.limpio .env
```

### Estado de la API
```bash
docker compose ps                                  # debe decir Up (healthy)
curl http://127.0.0.1:8010/salud
docker compose logs --tail 100
```

### Verificar conexiones
```bash
docker compose exec apuestas-api python -m apuestas verificar
docker compose exec apuestas-api python -m apuestas verificar --con-oddspapi   # gasta 1 consulta
```

### Datos y modelo
```bash
docker compose exec apuestas-api python -m apuestas migrar       # crea/actualiza tablas (solo lo que falte)
docker compose exec apuestas-api python -m apuestas historial    # baja resultados (8 temporadas)
docker compose exec apuestas-api python -m apuestas entrenar     # entrena el modelo de cada liga
docker compose exec apuestas-api python -m apuestas backtest --liga E0
docker compose exec apuestas-api python -m apuestas tarea prueba # mensaje de prueba a la bandeja
```

### Programador y mensajero (después de `vps/instalar.sh`)
```bash
sudo bash vps/instalar.sh                          # instala/actualiza timers y mensajero
python3 vps/apuestas_host.py probar-whatsapp       # mensaje directo por OpenClaw
python3 vps/apuestas_host.py estado                # ¿responde la API?
sudo systemctl start apuestas-tarea@prueba.service # prueba completa de punta a punta
sudo systemctl start apuestas-tarea@simulacro.service # UNA alerta inventada (no cuenta en estadísticas)
systemctl list-timers 'apuestas-*'                 # próximas ejecuciones
journalctl -u 'apuestas-*' -n 100 --no-pager       # qué pasó
```

### Registrar una apuesta real
```bash
docker compose exec apuestas-api python -m apuestas registrar-apuesta --evaluacion <ID> --monto 10 --cuota 2.08
```

---

## OpenClaw (en el VPS)

```bash
docker exec openclaw-wr93-openclaw-1 openclaw config get channels.whatsapp.allowFrom
docker exec openclaw-wr93-openclaw-1 openclaw config set channels.whatsapp.allowFrom '["XXXXXXXXX", "+51XXXXXXXXX"]' --strict-json
docker exec openclaw-wr93-openclaw-1 openclaw message send --channel whatsapp --target +51XXXXXXXXX --message "prueba"
docker exec openclaw-wr93-openclaw-1 printenv WHATSAPP_NUMBER
```
No reiniciar OpenClaw sin necesidad: al arrancar puede reescribir `allowFrom` (ver runbook).

---

## GitHub (acceso del VPS)

- El VPS baja el código con una **deploy key de solo lectura** (`/root/.ssh/apuestas_deploy`, alias `github-apuestas` en `/root/.ssh/config`).
- Clonar de nuevo si hiciera falta:
  ```bash
  git clone git@github-apuestas:Oscar20024/apuestas-sistema.git /docker/apuestas-sistema
  ```

---

## Supabase (panel web)

- Cadena de conexión: `BDapuestas` → **Connect** → **Session pooler** (puerto 5432).
- Cambiar la contraseña de la base: menú **Database** → **Settings** → **Reset database password** → **Generate a password**.
  Ojo: no es ninguna de las API keys (`sb_publishable_...`, `sb_secret_...`).
  Si tiene símbolos, en la URL van codificados: `?`→`%3F`, `#`→`%23`, `@`→`%40`, `:`→`%3A`, `/`→`%2F`, `%`→`%25`.
  Después: actualizar `DATABASE_URL` en el `.env` del VPS, y en `.env` y `.env.vps` de la PC.
- Consultas útiles: ver `03-datos/modelo-de-datos.md`.
