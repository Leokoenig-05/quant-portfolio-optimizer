# Mean-Variance Optimization vs. 1/N: An Out-of-Sample Study

Does a mean-variance optimized portfolio beat naive equal weighting when expected returns and covariances are estimated from history? This project tests that question on eight liquid ETFs, with and without covariance shrinkage.

## Summary

- Out-of-sample (2012–2026), the max-Sharpe portfolio has a higher point-estimate Sharpe ratio than 1/N (0.85 vs 0.66). A block bootstrap cannot distinguish this from zero (95% CI of the difference: [-0.16, 0.59]).
- Minimum-variance portfolios match the Sharpe ratio of 1/N at roughly half the volatility, with a moderately smaller maximum drawdown (-17% vs -21%).
- Ledoit-Wolf shrinkage changes Sharpe and turnover only marginally.
- The advantage of max-Sharpe varies with the estimation window (+0.06 to +0.25 Sharpe) and is largely driven by regime-specific allocations (bonds until 2021, gold and commodities afterwards).

My initial hypothesis was that Markowitz would underperform 1/N. The data neither confirm nor refute it.

## Data

Daily adjusted close prices from Yahoo Finance (`yfinance`), January 2007 to October 2026: SPY, EFA, EEM (equities), TLT, IEF (government bonds), GLD (gold), DBC (commodities), VNQ (real estate). Returns are simple daily returns.

## Method

Long-only portfolios, rebalanced at each month-end using only data up to that day (no look-ahead), held for the following month. Default estimation window: 1260 trading days (5 years).

| Strategy | Weights |
| --- | --- |
| 1/N | equal weights |
| Min-variance | minimize w'Σw s.t. 1'w = 1, w ≥ 0 |
| Max-Sharpe | convex reformulation: minimize y'Σy s.t. (μ − r_f)'y = 1, y ≥ 0, w = y/1'y |
| (LW) variants | Σ replaced by the Ledoit-Wolf shrinkage estimator |

Optimization is solved with `cvxpy`. The risk-free rate is set to zero. Significance of Sharpe differences is assessed with a moving-block bootstrap (block length 21 days, 5000 resamples), a simplified version of the approach in Ledoit and Wolf (2008), without studentization.

## Pre-specified expectations and outcomes

| Expectation (written before running) | Outcome |
| --- | --- |
| Max-Sharpe has much higher turnover and less stable weights than min-variance | Confirmed (13.6% vs 1.0% monthly turnover) |
| Min-variance is more robust | Partly: half the volatility, same Sharpe as 1/N, drawdown only moderately smaller |
| Ledoit-Wolf stabilizes but does not necessarily improve Sharpe | Small effect |
| Max-Sharpe does not clearly beat 1/N | Point estimate favors max-Sharpe, bootstrap CI includes zero |

## Results

Out-of-sample 2012–2026, 5-year window:

| Strategy | Return p.a. | Vol | Sharpe | Max drawdown | Monthly turnover |
| --- | --- | --- | --- | --- | --- |
| 1/N | 6.1% | 9.6% | 0.66 | -20.7% | n/a |
| Min-variance | 3.4% | 5.2% | 0.66 | -17.3% | 0.010 |
| Min-variance (LW) | 3.5% | 5.3% | 0.69 | -16.8% | 0.010 |
| Max-Sharpe | 7.4% | 8.9% | 0.85 | -20.1% | 0.136 |
| Max-Sharpe (LW) | 7.8% | 9.1% | 0.87 | -20.8% | 0.124 |
| SPY buy & hold | 14.7% | 16.5% | 0.91 | -33.7% | n/a |

Bootstrap of the Sharpe difference to 1/N:

| Strategy | Difference | 95% CI |
| --- | --- | --- |
| Max-Sharpe | +0.196 | [-0.156, 0.585] |
| Max-Sharpe (LW) | +0.210 | [-0.135, 0.591] |
| Min-variance | -0.002 | [-0.383, 0.351] |
| Min-variance (LW) | +0.028 | [-0.336, 0.365] |

Robustness to the estimation window (common sample from February 2014, point estimates only):

| Sharpe | 3 years | 5 years | 7 years |
| --- | --- | --- | --- |
| 1/N | 0.71 | 0.71 | 0.71 |
| Min-variance (LW) | 0.71 | 0.72 | 0.69 |
| Max-Sharpe | 0.77 | 0.94 | 0.94 |
| Max-Sharpe (LW) | 0.81 | 0.96 | 0.93 |

Figures (efficient frontier, cumulative wealth, max-Sharpe weights over time) are in the notebook.

## Limitations

- **Universe chosen with hindsight.** SPY, GLD and IEF performed well over the sample; the ETF selection is not blind.
- **Single historical path** of about 14.7 years. Regime effects (falling rates until 2021, rate shock in 2022, gold rally) dominate.
- **Zero risk-free rate** overstates Sharpe ratios, especially in 2007–2008 and 2023–2026.
- **No transaction costs.** At 10 bps per unit of turnover the max-Sharpe cost is roughly 0.16% p.a., which is small but not modelled.
- **Turnover** ignores weight drift within the month; 1/N is therefore reported as n/a.
- **Means** are arithmetic (daily mean × 252), the right input for mean-variance but above the realized geometric growth rate.
- **Shrinkage** is applied to Σ only. The larger source of estimation error, μ, is not shrunk.
- **Bootstrap** is simplified (not studentized), block length is not tuned, and four comparisons are made without multiple-testing correction.
- Robustness windows are reported as point estimates only.

## Reproduce

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter lab   # open notebooks/01_portfolio_optimizer.ipynb, Run All
```

Data are downloaded automatically. Results may differ slightly if Yahoo Finance revises historical data.

## References

- Markowitz (1952), Portfolio Selection, Journal of Finance
- DeMiguel, Garlappi, Uppal (2009), Optimal Versus Naive Diversification, Review of Financial Studies
- Ledoit, Wolf (2004), A well-conditioned estimator for large-dimensional covariance matrices, Journal of Multivariate Analysis
- Ledoit, Wolf (2008), Robust performance hypothesis testing with the Sharpe ratio, Journal of Empirical Finance