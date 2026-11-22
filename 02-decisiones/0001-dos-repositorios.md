# 0001 · Dos repositorios: sistema y contexto

**Estado:** Aceptada · **Fecha:** 2026-11-22

## Contexto
Oscar quiere usar GitHub para control de versiones, contexto del proyecto y documentación, en dos carpetas independientes.

## Decisión
- `apuestas-sistema`: solo lo que se despliega en el VPS (código, config, Docker, migraciones, scripts del VPS, pruebas).
- `apuestas-contexto`: visión, arquitectura, decisiones, guías, bitácora y contexto para agentes de IA (`AGENTS.md`).

Ambos privados.

## Consecuencias
- El VPS solo clona `apuestas-sistema`; la documentación no viaja al servidor.
- Los agentes de IA pueden leer `apuestas-contexto` sin tocar el código.
- Hay que acordarse de actualizar la bitácora cuando cambia el sistema (regla en `AGENTS.md`).
