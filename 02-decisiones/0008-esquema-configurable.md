# 0008 · Esquema de base de datos configurable (`DB_SCHEMA`)

**Estado:** Aceptada · **Fecha:** 2026-11-22 · Complementa [0002](0002-supabase-como-base-de-datos.md)

## Contexto
La organización de Supabase está en el plan gratis y ya tiene 2 proyectos activos (`saas-dev` y
`movil-negocios`), que es el máximo del plan. Crear un proyecto nuevo obliga a pausar uno o pagar.

## Decisión
- Variable `DB_SCHEMA` (por defecto `public`). Si vale `apuestas`, `apuestas migrar` crea ese esquema
  y todas las tablas quedan ahí; la conexión pone ese esquema primero en el `search_path`.
- Las migraciones se registran en `migraciones_aplicadas`: `migrar` solo aplica lo que falte.

## Consecuencias
- El sistema puede vivir dentro de un proyecto existente sin chocar con sus tablas (incluso si ese
  proyecto tiene su propia tabla `ligas` o `partidos`).
- Se comparte el límite de 500 MB con ese proyecto: `apuestas verificar` avisa al pasar el 80%.
- Hay que usar el **Session pooler** (5432): con el Transaction pooler el `search_path` no se mantiene.
