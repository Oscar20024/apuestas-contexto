# Técnicas y herramientas: catálogo completo

Todo lo que el sistema usa, lo que se probó y lo que se investigó y se dejó de lado, con su estado,
dónde está en el código y por qué. Estado al **2026-11-22, v1.4.0**.

Leyenda: ✅ en uso · 🧪 programado, disponible para pruebas, no decide alertas · ⏸️ investigado, no usado · ❌ descartado

---

## 1. Cómo se decide una alerta (en uso)

| Técnica | Estado | Qué hace | Dónde | Parámetros |
|---|---|---|---|---|
| **Precio justo desde el mercado sharp** | ✅ | La probabilidad "real" de cada resultado es la que implican Pinnacle y Betfair Exchange, sin su margen | `decision/embudo.py` (paso 2) | `embudo.fuente_probabilidad: mercado` |
| **Quitar el margen: método potencia** | ✅ | Convierte cuotas en probabilidades que suman 100 %. El método potencia corrige un poco el sesgo favorito-sorpresa; en el backtest salió algo mejor que el proporcional en las 3 ligas | `decision/margen.py` | `embudo.metodo_margen: potencia` |
| **Consenso de referencias** | ✅ | Precio justo = promedio de Pinnacle y Betfair. Si las dos tienen el partido, **las dos** deben ver la ventaja mínima | `decision/embudo.py` (paso 4) | `exigir_consenso_referencias: true`, `oddspapi.casas_referencia_extra: [betfair-ex]` |
| **Referencia extra bajo demanda** | ✅ | Betfair solo se pide para las ligas donde Pinnacle ya ve una alerta posible; si falla, sigue Pinnacle sola | `servicios/escanear.py` (`_torneos_con_alerta_posible`) | v1.4.1 |
| **Valor esperado (EV)** | ✅ | `EV = p_justa × cuota_Apuesta_Total − 1`. Se elige la selección con mayor EV entre local, empate y visita | `decision/margen.py`, `embudo.py` (paso 3) | 2 % sáb/dom, 3 % lun–vie |
| **Rango de cuota** | ✅ | Evita favoritos muy cortos y sorpresas muy largas (donde el precio justo es menos preciso) | paso 4 | 1,50 – 4,00 |
| **Diferencia sospechosa** | ✅ | Si Apuesta Total y la referencia difieren más de 10 puntos, suele ser cuota vieja, mercado equivocado o equipos cruzados | paso 4 | `diferencia_maxima_vs_pinnacle: 0.10` |
| **Frescura de la referencia** | ✅ | Cada precio de OddsPapi trae `changedAt`. Si la cuota de Pinnacle no cambia hace más de 12 h, no se confía en ella | paso 4, `fuentes/oddspapi.py` | `antiguedad_maxima_referencia_horas: 12` |
| **Movimiento del mercado** | ✅ | Si Pinnacle bajó la probabilidad de la selección más de 3 puntos desde la primera vez que se vio, se descarta | paso 4 | `movimiento_maximo_en_contra: 0.03` |
| **Kelly fraccional** | ✅ | Monto = 25 % de Kelly, con tope 2 % del fondo y mínimo de la casa | `decision/kelly.py` | `fraccion_kelly: 0.25`, `tope_por_apuesta: 0.02` |
| **Cuota mínima para apostar** | ✅ | La alerta dice debajo de qué cuota ya no hay valor | `margen.cuota_minima_para` | — |
| **Límites globales** | ✅ | 3 alertas/día, 10 % del fondo por semana, pausa si se pierde 15 % en el mes | `servicios/escanear.py` | sección `limites` |
| **Sin duplicados y caducidad** | ✅ | Una alerta por partido y selección; caduca al empezar el partido | `db.ya_alertado`, `mensajes_salida.caduca_en` | — |
| **Modo prueba** | ✅ | Las alertas dicen "no apostar" mientras se mide el CLV | `estrategia.yaml` | `modo_prueba: true` |

## 2. Cómo se mide si funciona

