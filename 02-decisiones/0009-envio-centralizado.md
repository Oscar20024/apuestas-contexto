# 0009 · Envío de mensajes centralizado en un flujo de n8n

**Estado:** Reemplazada por [0010](0010-programador-systemd-y-mensajero.md) (n8n no está instalado en el VPS) · **Fecha:** 2026-11-22

## Contexto
Varios flujos mandan mensajes (alertas, resumen, errores, prueba). Todavía no está decidido cómo
OpenClaw recibirá los mensajes, y depende de cómo esté instalado n8n.

## Decisión
- Un único flujo **98 - Enviar mensaje** recibe `{mensaje}` y lo entrega. Los demás lo llaman.
- Tres modos, elegidos en su nodo **Config envio**:
  - `comando` (preferido): `openclaw message send --channel whatsapp --target <número> --message <texto>`.
    Entrega el texto exacto, sin pasar por el modelo. El texto viaja en base64 para que comillas,
    tildes o saltos de línea no rompan el comando ni permitan inyectar órdenes.
  - `webhook_openclaw`: POST a `/hooks/agent` del Gateway con entrega a WhatsApp. Pasa por el agente
    (Gemini): se le pide reenviar sin cambios, pero no es una garantía. Solo si n8n corre en Docker.
  - `webhook`: POST `{"mensaje": ...}` a cualquier URL.
- Flujo **00 - Prueba de conexiones** para comprobar todo el circuito de punta a punta.

## Consecuencias
- Cambiar de canal (WhatsApp → app Android, Telegram, etc.) es tocar un solo flujo.
- Con `webhook_openclaw` hay que comparar las primeras alertas contra `evaluaciones` en Supabase.
