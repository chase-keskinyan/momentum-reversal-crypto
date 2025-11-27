# Statistical Arbitrage in Cryptocurrencies  
_Momentum & Reversal Signals on BTC and Altcoins_

## Overview

This project studies simple, interpretable **momentum and reversal signals** in daily cryptocurrency data and evaluates whether they deliver persistent statistical edge after realistic trading costs.

The work is inspired by institutional **stat-arb research**: we test a family of rules on BTC, then examine how they generalize to a small altcoin universe (ETH, SOL, XRP, ADA, LTC, DOGE).

The goal is to show, end-to-end, how a junior quant might:
- Formulate hypotheses,
- Clean and align market data,
- Implement and backtest trading signals with costs,
- Evaluate performance (Sharpe, drawdowns, alpha/beta),
- Stress-test robustness across assets and regimes.

All research and implementation was done as part of the **Wall Street Quants** program.

---

## Data

- **Universe:** BTC, ETH, SOL, XRP, ADA, LTC, DOGE  
- **Frequency:** Daily candles (1d)  
- **Fields:** Open, High, Low, Close, Volume  
- **Sample:** 2023-12-01 to 2025-11-18 (rolling window for some tests)

Data are stored as CSVs:

- `btc_kraken_1d_2023-12-01_to_2025-11-18.csv`
- `ETH_1d_2023-12-01_to_2025-11-18.csv`
- `SOL_1d_2023-12-01_to_2025-11-18.csv`
- `XRP_1d_2023-12-01_to_2025-11-18.csv`
- `ADA_1d_2023-12-01_to_2025-11-18.csv`
- `LTC_1d_2023-12-01_to_2025-11-18.csv`
- `DOGE_1d_2023-12-01_to_2025-11-18.csv`

All timestamps are aligned to a common daily close.

---

## Strategy Hypotheses

I focus on **short-horizon reversal** and **medium-horizon dislocation mean reversion**, using simple, explainable rules.

Examples of tested hypotheses:

- **H4 – 1-Day Crash Reversal**  
  - Enter long after a large 1-day negative return (“crash”) when the move exceeds a volatility-scaled threshold.
  - Motivation: liquidity-driven selloffs that mean-revert once forced selling is done.

- **H5 – Bollinger-Band Reversion**  
  - Go long when price closes below a lower Bollinger band based on a rolling moving average and volatility estimate.
  - Motivation: over-extensions relative to recent trend tend to snap back.

- **H6 – Low-Volume Fade**  
  - Fade large up-moves that occur on unusually low volume.  
  - Motivation: “weak” moves with poor participation have lower follow-through.

Each hypothesis is implemented as a rule that produces a **daily position** (+1 / 0 / -1) and corresponding **strategy return series**.

---

## Methodology

1. **Data Cleaning & Alignment**
   - Load raw CSVs into pandas.
   - Align each asset to a shared daily calendar.
   - Forward-fill missing days where appropriate and drop incomplete initial periods.

2. **Signal Construction**
   - Compute rolling returns and volatility (e.g., 20-day realized vol).
   - Define entry/exit conditions for each hypothesis (H4–H6) using volatility-scaled thresholds.
   - Generate position series per asset and per hypothesis.

3. **Backtesting with Costs**
   - Daily P&L: `position.shift(1) * daily_return`.
   - Transaction cost model:
     - 20 bps per turnover for market orders (commissions + slippage).
   - Apply costs whenever the position changes (|Δposition| > 0).

4. **Performance Evaluation**
   - Cumulative equity curve.
   - Annualized return & volatility.
   - Annualized Sharpe ratio.
   - Max drawdown.
   - Hit rate, average win / loss.
   - For BTC, compare against buy-and-hold.

5. **Multi-Signal Combination**
   - Combine H4, H5, H6 into a **composite strategy**:
     - Equal-weight average of normalized signal returns, and
     - Risk-parity combination (scale each leg to equalize volatility).
   - Evaluate the composite on BTC and then on each altcoin.

---

## Key Results (High-Level)

> Exact numbers may change as I iterate; the point is the process and robustness tests.

- Each of H4, H5, and H6 generates **positive standalone Sharpe** on BTC over the sample, even after 20 bps trading costs.
- A **simple equal-weight combination** of H4 + H5 + H6 on BTC:
  - Improves risk-adjusted returns vs any individual leg.
  - Reduces drawdowns via diversification of signal logic.
- A **risk-parity combination** of the three signals:
  - Further stabilizes the equity curve by down-weighting the most volatile leg.
  - Achieves an annualized Sharpe around **0.8–1.0** on BTC over Dec 2023–Nov 2025, with relatively shallow max drawdowns.
- When the BTC-trained rules are applied to altcoins (ETH, SOL, DOGE, etc.), performance remains positive in several names, suggesting **some cross-asset robustness**, though with higher volatility and dispersion.

Overall, the results are consistent with the idea that **short-horizon dislocation / reversal effects** exist in major crypto assets and can be captured via simple, transparent rules.

---

## Repository Structure

```text
├── Momentum_Reversal_Backtests.ipynb   # Main research notebook
├── btc_kraken_1d_2023-12-01_to_2025-11-18.csv
├── ETH_1d_2023-12-01_to_2025-11-18.csv
├── SOL_1d_2023-12-01_to_2025-11-18.csv
├── XRP_1d_2023-12-01_to_2025-11-18.csv
├── ADA_1d_2023-12-01_to_2025-11-18.csv
├── LTC_1d_2023-12-01_to_2025-11-18.csv
└── DOGE_1d_2023-12-01_to_2025-11-18.csv
```

## How to Run

### Clone
```bash
git clone https://github.com/chasekeskinyan/momentum-reversal-crypto.git
cd momentum-reversal-crypto

### Install Dependencies
```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Launch Notebook
```bash
jupyter lab
```

### Notebook Name
```text
Momentum_Reversal_Backtests.ipynb
```

## Future Work List
```text
- Expand universe to top-20 liquid crypto assets
- Add execution modeling (limit orders, volume-based slippage)
- Explore regime filtering (volatility, funding rates, on-chain metrics)
- Apply statistical validation (White’s Reality Check)
- Modularize signals + backtester into a src/ package
```

## Author Section
```text
Chase Keskinyan
B.A. Economics, University of Michigan (2026)
```

