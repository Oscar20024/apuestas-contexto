# Modelo y embudo de decisión

Estado: **v1.4.0 (2026-11-22)**. Catálogo de cada técnica y herramienta: [tecnicas-y-herramientas.md](tecnicas-y-herramientas.md).

## 0. La idea en una frase

Pinnacle y Betfair Exchange son los mercados que mejor predicen el fútbol. Si **Apuesta Total paga más
que el precio justo que dan esos dos mercados**, hay valor. El sistema busca esos casos y te avisa.

Hasta el 2026-11-22 la probabilidad salía de un modelo propio (Dixon-Coles + Elo). El backtest con
partidos reales mostró que perdía (ROI −5 % a −13 %, CLV −6 % en todas las temporadas), y ninguna
mejora lo arregló. Ver [resultados-backtest-2026-11.md](resultados-backtest-2026-11.md) y
[ADR 0011](../02-decisiones/0011-probabilidad-desde-pinnacle.md). El modelo sigue entrenándose y
aparece en la alerta como **dato informativo**.

## 1. De dónde sale cada número

| Dato | Fuente | Cuándo |
|---|---|---|
| Cuotas de Apuesta Total (donde apuestas) | OddsPapi `apuestatotal` | En cada escaneo |
| Cuotas de Pinnacle (referencia principal) | OddsPapi `pinnacle` | En cada escaneo |
| Cuotas de Betfair Exchange (segunda referencia) | OddsPapi `betfair-ex` | En cada escaneo |
| Hora del último cambio de cada cuota | Campo `changedAt` de OddsPapi | En cada escaneo |
| Primera cuota vista de Pinnacle | Tabla `cuotas` (tipo `apertura`) | Desde el primer escaneo del partido |
| Nuestro modelo (informativo) | Modelo activo de la liga en `modelos` | Se reentrena cada lunes |

Cada casa es **1 consulta** a OddsPapi por escaneo (con nuestra clave, pedirlas juntas da error 400).

## 2. El embudo, paso a paso

Configuración: `apuestas-sistema/config/estrategia.yaml` (sección `embudo` y `oddspapi`).
Código: `apuestas-sistema/src/apuestas/decision/embudo.py` (función pura: igual en backtest y en vivo).

| Paso | Pregunta | Si falla (motivo guardado) | Valor actual |
|---|---|---|---|
| 1. Calidad de datos | Solo con `fuente_probabilidad: modelo`: ¿ambos equipos tienen partidos suficientes? | `pocos_datos` | ≥ 10 partidos |
| 2. Probabilidad | ¿Hay cuotas válidas de Pinnacle? Precio justo = promedio sin margen de Pinnacle y Betfair (si Betfair tiene el partido) | `sin_referencia` | método `potencia` |
| 3a. Cuotas | ¿Hay cuotas válidas de Apuesta Total? | `sin_cuotas` | — |
| 3b. Valor | De local, empate y visita, ¿la de mayor `EV = p_justa × cuota − 1` llega al mínimo? | `sin_valor` | 2 % sáb/dom · 3 % lun–vie |
| 4a. Rango | ¿La cuota está en el rango? | `cuota_fuera_rango` | 1,50 – 4,00 |
| 4b. Sospecha | ¿Apuesta Total y el precio justo difieren más de lo creíble? (cuota vieja, error de datos) | `ventaja_sospechosa` | > 10 puntos |
| 4c. Frescura | ¿La cuota de Pinnacle lleva demasiado sin cambiar? | `referencia_vieja` | > 12 h |
| 4d. Consenso | Con Pinnacle y Betfair: ¿**las dos** ven la ventaja mínima? | `sin_consenso` | sí, ambas |
| 4e. Mercado | ¿Pinnacle bajó la probabilidad de la selección desde la primera vez que se vio? | `mercado_en_contra` | > 3 puntos |
| 5. Monto | Kelly × 25 %, con tope y mínimo de la casa | — | tope 2 % · mínimo S/ 2 |

Después, **límites globales** (`servicios/escanear.py`): máximo 3 alertas por día (primero las de mayor
ventaja), máximo 10 % del fondo expuesto por semana, pausa total si en el mes se perdió 15 % del fondo.
Una alerta no se repite para el mismo partido y selección, y caduca cuando empieza el partido.

Todas las evaluaciones (alertas y descartes) se guardan en `evaluaciones` con su motivo y los detalles:
probabilidades de cada referencia, EV por referencia, antigüedad de las cuotas y lo que decía nuestro modelo.

## 3. Ejemplo con números reales

Partido de la Premier del 27-11-2026 (cuotas leídas de OddsPapi el 22-11):

