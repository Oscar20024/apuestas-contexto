# 0004 · n8n solo orquesta; el cálculo vive en la API

**Estado:** Aceptada · **Fecha:** 2026-11-22

## Decisión
n8n solo hace tres cosas: disparar horarios, llamar a `apuestas-api` y reenviar los mensajes. Toda la lógica (modelo, embudo, límites, mensajes) está en Python, versionada y con pruebas.

## Por qué
- La lógica en n8n no se puede probar con `pytest` ni revisar bien en GitHub.
- El backtest y la producción usan exactamente el mismo código del embudo.
- Si se cambia n8n o el canal (WhatsApp → app), el sistema no cambia.
