# apuestas-contexto

Contexto, decisiones y documentación del **sistema de predicción de apuestas**.
El código vive en otro repositorio: **`apuestas-sistema`** (es lo que se despliega en el VPS).

¿Por qué dos repositorios? Ver [ADR 0001](02-decisiones/0001-dos-repositorios.md).

## Índice

| Carpeta | Contenido |
|---|---|
| [00-vision](00-vision/vision.md) | Qué es, qué no es, objetivos y criterios de éxito |
| [01-arquitectura](01-arquitectura/arquitectura.md) | Componentes, flujos y diagramas |
| [02-decisiones](02-decisiones/) | Decisiones de diseño (ADR) y su porqué |
| [03-datos](03-datos/modelo-de-datos.md) | Tablas de Supabase y fuentes de datos |
| [04-modelo](04-modelo/modelo-y-embudo.md) | Cómo predice el modelo y cómo decide el embudo; cómo leer un backtest |
| [05-operacion](05-operacion/calendario-y-consultas.md) | Calendario, presupuesto de consultas, qué hacer si algo falla |
| [06-guias](06-guias/instalacion-paso-a-paso.md) | Instalación paso a paso y [conexiones](06-guias/conexiones.md) |
| [preguntas-abiertas.md](preguntas-abiertas.md) | Lo que falta definir |
| [bitacora.md](bitacora.md) | Historial de cambios del proyecto |

## Para agentes de IA

Leer primero [AGENTS.md](AGENTS.md): resume el proyecto y las reglas que no se deben romper.
