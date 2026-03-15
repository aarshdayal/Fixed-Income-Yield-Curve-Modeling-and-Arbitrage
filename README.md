# Fixed Income Yield Curve Modeling and Arbitrage

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A complete implementation of yield curve modeling using the Nelson‑Siegel model and a butterfly arbitrage strategy on US Treasury yields. This project demonstrates how to:

- Fetch constant maturity Treasury (CMT) rates from FRED.
- Fit the Nelson‑Siegel model to observed yields.
- Extract forward rates and curve signals.
- Backtest a butterfly trade (2s5s10s) using a mean‑reversion signal based on z‑score.
- Evaluate performance and visualise results including the z‑score entry/exit thresholds.

---

## Overview

The yield curve is a fundamental tool in fixed income markets. This project builds a daily yield curve from 2000 to 2023 using CMT rates for maturities 1y to 30y. The Nelson‑Siegel model is fitted each day to obtain smooth curves and interpretable parameters (level, slope, curvature). A butterfly trade – betting on the curvature of the curve – is implemented and backtested with a simple z‑score signal.

**Key results**:
- **Total P&L**: $441
- **Annualised Return**: 0.18%
- **Annualised Volatility**: 0.18%
- **Sharpe Ratio**: 0.99

These figures assume a fixed capital of $10,000 and a simplified P&L calculation (dollars per 1% change in butterfly spread). The positive Sharpe indicates that the strategy captured mean‑reversion in the butterfly spread.

---

## Data Sources

- **Treasury yields**: Constant Maturity Treasury (CMT) rates from FRED (series: DGS1, DGS2, DGS3, DGS5, DGS7, DGS10, DGS20, DGS30).  
  Retrieved via `pandas_datareader` for the period 2000–2023.

---

## Methodology

### 1. Yield Curve Data
Daily yields for 1, 2, 3, 5, 7, 10, 20, and 30 years are downloaded and aligned. Missing values are dropped. For simplicity, these par yields are treated as spot rates (a common approximation).

### 2. Nelson‑Siegel Model
The Nelson‑Siegel function is defined as:

\[
y(m) = \beta_0 + \beta_1 \frac{1 - e^{-m/\tau}}{m/\tau} + \beta_2 \left( \frac{1 - e^{-m/\tau}}{m/\tau} - e^{-m/\tau} \right)
\]

where:
- \(\beta_0\) = long‑term level,
- \(\beta_1\) = short‑term slope,
- \(\beta_2\) = medium‑term curvature,
- \(\tau\) = decay parameter.

Each day, the model is fitted to the observed yields using `scipy.optimize.curve_fit`. The parameters are stored for further analysis.

### 3. Butterfly Trade Signal
The butterfly spread is defined as:

\[
\text{butterfly} = y_{5y} - \frac{y_{2y} + y_{10y}}{2}
\]

A rolling 60‑day window is used to compute the mean and standard deviation of the spread, yielding a z‑score. Entry signals are generated when the z‑score exceeds a threshold of ±1.0:
- **z > 1.0** → sell butterfly (expect spread to decrease)
- **z < -1.0** → buy butterfly (expect spread to increase)

### 4. Backtest
- **Capital**: $10,000 per trade.
- **P&L**: For each day the position is held, P&L = signal × Δspread × factor, where factor = $100 per 1% change in spread.
- **Returns** are computed relative to the capital and annualised.
- The signal is shifted by one day to avoid look‑ahead bias.

---

## Results

### Yield Curves Over Time
The plot below shows selected yield curves from the sample period, illustrating how the shape evolved (normal, flat, inverted).

### Nelson‑Siegel Parameters
The evolution of the Nelson‑Siegel parameters reveals changing market conditions. \(\beta_0\) (level) generally declined from the early 2000s to near zero post‑2008, then rose again. \(\beta_1\) (slope) and \(\beta_2\) (curvature) capture shorter‑term dynamics.

### Butterfly Trade with Z‑Score Signals
The three‑panel chart displays the butterfly spread, its z‑score with entry thresholds (±1.0), and the resulting cumulative P&L. This visualisation makes the strategy's logic transparent.

### Performance Summary
| Metric                 | Value  |
|------------------------|--------|
| Total P&L              | $441   |
| Annualised Return      | 0.18%  |
| Annualised Volatility  | 0.18%  |
| Sharpe Ratio           | 0.99   |

The modest but positive Sharpe indicates that the butterfly strategy captured mean‑reversion opportunities, though with low volatility. The absolute return is small due to the conservative scaling factor; in practice, leverage could be applied.
