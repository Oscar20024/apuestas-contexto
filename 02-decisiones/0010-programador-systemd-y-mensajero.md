# 0010 · Programador con systemd y mensajero con bandeja de salida

**Estado:** Aceptada · **Fecha:** 2026-11-22 · Reemplaza a [0009](0009-envio-centralizado.md) y ajusta [0004](0004-n8n-solo-orquesta.md)

## Contexto
Datos reales del VPS (inventario del 2026-11-01):
- n8n **no está instalado**.
- 1 vCPU y 3.8 GB de RAM, compartidos con OpenClaw, memory-api, riachuelo (web y worker) y Traefik.
- OpenClaw corre en Docker (`openclaw-wr93-openclaw-1`). Su Gateway escucha solo dentro del
  contenedor y se administra con `docker exec ... openclaw ...`.
- Gemini falla con frecuencia (503, 429 y 404 en el último modelo de respaldo).
- UFW está desactivado.

## Decisión
1. **Programador = timers de systemd** con hora de Lima (`OnCalendar=... America/Lima`,
   `Persistent=true`). Cada timer llama a `POST /tareas/<nombre>` de la API.
2. **Bandeja de salida** (`mensajes_salida`): la API nunca envía. Deja cada mensaje con su
   caducidad; las alertas caducan al empezar el partido.
3. **Mensajero en el VPS** (`vps/apuestas_host.py`, solo biblioteca estándar de Python): lee la
   bandeja y entrega con `docker exec <openclaw> openclaw message send --channel whatsapp`, pasando el
   texto como argumento sin shell. Registra el resultado y reintenta hasta 5 veces.
4. Si la API o la base de datos fallan, el programador avisa **directo** por WhatsApp (sin bandeja).
5. n8n pasa a ser **opcional**: sus flujos solo llaman a `/tareas/...`.

## Alternativas descartadas
- **Instalar n8n ahora:** suma RAM, actualizaciones y una interfaz web que proteger en un VPS sin
  firewall, solo para disparar 5 horarios.
- **Webhook `/hooks/agent` de OpenClaw:** el texto pasa por Gemini (puede alterar cifras y hoy falla
  seguido) y requiere abrir el Gateway a la red de Docker.
- **Montar `docker.sock` en el contenedor de la API:** equivale a darle root del VPS a la API.

## Consecuencias
- El texto de las alertas llega exacto y no depende de Gemini.
- Una caída de WhatsApp no pierde alertas: quedan en la bandeja.
- El mensajero corre como root (lo necesita para `docker exec`). Por eso es pequeño, sin
  dependencias, y solo ejecuta un comando fijo.
- La API corre con UID 10001 (no 1000, el UID que ya comparten OpenClaw y memory-api) y con el
  contenedor endurecido: solo lectura, sin capacidades, 768 MB y 0.5 CPU.
