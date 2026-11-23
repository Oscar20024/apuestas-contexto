# Modelo y embudo de decisión

> **Cambio 2026-11-22 ([ADR 0011](../02-decisiones/0011-probabilidad-desde-pinnacle.md)):** la probabilidad que
> decide las alertas sale de **Pinnacle sin margen** (`embudo.fuente_probabilidad: mercado`). Se alerta cuando
> Apuesta Total paga más que ese precio justo. El modelo de abajo sigue entrenándose y aparece en la alerta como
> dato informativo. Motivo: en el backtest perdía (ver `resultados-backtest-2026-11.md`).
> Componentes nuevos disponibles para pruebas: pi-ratings (goles y tiros a puerta) y combinador (stacking).

## 1. Cómo predice el modelo

**Dixon-Coles** (60% del resultado final)
- Cada equipo tiene "ataque" y "defensa". Goles esperados del local = ventaja de local × ataque local × debilidad defensiva del visitante (y al revés para el visitante).
- Con los goles esperados se arma la tabla de marcadores posibles (0-0, 1-0, 2-1…) y se suman las casillas de "gana local", "empate" y "gana visita".
- Corrige que 0-0, 1-0, 0-1 y 1-1 pasan un poco distinto de lo que dice la fórmula.
- Usa 3 años de partidos; uno de hace 1 año pesa la mitad que uno de esta semana.

**Elo + regresión logística** (40%)
- Todos empiezan en 1500; el que gana sube y el que pierde baja (más si fue sorpresa o goleada).
- Al empezar una temporada, cada equipo regresa 1/3 hacia 1500 (fichajes, cambios).
- Una regresión logística aprende cómo una diferencia de Elo se traduce en P(local), P(empate), P(visita).

**Lo que el modelo NO sabe:** lesiones, rotaciones, clima, motivación, cambio de técnico.
Por eso el paso 4 mira si Pinnacle (que sí lo sabe) se movió en contra.

## 2. El embudo, paso a paso

Configuración: `apuestas-sistema/config/estrategia.yaml` (sección `embudo`).

| Paso | Pregunta | Si falla | Valor por defecto |
|---|---|---|---|
| 1. Calidad de datos | ¿Ambos equipos tienen suficientes partidos en el modelo? | `pocos_datos` | ≥ 10 partidos |
| 2. Probabilidad | ¿El modelo conoce a ambos equipos? | `sin_prediccion` | — |
| 3. Valor | ¿`p_modelo × cuota − 1` supera el mínimo? | `sin_valor` | 5% sáb/dom · 8% lun–vie |
| 4a. Rango | ¿La cuota está en el rango? | `cuota_fuera_rango` | 1.50 – 3.50 |
| 4b. Sospecha | ¿El modelo difiere del mercado más de lo creíble? | `ventaja_sospechosa` | > 15 puntos |
| 4c. Mercado | ¿Pinnacle bajó la probabilidad de nuestra selección desde que la vimos? | `mercado_en_contra` | > 3 puntos |
| 5. Monto | Kelly × 25%, con tope | — | tope 2% del fondo |

Después, límites globales: máximo 3 alertas por día, máximo 10% del fondo expuesto por semana
y pausa total si en el mes se perdió 15% del fondo.

### Ejemplo

Arsenal (local) vs Chelsea, sábado. Apuesta Total paga 2.10 / 3.40 / 3.60.
- Margen de la casa: 1/2.10 + 1/3.40 + 1/3.60 = 1.048 → 4.8%. Sin margen, el mercado da 45.4% al local.
- El modelo da 52%. EV = 0.52 × 2.10 − 1 = **+9.2%** → pasa el paso 3 (mínimo 5%).
- Cuota 2.10 está entre 1.50 y 3.50; diferencia con Pinnacle ≈ 6 puntos (< 15); Pinnacle no se movió en contra → pasa el paso 4.
- Kelly: (0.52 × 2.10 − 1) / 1.10 = 8.4% → × 25% = 2.1% → tope 2% → con fondo de S/ 500 = **S/ 10**.
- La alerta dice además "solo si la cuota sigue en 2.02 o más" (por debajo, el EV cae del 5%).

## 3. Cómo leer un backtest

```bash
apuestas backtest --liga E0
```

Genera `datos/backtest/backtest_E0.md`. Qué mirar, en orden:

1. **Log loss modelo vs mercado.** Lo normal es que el modelo sea *un poco peor* que el cierre de Pinnacle. Si es mucho peor (diferencia > 0.03), el modelo está flojo para esa liga.
2. **CLV medio.** Positivo = las cuotas elegidas eran mejores que el cierre. Es la mejor señal de ventaja real. Negativo = mala señal aunque el ROI salga positivo.
3. **ROI con su margen de error.** Con 100 apuestas a cuota ~2.5, el ROI tiene un error típico de ±10–15%. Un ROI de +8% con 100 apuestas **no prueba nada**.
4. **Validación.** Los umbrales se eligen con las temporadas de calibración y se prueban en la última, que no se usó para elegir. Si en validación se derrumba, los umbrales "ganadores" eran suerte.

**Advertencia honesta:** las casas grandes son muy eficientes. Es posible que el backtest muestre
que no hay ventaja en las ligas principales. Ese también es un resultado útil: evita perder dinero.
Donde más chance hay es en ligas menos seguidas y en casas "blandas" que se atrasan respecto a Pinnacle.
