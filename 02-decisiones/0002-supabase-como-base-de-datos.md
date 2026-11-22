# 0002 · Supabase (Postgres) como base de datos

**Estado:** Aceptada · **Fecha:** 2026-11-22

## Contexto
Oscar ya usa Supabase y lo quiere como capa de datos de sus proyectos. La futura app Android necesita leer datos con autenticación.

## Decisión
- Proyecto de Supabase **separado** de sus otros negocios (no comparten los 500 MB del plan gratis).
- El backend se conecta por la cadena **Session pooler** (puerto 5432, funciona por IPv4).
- RLS activado en todas las tablas desde el inicio.
- Cuotas guardadas en formato "ancho" (una fila = 3 cuotas de una casa en un momento) y solo para 2 casas, para ahorrar espacio.

## Alternativas
- Postgres dentro del VPS: más espacio, pero sin panel, backups ni auth listos para la app.

## Consecuencias
- Estimación: 10 ligas × 10 temporadas ≈ 38.000 partidos ≈ 50–80 MB + cuotas. Entra holgado en 500 MB.
- El plan gratis pausa proyectos tras 1 semana sin actividad; los escaneos diarios lo mantienen activo.