| Técnica | Estado | Qué mide | Dónde |
|---|---|---|---|
| **CLV (closing line value)** | ✅ | Cuota tomada × probabilidad justa al cierre de Pinnacle − 1. Positivo de forma sostenida = ventaja real, aunque se pierdan apuestas | `v_rendimiento_alertas`, resumen del lunes, backtest |
| **Backtest walk-forward** | ✅ | Reentrena semana a semana usando solo lo anterior; nunca mira el futuro | `backtest/motor.py` |
| **Comparación de variantes** | ✅ | Mismas predicciones, distintas formas de combinar; se evalúan en las mismas temporadas | `backtest/comparar.py`, `apuestas comparar` |
| **Log-loss** | ✅ | Calidad de las probabilidades (menor es mejor). Métrica principal para elegir modelos | `motor._log_loss` |
| **RPS (ranked probability score)** | ✅ | Como log-loss pero castiga más confundir local con visita que con empate | `comparar.rps` |
| **ECE (error de calibración)** | ✅ | Si el modelo dice 30 %, ¿pasa ~30 % de las veces? | `comparar.error_calibracion` |
| **Brier** | ✅ | Error cuadrático de las probabilidades | `motor._brier` |
| **Error típico del ROI** | ✅ | Con pocas apuestas el ROI es casi azar; se reporta su margen | `motor.metricas` |
| **Búsqueda de umbrales con validación** | ✅ | Elige umbrales en temporadas de calibración y los prueba en la última, que no se usó | `motor.buscar_umbrales` |
| **Elegir por calibración, no por aciertos** | ✅ (criterio) | Walsh y Joshi (2024): seleccionar modelos por calibración da mucho más retorno que por accuracy | criterio documentado |

## 3. Modelos propios (informativos desde ADR 0011)

| Técnica | Estado | Qué hace | Dónde | Resultado en el backtest |
|---|---|---|---|---|
| **Dixon-Coles** | ✅ informativo | Ataque y defensa por equipo, ventaja de local, corrección de marcadores bajos, decaimiento (vida media 365 días), ventana 3 años; gradiente analítico + L-BFGS-B | `modelos/dixon_coles.py` | Log-loss peor que Pinnacle |
| **Elo + regresión logística** | ✅ informativo | Rating 1500 inicial, K=20, multiplicador por goles, regresión 1/3 por temporada; la logística pasa la diferencia a 3 probabilidades | `modelos/elo.py` | Peor que Pinnacle |
| **Mezcla fija 0,6 DC + 0,4 Elo** | ✅ informativo | Es lo que aparece como "Nuestro modelo" en la alerta | `modelos/ensamble.py` | ROI −5 % a −13 %, CLV −6 % |
| **Pi-ratings** (Constantinou y Fenton, 2013) | 🧪 | Rating de local y de visita por separado, actualizado con el error en la diferencia de goles (λ=0,035, γ=0,7) | `modelos/pi_ratings.py` | No mejoró (`usar_pi: false`) |
| **Pi-ratings con tiros a puerta** | 🧪 | La misma técnica con `HST/AST` en vez de goles: un "xG casero" | `pi_ratings.py` + columnas `tiros_*` | No mejoró (`usar_pi_tiros: false`) |
| **Combinador / stacking** | 🧪 | Regresión logística multinomial sobre log(P_local/P_empate) y log(P_visita/P_empate) de cada modelo (+ mercado), entrenada walk-forward | `modelos/combinador.py` | No superó a Pinnacle solo |
| **Mezcla modelo + mercado** | 🧪 | Combinar nuestro modelo con Pinnacle | `comparar.py` | Cualquier peso al modelo empeora la predicción |

## 4. Datos y servicios

