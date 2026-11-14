 Risk-Aware Allocation to Large-Cap, High-Momentum Stocks

IEOR 4004 — Optimization Models and Methods — Team 14

**Team members:** Benjamin Ma, Grégoire Buot, Haixin Shen, Xuanyu Wang, Jiaxuan Yin

## Project Overview

We construct a portfolio of large-cap U.S. stocks with strong momentum for January 2026. Our objective is to minimize monthly 95% Conditional Value-at-Risk (CVaR) under a parametric Normal model.

The optimization is subject to:
- Full investment of available capital.
- No short selling.
- A minimum allocation of 1% and maximum of 20% for each selected stock.
- A minimum expected monthly return equal to the historical mean monthly S&P 500 return.
- A correlation-based diversification rule that prevents simultaneously holding two stocks with estimated correlation above 0.80.

With binary holding decisions, the model is a mixed-integer second-order cone program (MISOCP), solved using Gurobi.

## Candidate Stocks

Stocks are screened using CRSP data accessed through WRDS.

- **Size:** Market capitalization at the end of December 2025.
- **Momentum:** Cumulative return from January to November 2025, following the 12–2 momentum convention.

Stocks are independently ranked into 15 groups by each factor. The intersection of the highest-size and highest-momentum groups produces 37 candidate stocks.

## Optimization Model

The objective is to minimize:

    -μᵀw + kα ‖Aw‖₂

Subject to:

    Σᵢ wᵢ = 1
    0.01zᵢ ≤ wᵢ ≤ 0.20zᵢ
    μᵀw ≥ b
    zᵢ + zⱼ ≤ 1, for (i,j) ∈ P
    zᵢ ∈ {0,1}

where:

| Symbol | Definition |
|---|---|
| wᵢ | Portfolio weight of stock i |
| zᵢ | Binary variable indicating whether stock i is selected |
| μ | Estimated monthly stock returns, January 2021–December 2025 |
| Σ | Sample covariance matrix over the same 60-month window |
| A | Covariance factor satisfying AᵀA = Σ |
| kα | Normal CVaR coefficient, approximately 2.062713 at α = 0.95 |
| b | Historical mean monthly S&P 500 return |
| P | Stock pairs whose estimated return correlation exceeds 0.80 |

The correlation restriction is intended to provide an additional diversification rule beyond the covariance-based CVaR objective.

For computational implementation, auxiliary variables are used to represent the portfolio standard deviation in Gurobi.

## Repository Contents

| File | Description |
|---|---|
| `model.ipynb` | Main Gurobi optimization model, including portfolio constraints, optimal weights, out-of-sample evaluation, and sanity checks. |
| `selected_monthly_return.csv` | Monthly returns for 37 candidate stocks, including 2026 observations and historical mean returns. |
| `covariance_matrix.csv` | 37 × 37 sample covariance matrix estimated using 60 monthly observations. |
| `SP500_monthly_202001_202603.csv` | S&P 500 monthly returns used for the benchmark. |
| `jan_2026_monthly_returns.csv` | January 2026 stock returns used for out-of-sample evaluation. |
| `README.md` | Project documentation. |

## Model Validation

The implementation includes independent checks of:
- Full investment and portfolio weight bounds.
- Minimum expected return requirement.
- Correlation-based exclusion constraints.
- Portfolio standard deviation and CVaR calculations.
- Maximum constraint violation reported by Gurobi.

January 2026 realized stock returns are used to evaluate the optimized portfolio out of sample. Only information available through December 2025 is used for parameter estimation and portfolio construction.

## Data Source

CRSP U.S. Stock Databases, accessed through WRDS. Raw CRSP data is not redistributed; the repository contains derived monthly returns and covariance data.

## References

- Fama, E. F., and French, K. R. (1993). Common Risk Factors in the Returns on Stocks and Bonds. *Journal of Financial Economics*, 33(1), 3–56.
- Jegadeesh, N., and Titman, S. (1993). Returns to Buying Winners and Selling Losers. *The Journal of Finance*, 48(1), 65–91.
- Rockafellar, R. T., and Uryasev, S. (2000). Optimization of Conditional Value-at-Risk. *The Journal of Risk*, 2(3), 21–41.