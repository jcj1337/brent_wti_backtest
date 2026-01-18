# Brent and WTI Signal Backtest + Tableau Dashboard

- Built/tested a backtest model with a couple trading signals on commodities (Brent + WTI crude oil) through Python. Visualizations on a Tableau dashboard
- See [Notebook](commodities_backtest.ipynb) for code/documentation
- See [Dashboard](https://public.tableau.com/app/profile/jet.c7615/viz/Backtest_Tableau_Done/InsightsDashboard#1) for visualizations

## Data:
- **Assets:** Brent spot daily prices, WTI spot daily prices
- **Frequency:** Daily
- **Cleaning:** date parsing, sorting, duplicate removal
- **Backtest window:**
  - Train: 2005-01-04 → 2016-12-31
  - Test: 2017-01-01 → 2025-12-31

## Strategy: 
- Brent and WTI were modeled with *different strategies*. Brent uses an MA-based trend signal with a deadband threshold, while WTI uses a z-score **continuation** signal (inverted mean-reversion sign convention), with thresholds tuned on the training set and evaluated on the test set each. This was due to the fact that it was found only one signal each produced edge (although both were backtested on each)

### 1) Trend (Moving Average + Deadband Delta)
- Compute a moving average (e.g. 50-day)
- Compute a score:  
  `trend_score = Price / MA - 1`
- Convert score to a discrete position with a quantitatively picked deadband (see notebook)
  - Long if `trend_score > delta`    (1)
  - Short if `trend_score < -delta`  (-1)
  - Flat otherwise                   (0)
- `delta` is selected on the training period through a grid search maximizing an annual Sharpe ratio with a penalization on turnovers

### 2) Regime Continuation (Z-score Breakout)
Instead of fading extremes (mean reversion), this signal assumes that large deviations (e.g. a black swan event) from the mean can indicate a new regime. 
- Compute log-price and a rolling z-score
- Enter positions when `|z|` exceeds threshold `a`  
  (`a` is chosen similar to delta, through a grid-search to maximize a function, s) 
   - **Long** if \( z_t > a \)
   - **Short** if \( z_t < -a \)
   - **Flat** otherwise

## Backtest Setup
- Positions are shifted by 1 day to avoid lookahead 
- Transaction costs modeled as bps per position change
- Uses volatility targetting as main safeguard to Black Swan events (can adjust scale for a target vol)
- Backtest on both Brent and WTI 

## Metrics
- Total return, annualized return, annualized volatility
- Sharpe 
- Max drawdown
- Turnover

## Portfolio Construction 
- Initialized an even 50/50 baseline
- Used inverse-volatility weights to find a 44/56 split (Brent, WTI)

## Tableau Dashboard
The exported CSV (commodities_tableau.csv) is used to build:
- Equity curves over time (i.e. compounded growth of a dollar invested in each over time)
- Brent price along with position over time
- WTI price along with position over time
- Portfolio drawdown over time

## Results

| Strategy | Total Return | Ann Return | Ann Vol | Sharpe | Max Drawdown |
|---|---:|---:|---:|---:|---:|
| **Brent** | 66.62% | 5.80% | 10.12% | 0.57 | -20.68% |
| **WTI** | 20.05% | 2.07% | 8.02% | 0.26 | -21.27% |
| **Portfolio (50/50)** | 33.69% | 3.35% | 7.66% | 0.437 | -21.48% |