| Herramienta | Estado | Para qué | Límite / detalle |
|---|---|---|---|
| **football-data.co.uk** | ✅ | Historial: resultados, tiros, cuotas previas y de cierre (Bet365, Pinnacle, promedio) | Sin límite práctico. Pinnacle previa dejó de venir a mitad de 2025/26 |
| **OddsPapi v4** (`odds-by-tournaments`) | ✅ | Cuotas en vivo de Apuesta Total, Pinnacle y Betfair Exchange | 250 consultas/mes gratis (reserva 20). **1 casa por consulta** (la lista dio 400); hasta 5 ligas por consulta salvo `betfair-ex` (1). Precio con `changedAt` |
| **OddsPapi `/bookmakers`, `/participants`, `/tournaments`** | ✅ | Lista de casas, nombres de equipos (caché 30 días), IDs de ligas | 1 consulta cada uno |
| **Betfair Exchange** (`betfair-ex`) | ✅ | Segunda referencia del precio justo | 19/21 partidos de Premier, cuotas de < 1 h, margen ~1 %. **1 liga por consulta**; se pide solo si Pinnacle ve alerta posible (v1.4.1) |
| **Pinnacle** (`pinnacle`) | ✅ | Referencia principal del precio justo y del CLV | — |
| **Apuesta Total** (`apuestatotal`) | ✅ | Donde apuestas; la casa "blanda" que se compara | Margen ~6 % |
| **Supabase (Postgres 17)** | ✅ | Partidos, cuotas, modelos, evaluaciones, bandeja, apuestas, uso de API | Plan gratis 500 MB (usa ~22 MB) |
| **OpenClaw + WhatsApp** | ✅ | Entrega de mensajes con `docker exec ... message send` | Número en `allowFrom` con `+51` |
| **systemd timers** | ✅ | Programador con hora de Lima y `Persistent=true` | 6 timers |
| **Docker** | ✅ | API aislada (UID 10001, solo lectura, 768 MB, 0,5 CPU, 127.0.0.1:8010) | — |
| sbobet | ❌ | Probada como referencia | 11/21 partidos, hasta 37 h sin cambios, margen ~10 % |
| singbet | ❌ | Probada como referencia | Sin cuotas en nuestro plan |
| ps3838, pin88, pinnacle+5/+30 | ❌ | Son copias de Pinnacle: no aportan una opinión independiente | — |
| FBref | ❌ | Fuente de xG | Perdió los datos avanzados de Opta el 20-01-2026 |
| Understat (vía `soccerdata`) | ⏸️ | xG por partido de Premier, LaLiga y Serie A desde 2014/15 | Pausado: el modelo propio no decide alertas |
| API-Football | ⏸️ | Bajas y alineaciones | 100 consultas/día gratis. Pausado: Pinnacle ya refleja las noticias |

## 5. Librerías y repositorios investigados

| Nombre | Estado | Nota |
|---|---|---|
| pandas, numpy, scipy, scikit-learn | ✅ | Datos, optimización (L-BFGS-B), regresiones logísticas |
| FastAPI, pydantic, httpx, psycopg 3, PyYAML | ✅ | API, configuración validada, HTTP, Postgres |
| pytest | ✅ | 69 pruebas (incluye base de datos local) |
| **penaltyblog** (Martin Eastwood) | ⏸️ | Poisson, Bivariate, Dixon-Coles, Elo, Massey, Colley, Pi-ratings, de-vig. Se programaron versiones propias de lo necesario (pi-ratings) para no sumar dependencias compiladas |
| **soccerdata** | ⏸️ | Scrapers de ClubElo, ESPN, FBref, football-data, Sofascore, Understat, WhoScored |
| CatBoost / XGBoost sobre ratings | ⏸️ | Ganadores del 2017 Soccer Prediction Challenge. No se probó: el problema no es el modelo, es que el mercado sharp ya sabe más |
| BeatTheBookie (Kaunitz et al., 2017) | ✅ (idea) | Origen de la estrategia: apostar donde una casa paga más que el consenso. Advierte que las casas limitan cuentas ganadoras |
| Skills de Claude: `machina-sports/sports-skills` (`betting`), `sports-betting-analyzer` | ❌ | Hacen de-vig, Kelly y edge, que el sistema ya tiene. No entrenan modelos |
| Redes neuronales / LLM que opinen del partido | ❌ | No mejoran la calibración con estos datos; además una IA podría alterar cifras (ADR 0006) |

## 6. Ideas pendientes (no hechas)

| Idea | Por qué | Estado |
|---|---|---|
| Resumen de los lunes con "cuántas veces Apuesta Total pagó más que Pinnacle", aunque no haya alertas | Saber en 1–2 semanas si la estrategia tiene futuro | Propuesta (mejora 5) |
| Registrar apuestas respondiendo por WhatsApp | Sin registro no funciona el límite de pérdida | Pendiente |
| Más ligas (Bundesliga, Ligue 1, Liga 1 de Perú) | Más oportunidades | Limitado por las 250 consultas/mes |
| Descartes por escenario (inicio/fin de temporada, copas, derbis) | Pausado: con el modelo perdiendo en todos lados, segmentar sería buscar suerte | Pausado |
