# Risk-Aware Allocation to Large-Cap, High-Momentum Stocks

IEOR 4004 — Optimization Models and Methods — Team 14

**Team members:** Benjamin Ma, Grégoire Buot, Haixin Shen, Xuanyu Wang, Jiaxuan Yin

## Project Overview

We construct a portfolio of large-cap U.S. stocks with strong momentum for January 2026. Our objective is to minimize monthly 95% Conditional Value-at-Risk (CVaR) under a parametric Normal model.

The optimization is subject to:
- Full investment of available capital.
- No short selling.
- A minimum allocation of 1% and maximum of 20% for each selected stock.
- A minimum expected monthly return equal to the historical mean monthly S&P 500 return.
- A correlation-based diversification rule that prevents simultaneously holding two stocks with estimated correlation above 0.40.

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
| P | Stock pairs whose estimated return correlation exceeds 0.40 |

The correlation restriction is intended to provide an additional diversification rule beyond the covariance-based CVaR objective.

For computational implementation, auxiliary variables are used to represent the portfolio standard deviation in Gurobi. The norm is scaled by 100 inside the solver to improve numerical accuracy.

## Main Results

| Metric | Value |
|---|---|
| Optimal monthly 95% CVaR | 5.9301% |
| Expected monthly return | 3.0044% |
| Monthly standard deviation | 4.3314% |
| S&P 500 benchmark mean monthly return (b) | 1.0990% |
| Stocks held | 9 (ESLT and CAH at about 20% each) |
| Out-of-sample cumulative return, January–March 2026 (buy-and-hold) | 10.55% (S&P 500: −4.63%) |

## Sensitivity Analysis

Each policy parameter is varied one at a time and the model is re-solved (baseline: correlation threshold 0.40, weight cap 20%):
- **Correlation threshold (0.20 to 0.80, baseline 0.40):** the optimal CVaR stays at 5.93% for any threshold of 0.40 or higher, because the nine holdings have pairwise correlations of at most about 0.30. The rule only becomes costly when tightened further: CVaR rises to 5.98% at 0.30 and to 6.47% at 0.20, where only six stocks can be held.
- **Maximum weight cap (10% to 30%):** a tighter cap spreads the portfolio over more stocks but raises CVaR (6.08% at a 15% cap with 10 stocks, 7.12% at a 10% cap with 12 stocks); raising the cap above 20% brings almost no improvement.
- **Maximum number of holdings (new constraint, K = 8, 7, 6):** limiting the portfolio to 8 stocks drops BABA and raises CVaR by only 0.05 percentage points; the cost then grows to 0.18 points with 7 stocks and 0.40 points with 6 stocks. ESLT, CAH, AVGO, WPM and FIX remain in every portfolio.

## Monte Carlo Simulation: Testing the Normality Assumption

The model's CVaR relies on jointly Normal returns. Keeping the optimal portfolio fixed, we simulate 100,000 months of returns under three generators and compare the empirical 95% CVaR (Rockafellar–Uryasev estimator) with the model's value:

| Generator | Empirical 95% CVaR | Ratio to model CVaR |
|---|---|---|
| Normal N(μ, Σ), the model's assumption | 5.93% | 1.00 |
| Multivariate Student-t, ν = 4, same mean and covariance | 6.83% | 1.15 |
| Historical bootstrap of the 60 estimation months | 5.54% | 0.93 |

If returns have fatter tails than the Normal distribution, the model underestimates tail risk by about 2% to 15%, depending on the degrees of freedom ν (3 to 30). The bootstrap does not show heavier tails over 2021–2025, but it is optimistic because the portfolio was optimized on the same months, and its 95% CVaR rests on only 3 months. The simulation is checked against closed-form CVaR values and Monte Carlo standard errors.

## What-if Analysis: Higher Return Requirement

We raise the minimum expected return b step by step and re-solve the model:
- Up to b ≈ 3.00%, the recommended portfolio stays optimal and the CVaR stays at 5.93%: the return constraint is not binding.
- Above 3.00%, the CVaR rises at an accelerating rate (6.33% at b = 3.50%, 8.16% at 4.00%, 12.40% at 4.50%) and the portfolio becomes more concentrated.
- The highest feasible requirement is b = 4.73%, reached only by holding LITE, PLTR, CLS, INSM and NRG at 20% each (CVaR 16.93%).

## Repository Contents

| File | Description |
|---|---|
| `model.ipynb` | Main notebook: Gurobi optimization model, optimal weights, sanity checks, out-of-sample evaluation, sensitivity analysis, Monte Carlo simulation, and what-if analysis. |
| `selected_monthly_return.csv` | Monthly returns for the 37 candidate stocks (January 2021–March 2026) and their historical mean returns. |
| `covariance_matrix.csv` | 37 × 37 sample covariance matrix estimated using 60 monthly observations. |
| `SP500_monthly_202001_202603.csv` | S&P 500 monthly returns used for the benchmark. |
| `jan_2026_monthly_returns.csv` | January 2026 stock returns, kept for reference (the notebook reads the 2026 returns from `selected_monthly_return.csv`). |
| `performance_comparison.png` | Out-of-sample cumulative returns of the portfolio and the S&P 500, January–March 2026. |
| `correlation_sensitivity.csv`, `correlation_sensitivity.png` | Results of the correlation threshold sensitivity analysis. |
| `weight_cap_sensitivity.csv`, `weight_cap_sensitivity.png` | Results of the maximum weight cap sensitivity analysis. |
| `max_holdings_sensitivity.csv`, `max_holdings_sensitivity.png` | Results of the maximum number of holdings sensitivity analysis. |
| `monte_carlo_cvar.csv`, `monte_carlo_cvar.png` | Monte Carlo results: empirical CVaR under the Normal, Student-t and bootstrap generators. |
| `monte_carlo_tail_sensitivity.csv` | Student-t CVaR for several degrees of freedom ν (simulated and closed form). |
| `whatif_return_hurdle.csv`, `whatif_return_hurdle.png` | What-if results: optimal portfolio for each required return b. |
| `README.md` | Project documentation. |

## Model Validation

The implementation includes independent checks of:
- Full investment and portfolio weight bounds.
- Minimum expected return requirement.
- Correlation-based exclusion constraints.
- Portfolio standard deviation and CVaR calculations.
- Maximum constraint violation reported by Gurobi.

Realized stock returns from January to March 2026 are used to evaluate the optimized portfolio out of sample: the portfolio is bought at the end of December 2025 and held without trading (buy-and-hold). Only information available through December 2025 is used for parameter estimation and portfolio construction.

## How to Run

1. Install Python 3 with `numpy`, `pandas`, `scipy`, `matplotlib`, `gurobipy` and Jupyter. Gurobi requires a license (a free academic license is available).
2. Open `model.ipynb` from the repository folder, so that the CSV files are found with relative paths.
3. Run all cells in order. The Monte Carlo cells use a fixed random seed, so their results are reproducible. The notebook regenerates the CSV and PNG result files listed above.

## Data Source

CRSP U.S. Stock Databases, accessed through WRDS. Raw CRSP data is not redistributed; the repository contains derived monthly returns and covariance data.

## References

- Fama, E. F., and French, K. R. (1993). Common Risk Factors in the Returns on Stocks and Bonds. *Journal of Financial Economics*, 33(1), 3–56.
- Jegadeesh, N., and Titman, S. (1993). Returns to Buying Winners and Selling Losers. *The Journal of Finance*, 48(1), 65–91.
- Rockafellar, R. T., and Uryasev, S. (2000). Optimization of Conditional Value-at-Risk. *The Journal of Risk*, 2(3), 21–41.
