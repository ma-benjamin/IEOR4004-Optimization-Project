# Risk-Aware Allocation to Large-Cap, High-Momentum Stocks

IEOR 4004 — Optimization Models and Methods — Team 14

**Team members:** Benjamin Ma, Grégoire Buot, Haixin Shen, Xuanyu Wang, Jiaxuan Yin

## Summary of the project

An investment manager allocates capital among large-cap U.S. stocks with strong momentum for **January 2026**. The portfolio weights are chosen to **minimize the monthly 95% Conditional Value-at-Risk (CVaR)** under a parametric Normal model, subject to:

- full investment,
- no short selling,
- at most 20% of the budget invested in any single stock,
- a minimum expected monthly return equal to the historical mean monthly S&P 500 return.

Under the joint-normality assumption the problem is a convex **second-order cone program (SOCP)**, solved with Gurobi.

## Candidate stocks

Starting from U.S. common stocks in CRSP (via WRDS), each stock is ranked independently by:

- **Size**: market capitalization on the formation date (last trading day of December 2025).
- **Momentum**: cumulative return from January to November 2025 (12–2 convention; December excluded to limit short-term reversal).

Each ranking is split into 15 groups. The candidate set is the intersection of the largest-size group and the highest-momentum group, which yields **37 stocks**.

## Model

```
minimize    − μᵀw + kα · s

subject to  Σᵢ wᵢ = 1                 (full investment)
            0 ≤ wᵢ ≤ 0.20             (no short selling, 20% cap)
            μᵀw ≥ b                   (beat the S&P 500 mean return)
            ‖A w‖₂ = s,  s ≥ 0        (s bounds the portfolio std. dev.)
```

| Symbol | Meaning |
|---|---|
| `w` | Portfolio weights (decision variable), one per candidate stock |
| `s` | Upper bound on the portfolio monthly standard deviation (decision variable) |
| `μ` | Sample mean monthly return of each stock, Jan 2021 – Dec 2025 (60 months) |
| `Σ` | Sample covariance matrix of monthly returns over the same window |
| `A` | Matrix such that `AᵀA = Σ`, so that `‖Aw‖₂ = √(wᵀΣw)` |
| `kα` | `φ(Φ⁻¹(α)) / (1 − α) ≈ 2.0627` for α = 0.95 (Normal CVaR constant) |
| `b` | Mean monthly S&P 500 total return over the same window |

The objective `−μᵀw + kα·√(wᵀΣw)` is the 95% Normal CVaR of the monthly portfolio loss.

## Repository contents

| File | Description |
|---|---|
| `Milestone4_notebook.ipynb` | Builds and solves the CVaR SOCP with Gurobi - latest version. |
| `modified_milestone4.ipynb` | Builds and solves the CVaR SOCP with Gurobi - version2. |
| `model implementation.ipynb` | Builds and solves the CVaR SOCP with Gurobi - version1. |
| `selected_monthly_return_updated.csv` | Monthly returns (Jan 2021 – Mar 2026) of the 37 candidate stocks, with `PERMNO`, `Ticker` and `average_monthly_return` (μ). |
| `covariance_matrix.csv` | 37 × 37 sample covariance matrix of monthly returns (60 observations, ddof = 1, not annualized). The first line is a header note and is skipped when loading. |

All returns are monthly simple total returns in decimals.


## Data source

CRSP U.S. Stock Databases, accessed through WRDS (<https://wrds-www.wharton.upenn.edu/>). Raw CRSP data is not redistributed in this repository; only the derived monthly returns and covariance matrix are included.

## References

- Fama, E. F., and French, K. R. (1993). Common Risk Factors in the Returns on Stocks and Bonds. *Journal of Financial Economics*, 33(1), 3–56.
- Jegadeesh, N., and Titman, S. (1993). Returns to Buying Winners and Selling Losers. *The Journal of Finance*, 48(1), 65–91.
- Rockafellar, R. T., and Uryasev, S. (2000). Optimization of Conditional Value-at-Risk. *The Journal of Risk*, 2(3), 21–41.
