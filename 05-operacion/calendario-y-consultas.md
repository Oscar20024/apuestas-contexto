# Calendario y presupuesto de consultas

Hora de Lima (UTC−5). Los horarios están en los flujos de n8n (`apuestas-sistema/n8n/flujos/`).

| Día | Hora | Flujo | Horizonte | Consultas OddsPapi* |
|---|---|---|---|---|
| Lunes | 7:00 | Historial + entrenar + resumen semanal | — | 0 |
| Mar, mié, jue | 9:00 | Escaneo entre semana | 48 h | 2 |
| Viernes | 12:00 | Escaneo del fin de semana | 72 h | 2 |
| Sábado y domingo | 6:00 y 10:00 | Revisión antes de los partidos | 12 h | 2 + 2 |

\* Con hasta 5 ligas activas. Cada escaneo = 1 consulta por casa (Apuesta Total y Pinnacle) por cada grupo de 5 ligas.

## Presupuesto mensual

| Ligas activas | Consultas por escaneo | Escaneos/mes | Consultas/mes | De 250 gratis |
|---|---|---|---|---|
| 1–5 | 2 | ~35 | ~70 | 28% |
| 6–10 | 4 | ~35 | ~140 | 56% |
| 11–15 | 6 | ~35 | ~210 | 84% (justo) |

Extras: 1 consulta al mes para la lista de equipos (participantes) y las que uses en pruebas.
El sistema se bloquea solo al llegar a 230 (deja 20 de reserva).

## Partidos europeos en hora de Lima

En invierno europeo (noviembre–marzo) Lima está 5 h detrás de Londres y 6 h detrás de Madrid/Roma.
Los partidos de fin de semana van de ~6:30 a ~15:00 en Lima. Si se agregan ligas sudamericanas
(tardes y noches), conviene sumar un escaneo a las 14:00.

## ¿Cuándo llegan las alertas para apostar?

- **Viernes 12:00**: primera lista para el fin de semana.
- **Sáb/dom 6:00 y 10:00**: confirmaciones. Si una cuota cambió o Pinnacle se movió en contra, el partido se descarta.
- **Mar–jue 9:00**: solo partidos entre semana con ventaja ≥ 8%.

Una alerta no se repite para el mismo partido y selección.
