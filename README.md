# Projet Finance - Jet Fuel Hedging Strategy (JetBlue Case)

**Team:** Aicha Atouch, Assmaa Bamaarouf, Najma Atchany, Rachid Ait Jalloul, Soulaimane Elamari

## Overview

This project looks at hedging strategies for jet fuel (Jet A-1 kerosene) price risk, using JetBlue as the reference airline. The idea is to quantify how exposed the airline is to crude oil price swings and to recommend a hedging approach based on quantitative risk metrics.

## Problem Statement

JetBlue burns through roughly 853M gallons of jet fuel a year (about 1.7M barrels a month). Since fuel cost is simply price times quantity, the airline is directly exposed to crude oil price swings driven by macro and geopolitical shocks, including:

- 2008 to 2009: the global financial crisis (a demand shock)
- 2014 to 2016: the price war (US shale oversupply, OPEC quota standoff)
- 2020: the COVID-19 demand collapse
- 2022: the Russia-Ukraine war (geopolitical risk premium)
- 2025: Middle East tensions (Red Sea, Iran) threatening supply routes

## Methodology

### 1. Price Modeling (Black-Scholes / GBM)

Future crude oil prices are simulated with Geometric Brownian Motion:

```
S_t = S_0 · exp((μ − σ²/2)t + σW_t)
```

- **S₀** ≈ 60.88 $/bbl (spot price as of 08/12/2025)
- **μ** ≈ −1.16% (trailing 12-month drift)
- **σ** = 23% annualized (trailing 12-month volatility)
- **r** = 5% (industry-standard discount rate)

We ran a **Monte Carlo simulation with 200,000 scenarios** to generate the expected spot price along with the 5%, 50%, and 95% quantiles for each month of the hedge horizon.

### 2. Hedging Strategies Compared

| Strategy | Description |
|---|---|
| **No hedge** | Buy fuel at prevailing spot/short-term market prices |
| **Swap** | Average-price swap settled against trailing futures prices (additive in log-returns, standard in commodity finance) |
| **Strip of calls** | Monthly call options giving the right, not the obligation, to buy at a preset strike, derived from simulated price paths via log-returns |

### 3. Risk Metrics

- **VaR (Value at Risk)**: the cost threshold not exceeded in 95% of scenarios
- **CVaR / Expected Shortfall**: the average cost in the worst 5% of scenarios
- **Worst month (95%)**: the most expensive single month plausible at the 95% level
- **Upside**: the average cost in the best 5% of scenarios (i.e., how much the airline benefits if prices drop)

## Key Results

| Strategy | Avg. cost ($/bbl) | VaR95 | CVaR95 | Worst month (95%) | Upside (best 5%) |
|---|---|---|---|---|---|
| No hedge | 60.11 | 65.22 | 66.66 | 124 | 54.21 |
| Swap | 60.44 | 60.44 | 60.44 | 60.8 | 60.44 |
| Strip of calls | 59.49 | 60.85 | 61.10 | 62.8 | 58.01 |

**Takeaways:**
- **No hedge** has the widest cost dispersion. It offers the highest upside potential but also carries extreme tail risk (the worst month can reach $124/bbl).
- **Swap** locks in cost certainty (VaR equals CVaR equals average cost) but gives up any upside entirely.
- **Strip of calls** turns out to be the best middle ground: lowest average cost, protection against spikes that's close to what the swap offers, and it still lets the airline benefit if prices fall.

### Backtesting

A historical backtest (January 2024 to December 2025) confirms that a strip of calls (`P = min(S_real, K) + premium`) would have smoothed out the realized cost curve compared to raw spot prices. The calls strategy outperformed by roughly **$60M cumulative** over the period, mainly because realized spot prices exceeded the strike often enough to more than cover the option premium.

## Final Recommendation

Don't go "all or nothing." A blended approach balances budget certainty against optionality:

- **50 to 80% of volume in swaps**: secures the budget and minimizes variance/VaR
- **20 to 50% in calls**: protects against extreme spikes while keeping some downside upside
- Adjust the swap/call mix depending on risk tolerance and where option premiums stand

| Business priority | Recommended strategy |
|---|---|
| Budget certainty, zero surprises | Strip of swaps |
| Protection against spikes plus retained upside | Strip of calls |
| Balanced risk/cost tradeoff | Mixed swap/call portfolio |

## Repository Structure (suggested)

```
├── data/                  # Historical crude oil price series
├── notebooks/             # Monte Carlo simulation & backtesting code
├── Pres_finance_Groupe_2.pdf   # Final presentation
└── README.md
```
