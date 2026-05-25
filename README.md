# Cuartinhos AMD
### Modelo de Trading Algorítmico — BTCUSDT · Ciclo de Sesiones 6h · Patrón AMD

[![Edge](https://img.shields.io/badge/Veredicto-Edge%20Real%20Sólido-brightgreen)](https://joshua-barrientos.github.io/Cuartinhos-AMD/)
[![WR](https://img.shields.io/badge/Win%20Rate-63.8%25-blue)](https://joshua-barrientos.github.io/Cuartinhos-AMD/)
[![PF](https://img.shields.io/badge/Profit%20Factor-1.47-blue)](https://joshua-barrientos.github.io/Cuartinhos-AMD/)
[![Sharpe OOS](https://img.shields.io/badge/Sharpe%20OOS-1.19-blue)](https://joshua-barrientos.github.io/Cuartinhos-AMD/)

**→ [Ver Dashboard Completo](https://joshua-barrientos.github.io/Cuartinhos-AMD/)**

---

## Descripción

Cuartinhos AMD es un modelo de trading sistemático basado en reglas, construido sobre el framework de estructura de mercado **AMD (Acumulación, Manipulación, Distribución)**. La idea central: dividir el día de trading en cuatro sesiones de 6 horas alineadas con las principales sesiones de mercado globales, clasificar cada sesión según su fase de comportamiento, y entrar solo cuando varios filtros de confirmación en múltiples timeframes se alinean simultáneamente.

El modelo fue desarrollado a partir de experiencia en trading discrecional y formalizado mediante un proceso de backtesting sistemático usando Python y datos de la API de Binance.

---

## Cómo Funciona

El ciclo diario se divide en cuatro cuartos de 6 horas (UTC):

| Cuarto | Hora UTC | Sesión |
|--------|----------|--------|
| Q1 | 18:00 – 00:00 | Asian |
| Q2 | 00:00 – 06:00 | London |
| Q3 | 06:00 – 12:00 | NY AM |
| Q4 | 12:00 – 18:00 | NY PM *(descartada — sin edge consistente)* |

Cada sesión se clasifica como una de las siguientes fases:

- **ACUMULACIÓN** → rango comprimido respecto al ATR14 rolling — el mercado está acumulando energía
- **MANIPULACIÓN** → expansión de rango + barre el extremo de la acumulación + rechazo — trampa de smart money
- **DISTRIBUCIÓN** → zona de entrada tras manipulación confirmada — comienza el movimiento direccional

Una operación solo se activa cuando **tres componentes MTF independientes** se alinean:

| Componente | Timeframe | Lógica |
|------------|-----------|--------|
| Filtro de tendencia | 1h | Ratio mínimo de velas alcistas/bajistas alineadas con la dirección |
| Sweep de liquidez | 30m | Pool de swing high/low previo barrido antes de la entrada |
| Vela de rechazo | 5m | Primera vela confirmatoria al inicio del DIST con filtro body/range |

**Score mínimo requerido: 2/4** (ponderado: sweep = 2pts, tendencia = 1pt, rechazo = 1pt)

---

## Parámetros Clave

```python
SYMBOL           = 'BTCUSDT'
CAPITAL_INIT     = 1_000
RISK_PCT         = 0.03          # riesgo flat — siempre 3% del capital inicial
USE_COMPOUNDING  = False         # sin interés compuesto
COMMISSION_RATE  = 0.001         # 0.1% por lado (Binance Futures)
SLIPPAGE_RATE    = 0.0002        # estimación conservadora de slippage

ACCUM_FACTOR     = 1.0           # rango ≤ ATR × factor → ACUMULACIÓN
MANIP_FACTOR     = 0.85          # rango > ATR × factor → candidato MANIPULACIÓN
SWEEP_PCT        = 0.001         # profundidad mínima del sweep

MTF_TREND_BARS   = 8             # ventana lookback (velas 1h)
MTF_SWEEP_BARS   = 30            # ventana lookback (velas 30m)
MTF_REJECT_BARS  = 4             # ventana de búsqueda al inicio del DIST (velas 5m)
MTF_MIN_SCORE    = 2             # score mínimo ponderado para validar el setup
```

---

## Resultados del Backtest

**Período:** 2024-01-01 → 2026-05-12 · **Fuente de datos:** API de Binance  
**Todos los resultados son netos de costos de transacción (entrada + salida)**

| Métrica | Valor |
|---------|-------|
| Trades totales | 141 |
| Win rate | 63.8% |
| Profit factor | 1.47 |
| PnL total | +106.8% ($1,000 → $2,068) |
| Avg win | +$37.0 |
| Avg loss | -$44.3 |
| Expectancy / trade | +$7.6 |
| RR promedio | 1.23 |
| Drawdown máximo | -17.5% |
| Sharpe (OOS) | 1.19 |
| Sortino (OOS) | 1.56 |
| Calmar (OOS) | 1.76 |
| Prob. de ruina (Monte Carlo 2,000 sim.) | 0.8% |

### Walk-Forward Validation (split: 2025-01-01)

| Métrica | In-Sample (2024) | Out-of-Sample (2025–26) |
|---------|-----------------|------------------------|
| Trades | 53 | 88 |
| Win rate | 62.3% | **64.8%** |
| PnL | +$468 | +$600 |
| Sharpe | 1.48 | 1.19 |
| Sortino | 3.93 | 1.56 |
| Calmar | 3.58 | 1.76 |
| Profit factor | 1.53 | 1.43 |
| Drawdown máximo | -13.8% | -23.6% |

> El win rate out-of-sample **supera** al in-sample — un indicador sólido de que el modelo captura estructura de mercado real y no patrones sobreajustados.

### Rendimiento por Sesión

| Sesión | Trades | Win Rate | PnL |
|--------|--------|----------|-----|
| London | 18 | **83.3%** | +$293 |
| NY AM | 25 | 68.0% | +$115 |
| Asian | 98 | 59.2% | +$661 |
| NY PM | — | — | Descartada |

### Análisis Monte Carlo (2,000 simulaciones — iid + Block Bootstrap)

| Métrica | Valor |
|---------|-------|
| Probabilidad de ruina | **0.8%** |
| Capital P5 (peor 5%) | $1,431 |
| Capital mediana | $2,068 |
| Capital P95 (mejor 5%) | $2,643 |
| DD peor caso (block P95) | -10.6% |
| Pérdidas consecutivas P95 | 5 |

---

## Notas Metodológicas

- **La baja frecuencia de trades es por diseño.** 141 trades en 2.5 años refleja la naturaleza selectiva del modelo: las entradas requieren alineación en múltiples timeframes a través de tres componentes independientes. Menos señales, mayor calidad.
- **Gestión de riesgo flat.** Cada trade arriesga exactamente el 3% del capital inicial ($30), independientemente del equity actual. Sin interés compuesto. Esto hace los resultados más conservadores y reproducibles.
- **Costos de transacción incluidos.** Comisión (0.1% por lado) y slippage (0.02%) aplicados a cada operación. Todos los PnL mostrados son netos.
- **Sin lookahead bias.** El split walk-forward y el uso cuidadoso de datos indexados por tiempo garantizan que ninguna información futura se filtre en la generación de señales.
- **NY PM descartada.** Tras pruebas exhaustivas de parámetros, Q4 no mostró edge consistente en ninguna configuración — probablemente debido a la reducida participación institucional y menor estructura de volatilidad en ese horario.

---

## Stack Tecnológico

- **Lenguaje:** Python 3.11
- **Datos:** API de Binance (OHLCV en 1m, 5m, 15m, 30m, 1h, 6h)
- **Librerías:** pandas, numpy, matplotlib
- **Validación:** Walk-forward split, Monte Carlo (iid + Block Bootstrap, 2,000 iteraciones)
- **Asistencia IA:** Claude (Anthropic) — revisión de código y debugging

---

## Disclaimer

Este es un modelo backtestado. El rendimiento pasado no garantiza resultados futuros. Los resultados mostrados reflejan simulación histórica, no trading en vivo. Esto no constituye asesoramiento financiero.

---

*Joshua Barrientos
