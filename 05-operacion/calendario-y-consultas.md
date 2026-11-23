# Calendario y presupuesto de consultas

Hora de Lima (UTC−5). Los horarios están en los timers de systemd del VPS (`apuestas-sistema/vps/systemd/`).
Ver próximas ejecuciones: `systemctl list-timers 'apuestas-*'`.

| Día | Hora | Tarea | Horizonte | Consultas OddsPapi* |
|---|---|---|---|---|
| Lunes | 7:00 | Historial + entrenar + resumen semanal | — | 0 |
| Mar, mié, jue | 9:00 | Escaneo entre semana | 48 h | 2 |
| Viernes | 12:00 | Escaneo del fin de semana | 72 h | 2 |
| Sábado y domingo | 6:00 y 10:00 | Revisión antes de los partidos | 12 h | 2 + 2 |
| **Todos menos viernes** | **12:30** | **Previa: revisión cerca de los partidos** (desde v1.4.0) | 8 h | 2 |

\* Con hasta 5 ligas: **1 consulta por casa** (Apuesta Total y Pinnacle = 2; con una referencia extra = 3).
La documentación de OddsPapi dice que se pueden pedir hasta 3 casas juntas, pero con nuestra clave
`apuestatotal,pinnacle` respondió **400** (2026-11-22). Por eso `oddspapi.casas_por_consulta: 1`.

## Presupuesto mensual (v1.4.0)

| Configuración | Escaneos/mes | Consultas/mes | De 250 gratis |
|---|---|---|---|
| 1–5 ligas, Apuesta Total + Pinnacle | ~61 | ~122 | 49 % |
| **1–5 ligas, + 1 referencia extra (actual: 5 ligas + Betfair)** | ~61 | ~183 | 73 % |
| 6–10 ligas, Apuesta Total + Pinnacle | ~61 | ~244 | **no alcanza** |

Extras: 1 consulta al mes para la lista de equipos (participantes) y las que uses en pruebas.
Si se sube `casas_por_consulta` y la API responde 400, el sistema pide las casas de a una
(gasta 1 consulta extra por la rechazada). El sistema se bloquea solo al llegar a 230 (deja 20 de reserva).

## Partidos europeos en hora de Lima

En invierno europeo (noviembre–marzo) Lima está 5 h detrás de Londres y 6 h detrás de Madrid/Roma.
Los partidos de fin de semana van de ~6:30 a ~15:00 en Lima; entre semana, ~12:30 a ~15:00.
La previa de las 12:30 cubre los partidos de la tarde de Lima. Si se agregan ligas sudamericanas
(tardes y noches), conviene sumar un escaneo a las 17:00.

## ¿Cuándo llegan las alertas para apostar?

- **Viernes 12:00**: primera lista para el fin de semana.
- **Sáb/dom 6:00 y 10:00**: confirmaciones.
- **12:30, todos menos viernes**: cuotas de Apuesta Total que no se actualizaron cerca del partido.
  (El viernes ya está el escaneo de las 12:00.)
- **Mar–jue 9:00**: partidos entre semana (ventaja mínima 3 %).

Una alerta no se repite para el mismo partido y selección.
