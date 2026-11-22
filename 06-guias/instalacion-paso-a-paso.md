# Instalación paso a paso

Orden recomendado. Cada fase se puede probar antes de pasar a la siguiente.
Detalle de cada conexión (qué dato, dónde va, errores típicos): [conexiones.md](conexiones.md).

Herramientas que te ayudan en todo el camino:
- `apuestas configurar`: arma el `.env` pidiéndote cada dato y prueba la base de datos al momento.
- `apuestas verificar`: revisa todo y te dice cómo arreglar lo que falle.

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

### B1. Elegir el proyecto
Tu cuenta gratis ya tiene 2 proyectos activos (el máximo del plan gratis). Opciones en
[conexiones.md → Supabase](conexiones.md#1-supabase):
- **A.** Pausar uno y crear `apuestas` (región **South America (São Paulo)**) → `DB_SCHEMA=public`.
- **B.** Usar un proyecto existente → `DB_SCHEMA=apuestas` (las tablas quedan aparte).

### B2. Cadena de conexión y `.env`
Botón **Connect** (arriba) → **Session pooler**. Se ve así:

```
postgresql://postgres.<ref>:<CONTRASEÑA>@aws-0-sa-east-1.pooler.supabase.com:5432/postgres
```

En tu PC, dentro de `apuestas-sistema`:
```bash
apuestas configurar
```
Pega la cadena (te dice al instante si conecta), elige el esquema, y deja la API key de OddsPapi
vacía por ahora si aún no la tienes. El token de la API se genera solo.

### B3. Crear las tablas
```bash
apuestas migrar
apuestas verificar
```
`migrar` solo aplica lo que falte (se puede correr las veces que quieras).
`verificar` debe mostrar "Tablas: OK" y avisos de "Sin historial cargado" (normal en este punto).

### B4. Cargar historial y entrenar (desde tu PC, para probar)
```bash
apuestas historial
apuestas entrenar
```
En Supabase deberías ver partidos en `partidos` y 3 filas en `modelos`.

---

## C. OddsPapi

### C1. Cuenta
oddspapi.io → crear cuenta → copiar la **API key** → `apuestas configurar` (paso 3).

### C2. Verificar key, IDs y formato (usa ~3 de tus 250 consultas)
```bash
apuestas verificar --con-oddspapi
apuestas probar-oddspapi --torneo 17 --casa apuestatotal
apuestas probar-oddspapi --torneo 17 --casa pinnacle
```
`verificar --con-oddspapi` muestra el nombre del torneo de cada ID: confirma que cada uno es la liga correcta.
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
Lo más simple: usar el mismo `.env` que armaste en tu PC (pegando su contenido, nunca por GitHub).
```bash
cd ~/apuestas-sistema
nano .env                  # pegar el contenido del .env de tu PC
chmod 600 .env
docker compose up -d --build
curl http://127.0.0.1:8010/salud
docker compose exec apuestas-api python -m apuestas verificar
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

Seguir `apuestas-sistema/n8n/README.md`. En corto:
1. Credencial **Header Auth** `Apuestas API` con el mismo `API_TOKEN`.
2. Importar **primero el flujo 98** (envío) y completar su nodo **Config envio**: modo y tu número.
3. Importar 00, 01, 02, 03, 04 y 99. En cada uno: `api_url` en **Config** y elegir el flujo 98 en **Enviar por WhatsApp**.
4. Ejecutar **Apuestas 00 - Prueba de conexiones**. Si te llega el mensaje, activar 01–04 y 99.

Si n8n está en Docker, hay que unir ambos contenedores a la misma red (ver `docker-compose.yml`)
y usar el modo de envío `webhook_openclaw`.

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