| | Local | Empate | Visita | Margen |
|---|---|---|---|---|
| Apuesta Total | 2,75 | 3,50 | 2,43 | 6,1 % |
| Pinnacle | 2,84 | 3,41 | 2,60 | 3,0 % |
| Betfair Exchange | 2,84 | 3,55 | 2,70 | ~1 % |
| **Precio justo** (promedio sin margen) | 34,6 % | 28,2 % | 37,2 % | — |

- EV de Apuesta Total: local 0,346 × 2,75 − 1 = **−4,8 %**; empate **−1,3 %**; visita **−9,7 %**.
- Ninguno llega al 2 % → descarte `sin_valor`. Es lo normal: Apuesta Total suele pagar menos.

**Caso hipotético con alerta:** si Apuesta Total pagara 2,60 / 3,30 / **2,95**:
- Visita: 0,372 × 2,95 − 1 = **+9,7 %** (con Pinnacle sola +10,5 %, con Betfair sola +8,8 %: hay consenso).
- Cuota 2,95 dentro de 1,50–4,00; diferencia con el precio justo < 10 puntos; Pinnacle reciente y sin moverse en contra.
- Kelly: 25 % de Kelly = 1,2 % del fondo → con S/ 500 = **S/ 6**.
- "Solo si la cuota sigue en **2,74** o más" (debajo, el EV cae del 2 %).

## 4. Nuestro modelo (informativo)

**Dixon-Coles**: ataque y defensa por equipo, ventaja de local, corrección de los marcadores bajos
(0-0, 1-0, 0-1, 1-1), partidos viejos pesan menos (vida media 365 días), ventana de 3 años.

**Elo + regresión logística**: todos empiezan en 1500; sube el que gana (más si fue sorpresa o goleada);
al empezar temporada cada equipo regresa 1/3 hacia 1500; una logística convierte la diferencia en 3 probabilidades.

Mezcla fija 60 % Dixon-Coles + 40 % Elo. Es lo que aparece en la alerta como "Nuestro modelo (solo informativo)".

**Disponibles para pruebas (no deciden nada):** pi-ratings con goles y con tiros a puerta, y un combinador
(stacking) que aprende a juntar modelos y mercado. Se activan con `usar_pi`, `usar_pi_tiros` y
`config/combinador.yaml`. En el backtest no mejoraron.

**Para volver a decidir con el modelo:** `embudo.fuente_probabilidad: modelo` en `estrategia.yaml`
(sin tocar código). Solo tiene sentido si un backtest nuevo muestra que le gana a Pinnacle.

## 5. Qué dice la alerta

```
[MODO PRUEBA - no apostar, solo registrar]
ALERTA DE VALOR - Premier League
Equipo A vs Equipo B
sab 28/11 10:00 (hora Lima)

Apuesta: gana Equipo B (visita)
Cuota Apuesta Total: 2.95
Prob. justa (Pinnacle + Betfair Exchange): 37.2% | Apuesta Total: 33.0%
Cuota de Apuesta Total sin cambios hace 25 min
Nuestro modelo (solo informativo): 34.0%
Ventaja estimada: +9.7%
Monto sugerido: S/ 6 (1.2% del fondo)
Solo si la cuota sigue en 2.74 o mas.

ID: 1287
```

"Prob. justa" es el promedio de las referencias que tenían el partido (si Betfair no lo tiene, dice solo Pinnacle). "Apuesta Total" es la probabilidad que implica su cuota
(sin su margen): cuanto más baja que la justa, más paga Apuesta Total de lo que debería.

## 6. Cómo leer un backtest

```bash
apuestas comparar --liga E0 --csv datos/football_data/E0_*.csv   # variantes del modelo y el mercado
apuestas backtest --liga E0                                       # reporte del embudo con búsqueda de umbrales
```

Qué mirar, en orden:

1. **Log-loss frente al mercado.** Si el modelo es peor que Pinnacle previa, no sirve para decidir.
2. **CLV medio.** Positivo = las cuotas elegidas eran mejores que el cierre de Pinnacle. Es la mejor señal.
3. **ROI con su error típico.** Con 100 apuestas a cuota ~2,5 el error es ±10–15 %: un ROI de +8 % no prueba nada.
4. **Validación.** Umbrales elegidos en unas temporadas y probados en otra. Si se derrumba, eran suerte.

**Limitación del backtest de la estrategia actual:** Apuesta Total no tiene historial público, así que se
usa Bet365 como sustituto. Bet365 casi nunca paga más que Pinnacle (0–7 alertas por liga en 4,5 temporadas).
Apuesta Total puede comportarse distinto: **solo se sabe en vivo**, y para eso es el modo prueba.
Tampoco hay historial de `changedAt` ni de Betfair, así que los filtros 4c y 4d no se pueden backtestear.
