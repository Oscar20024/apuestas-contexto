# Instalación paso a paso

Orden recomendado. Cada fase se puede probar antes de pasar a la siguiente.
Detalle de cada conexión: [conexiones.md](conexiones.md).

- [x] **A.** GitHub (`Oscar20024/apuestas-sistema` y `apuestas-contexto`, privados)
- [ ] **A2.** Prueba y backtest en tu PC
- [x] **B.** Supabase: `BDapuestas` conectado
- [x] **C.** OddsPapi (key válida, IDs 17/8/23 verificados; 35 Bundesliga y 34 Ligue 1 activadas y verificadas el 2026-11-22)
- [x] **D.** VPS: historial cargado, modelos entrenados, `verificar` 16 ok / 0 avisos, `instalar.sh` hecho (timers activos), `probar-whatsapp` OK
- [x] **D2.** Estrategia v1.4.0 desplegada: precio justo Pinnacle + Betfair, 6 timers (incluye `previa` 12:30). Si se agregan timers, volver a correr `sudo bash vps/instalar.sh`
- [~] **E.** Modo prueba — empieza el lunes 2026-11-23. Medir oportunidades (1–2 semanas) y CLV

Todos los comandos en una sola hoja: [05-operacion/comandos.md](../05-operacion/comandos.md).

Herramientas que ayudan en todo el camino:
- `apuestas configurar`: arma el `.env` pidiéndote cada dato y prueba la base de datos al momento.
- `apuestas verificar`: revisa todo y dice cómo arreglar lo que falle.

---

## A2. Prueba y backtest en tu PC

Necesitas Python 3.11 o superior (en Windows: python.org, marcando "Add to PATH").

```bash
cd C:\Users\lenovo\Downloads\apuestas-sistema
python -m venv .venv
.venv\Scripts\activate
pip install -e ".[dev]"
pytest
apuestas backtest --liga E0
apuestas backtest --liga SP1
apuestas backtest --liga I1
```

Los reportes quedan en `datos/backtest/`. **Mándame los `.md`** y los revisamos juntos.

---

## B. Supabase (`BDapuestas`)

Las tablas ya están creadas. Solo falta tu cadena de conexión:

1. Supabase → `BDapuestas` → **Connect** → **Session pooler** → copiar.
2. Reemplazar `[YOUR-PASSWORD]` por tu contraseña de la base de datos.
3. En tu PC (con el `.venv` activado):
   ```bash
   apuestas configurar        # pega la cadena; esquema: public; OddsPapi puede ir vacío por ahora
   apuestas migrar            # "No habia migraciones pendientes"
   apuestas verificar
   apuestas historial         # baja 8 temporadas de las ligas activas a Supabase
   apuestas entrenar
   apuestas verificar         # ahora "Datos E0/SP1/I1: OK"
   ```

---

## C. OddsPapi

1. oddspapi.io → crear cuenta → copiar la **API key** → `apuestas configurar` (paso 3).
2. Verificar (≈3 de tus 250 consultas):
   ```bash
   apuestas verificar --con-oddspapi
   apuestas probar-oddspapi --torneo 17 --casa apuestatotal
   apuestas probar-oddspapi --torneo 17 --casa pinnacle
   ```
3. Revisar la lista de chequeo en `05-operacion/runbook.md` → "Primera conexión con OddsPapi".

---

## D. VPS

Todo por SSH en el VPS (`srv1864174`), como root. Sigue la convención del VPS: proyectos en `/docker/`.

### D1. Probar el envío por WhatsApp (antes de instalar nada)
```bash
docker exec openclaw-wr93-openclaw-1 openclaw message send --channel whatsapp --target +51TU_NUMERO --message "prueba"
```
Si te llega, seguimos. Si da error, mándame el mensaje.

### D2. Bajar el código (repositorio privado → deploy key de solo lectura)
```bash
ssh-keygen -t ed25519 -f ~/.ssh/apuestas_deploy -N ""
cat ~/.ssh/apuestas_deploy.pub
```
GitHub → `apuestas-sistema` → **Settings → Deploy keys → Add deploy key** (sin permiso de escritura) → pegar.
```bash
cat >> ~/.ssh/config <<'EOF'
Host github-apuestas
  HostName github.com
  IdentityFile ~/.ssh/apuestas_deploy
EOF
git clone git@github-apuestas:Oscar20024/apuestas-sistema.git /docker/apuestas-sistema
cd /docker/apuestas-sistema
```

### D3. `.env` y API
Pega el contenido del `.env` de tu PC (el que armó `apuestas configurar`):
```bash
nano .env
chmod 600 .env
mkdir -p datos && chown -R 10001:10001 datos
docker compose up -d --build
curl http://127.0.0.1:8010/salud
docker compose exec apuestas-api python -m apuestas verificar
```
La API queda solo en `127.0.0.1:8010` (no se expone a internet) y con 768 MB / 0.5 CPU como máximo.

### D4. Programador y mensajero
```bash
sudo bash vps/instalar.sh                       # pide tu número de WhatsApp una vez
python3 vps/apuestas_host.py probar-whatsapp    # mensaje directo de OpenClaw
sudo systemctl start apuestas-tarea@prueba.service
journalctl -u 'apuestas-*' -n 30 --no-pager
```
Te debe llegar "PRUEBA DE CONEXIONES" con el estado de todo. Desde ahí los horarios corren solos
(`systemctl list-timers 'apuestas-*'`).

### D5. Actualizar en el futuro
```bash
cd /docker/apuestas-sistema && git pull && docker compose up -d --build && sudo bash vps/instalar.sh
```

---

## E. Modo prueba (4–8 semanas)

`modo_prueba: true` en `config/estrategia.yaml`. Las alertas llegan marcadas "no apostar".

Cada lunes, con el resumen:
- [ ] ¿Cuántas alertas hubo? ¿Cuántos descartes y por qué?
- [ ] **CLV medio** de las alertas. ¿Es positivo?
- [ ] ¿Hay nombres de equipos pendientes?
- [ ] Anotar en `apuestas-contexto/bitacora.md`.

Pasar a dinero real (`modo_prueba: false`) solo si el CLV es positivo de forma sostenida, y empezar
con un fondo que puedas perder entero.
