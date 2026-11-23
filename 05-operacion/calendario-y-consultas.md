# Calendario y presupuesto de consultas

Hora de Lima (UTC−5). Los horarios están en los timers de systemd del VPS (`apuestas-sistema/vps/systemd/`).
Ver próximas ejecuciones: `systemctl list-timers 'apuestas-*'`.

| Día | Hora | Tarea | Horizonte | Consultas OddsPapi* |
|---|---|---|---|---|
| Lunes | 7:00 | Historial + entrenar + resumen semanal | — | 0 |
| Mar, mié, jue | 9:00 | Escaneo entre semana | 48 h | 1 |
| Viernes | 12:00 | Escaneo del fin de semana | 72 h | 1 |
| Sábado y domingo | 6:00 y 10:00 | Revisión antes de los partidos | 12 h | 1 + 1 |
| **Todos los días** | **12:30** | **Previa: revisión cerca de los partidos** (desde v1.4.0) | 8 h | 1 |

\* Con hasta 5 ligas y hasta 3 casas (Apuesta Total + Pinnacle + 1 referencia extra): **1 consulta por escaneo**.
Desde v1.4.0 las casas van juntas en una consulta (antes era 1 por casa). `betfair-ex` siempre va sola: +1.

## Presupuesto mensual (v1.4.0)

| Configuración | Escaneos/mes | Consultas/mes | De 250 gratis |
|---|---|---|---|
| 1–5 ligas, AT + Pinnacle (+1 extra) | ~65 | ~65 | 26 % |
| 1–5 ligas, + betfair-ex | ~65 | ~130 | 52 % |
| 6–10 ligas, AT + Pinnacle (+1 extra) | ~65 | ~130 | 52 % |

Extras: 1 consulta al mes para la lista de equipos (participantes) y las que uses en pruebas.
Si la API no aceptara varias casas juntas, el sistema las pide por separado (el doble de consultas;
igual entra en el presupuesto). El sistema se bloquea solo al llegar a 230 (deja 20 de reserva).

## Partidos europeos en hora de Lima

En invierno europeo (noviembre–marzo) Lima está 5 h detrás de Londres y 6 h detrás de Madrid/Roma.
Los partidos de fin de semana van de ~6:30 a ~15:00 en Lima; entre semana, ~12:30 a ~15:00.
La previa de las 12:30 cubre los partidos de la tarde de Lima. Si se agregan ligas sudamericanas
(tardes y noches), conviene sumar un escaneo a las 17:00.

## ¿Cuándo llegan las alertas para apostar?

- **Viernes 12:00**: primera lista para el fin de semana.
- **Sáb/dom 6:00 y 10:00**: confirmaciones.
- **Todos los días 12:30**: cuotas de Apuesta Total que no se actualizaron cerca del partido.
- **Mar–jue 9:00**: partidos entre semana (ventaja mínima 3 %).

Una alerta no se repite para el mismo partido y selección.
