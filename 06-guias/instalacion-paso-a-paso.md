# Instalación paso a paso

Orden recomendado. Cada fase se puede probar antes de pasar a la siguiente.

- [ ] **A.** GitHub y prueba en tu PC (sin gastar nada)
- [ ] **B.** Supabase
- [ ] **C.** OddsPapi
- [ ] **D.** VPS (Docker)
- [ ] **E.** n8n y WhatsApp
- [ ] **F.** Modo prueba (4–8 semanas)

---

## A. GitHub y prueba en tu PC

### A1. Crear los repositorios
En GitHub crea **dos repositorios privados vacíos** (sin README): `apuestas-sistema` y `apuestas-contexto`.

Descomprime los dos .zip que te pasé. En cada carpeta (así el commit queda con tu nombre):

```bash
git init -b main
git add .
git commit -m "v1.0.0: primera version"
git remote add origin https://github.com/<tu-usuario>/apuestas-sistema.git
git push -u origin main
```

(y lo mismo con `apuestas-contexto`, cambiando el nombre en la URL).
Antes del primer `git add`, confirma que **no** hay un archivo `.env` en la carpeta (el `.gitignore` lo excluye igual).

### A2. Instalar y correr las pruebas
Necesitas Python 3.11 o superior.

```bash
cd apuestas-sistema
python -m venv .venv
.venv\Scripts\activate            # Windows  (Linux/Mac: source .venv/bin/activate)
pip install -e ".[dev]"
pytest                            # deben pasar las pruebas (las de base de datos se saltan)
```

### A3. Primer backtest real
Descarga 8 temporadas de la Premier League y simula:

```bash
apuestas backtest --liga E0
apuestas backtest --liga SP1
apuestas backtest --liga I1
```

Los reportes quedan en `datos/backtest/`. **Mándamelos** (los `.md`) y los leemos juntos antes de seguir:
ahí vemos si hay alguna señal de ventaja y ajustamos los umbrales.

---

## B. Supabase

### B1. Crear el proyecto
- supabase.com → **New project**
- Nombre: `apuestas` · Región: **South America (São Paulo)** (la más cercana a Perú)
- Guarda la contraseña de la base de datos en un gestor de contraseñas.

### B2. Cadena de conexión
Botón **Connect** (arriba) → pestaña de cadenas de conexión → **Session pooler**. Se ve así:

```
postgresql://postgres.<ref>:<CONTRASEÑA>@aws-0-sa-east-1.pooler.supabase.com:5432/postgres
```

En tu PC: copia `.env.example` como `.env` y pega la cadena en `DATABASE_URL`.

### B3. Crear las tablas
Opción 1 (desde tu PC):
```bash
apuestas migrar
```
Opción 2: Supabase → **SQL Editor** → pegar `supabase/migrations/0001_esquema_inicial.sql` → **Run**.

Comprueba en **Table Editor** que hay 11 tablas.

### B4. Cargar historial y entrenar (desde tu PC, para probar)
```bash
apuestas historial
apuestas entrenar
```
En Supabase deberías ver partidos en `partidos` y 3 filas en `modelos`.

---

## C. OddsPapi

### C1. Cuenta
oddspapi.io → crear cuenta → copiar la **API key** → `ODDSPAPI_API_KEY` en `.env`.

### C2. Verificar formato e IDs (usa ~4 de tus 250 consultas)
```bash
apuestas listar-torneos --buscar italy
apuestas listar-torneos --buscar peru
apuestas probar-oddspapi --torneo 17 --casa apuestatotal
apuestas probar-oddspapi --torneo 17 --casa pinnacle
```
Sigue la lista de chequeo de `05-operacion/runbook.md` → "Primera conexión con OddsPapi".
Corrige en `config/ligas.yaml` los IDs marcados como `VERIFICAR`.

### C3. Primer escaneo manual
```bash
apuestas escanear --horizonte 72
```
Muestra las alertas en consola (en modo prueba). Revisa también `alias_pendientes` en Supabase.

---

## D. VPS (Docker)

### D1. Revisar Docker
Por SSH:
```bash
docker --version && docker compose version
```
Si no está: en el panel de Hostinger hay plantillas con Docker, o instálalo con la guía oficial de Docker para Ubuntu.

### D2. Bajar el código
El repositorio es privado, así que el VPS necesita permiso de **solo lectura**. Lo más simple:
una **deploy key**.

```bash
ssh-keygen -t ed25519 -f ~/.ssh/apuestas_deploy -N ""
cat ~/.ssh/apuestas_deploy.pub
```
GitHub → `apuestas-sistema` → Settings → Deploy keys → Add (sin permiso de escritura) → pegar la clave.

```bash
cat >> ~/.ssh/config <<'EOF'
Host github-apuestas
  HostName github.com
  IdentityFile ~/.ssh/apuestas_deploy
EOF
git clone git@github-apuestas:<tu-usuario>/apuestas-sistema.git ~/apuestas-sistema
```

### D3. Configurar y levantar
```bash
cd ~/apuestas-sistema
cp .env.example .env
python3 -c "import secrets; print(secrets.token_urlsafe(32))"   # copiar como API_TOKEN
nano .env                                                       # DATABASE_URL, ODDSPAPI_API_KEY, API_TOKEN
chmod 600 .env
docker compose up -d --build
curl http://127.0.0.1:8010/salud
```

### D4. Primera carga (si no la hiciste desde tu PC)
```bash
docker compose exec apuestas-api python -m apuestas migrar
docker compose exec apuestas-api python -m apuestas historial
docker compose exec apuestas-api python -m apuestas entrenar
```

### D5. Actualizar en el futuro
```bash
cd ~/apuestas-sistema && git pull && docker compose up -d --build
```

---

## E. n8n y WhatsApp

Seguir `apuestas-sistema/n8n/README.md`:
1. Credencial **Header Auth** `Apuestas API` con el mismo `API_TOKEN`.
2. Importar los 5 flujos.
3. En cada nodo **Config**: `api_url` (`http://127.0.0.1:8010` o `http://apuestas-api:8000` si n8n está en Docker) y `envio_url`.
4. Ajustar el nodo **Enviar a WhatsApp (OpenClaw)** a cómo recibe mensajes OpenClaw (pendiente).
5. Probar cada flujo con **Execute workflow**. Luego activarlos.

Si n8n está en Docker, hay que unir ambos contenedores a la misma red: ver comentarios en `docker-compose.yml`.

---

## F. Modo prueba (4–8 semanas)

`modo_prueba: true` en `config/estrategia.yaml`. Las alertas llegan marcadas "no apostar".

Cada lunes, con el resumen:
- [ ] ¿Cuántas alertas hubo? ¿Cuántos descartes y por qué?
- [ ] **CLV medio** de las alertas (en el resumen). ¿Es positivo?
- [ ] ¿Hay nombres de equipos pendientes?
- [ ] Anotar en `apuestas-contexto/bitacora.md`.

Pasar a dinero real (`modo_prueba: false`) solo si el CLV es positivo de forma sostenida.
Empezar con un fondo que puedas perder entero.
