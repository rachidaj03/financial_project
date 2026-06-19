# Projet Finance — Jet Fuel Hedging Strategy (JetBlue Case)

**Team:** Aicha Atouch, Assmaa Bamaarouf, Najma Atchany, Rachid Ait Jalloul, Soulaimane Elamari

## Overview

This project analyzes and compares hedging strategies for jet fuel (Jet A-1 kerosene) price risk, using JetBlue as the reference airline. The goal is to quantify the airline's exposure to crude oil price volatility and recommend an optimal hedging approach using quantitative risk metrics.

## Problem Statement

JetBlue consumes approximately 853M gallons of jet fuel per year (~1.7M barrels/month). Since fuel cost = price × quantity, the airline is directly exposed to crude oil price swings driven by macro and geopolitical shocks:

- 2008–2009: Global financial crisis (demand shock)
- 2014–2016: Price war (US shale oversupply, OPEC quota standoff)
- 2020: COVID-19 demand collapse
- 2022: Russia-Ukraine war (geopolitical risk premium)
- 2025: Middle East tensions (Red Sea / Iran) threatening supply routes

## Methodology

### 1. Price Modeling (Black-Scholes / GBM)
Future crude oil prices are simulated using Geometric Brownian Motion:

```
S_t = S_0 · exp((μ − σ²/2)t + σW_t)
```

- **S₀** ≈ 60.88 $/bbl (spot price as of 08/12/2025)
- **μ** ≈ −1.16% (trailing 12-month drift)
- **σ** = 23% annualized (trailing 12-month volatility)
- **r** = 5% (industry-standard discount rate)

A **Monte Carlo simulation with 200,000 scenarios** generates the expected spot price and 5%/50%/95% quantiles for each month of the hedge horizon.

### 2. Hedging Strategies Compared

| Strategy | Description |
|---|---|
| **No hedge** | Buy fuel at prevailing spot/short-term market prices |
| **Swap** | Average-price swap settled against trailing futures prices (additive in log-returns, standard in commodity finance) |
| **Strip of calls** | Monthly call options giving the right (not obligation) to buy at a preset strike, derived from simulated price paths via log-returns |

### 3. Risk Metrics

- **VaR (Value at Risk)** — the cost threshold not exceeded in 95% of scenarios
- **CVaR / Expected Shortfall** — average cost in the worst 5% of scenarios
- **Worst month (95%)** — the most expensive single month plausible at the 95% level
- **Upside** — average cost in the best 5% of scenarios (ability to benefit from price drops)

## Key Results

| Strategy | Avg. cost ($/bbl) | VaR95 | CVaR95 | Worst month (95%) | Upside (best 5%) |
|---|---|---|---|---|---|
| No hedge | 60.11 | 65.22 | 66.66 | 124 | 54.21 |
| Swap | 60.44 | 60.44 | 60.44 | 60.8 | 60.44 |
| Strip of calls | 59.49 | 60.85 | 61.10 | 62.8 | 58.01 |

**Takeaways:**
- **No hedge** carries the widest cost dispersion — highest upside potential, but extreme tail risk (worst month up to $124/bbl).
- **Swap** locks in cost certainty (VaR = CVaR = avg. cost) but eliminates all upside.
- **Strip of calls** is the best compromise: lowest average cost, near-swap protection against spikes, while preserving partial upside if prices fall.

### Backtesting
A historical backtest (Jan 2024–Dec 2025) confirms that a strip of calls (`P = min(S_real, K) + premium`) would have smoothed the realized cost curve versus raw spot prices, with the calls strategy outperforming by roughly **−$60M cumulative** over the period — largely because realized spot frequently exceeded the strike enough to offset the option premium.

## Final Recommendation

**Don't be "all or nothing."** A blended approach balances budget certainty with optionality:

- **50–80% of volume in swaps** — secures the budget, minimizes variance/VaR
- **20–50% in calls** — protects against extreme spikes while retaining downside upside
- Adjust the swap/call mix based on risk tolerance and prevailing option premiums

| Business priority | Recommended strategy |
|---|---|
| Budget certainty, zero surprises | Strip of swaps |
| Protection against spikes + retain upside | Strip of calls |
| Balanced risk/cost tradeoff | Mixed swap/call portfolio |

## Repository Structure (suggested)

```
├── data/                  # Historical crude oil price series
├── notebooks/             # Monte Carlo simulation & backtesting code
├── Pres_finance_Groupe_2.pdf   # Final presentation
└── README.md
```
