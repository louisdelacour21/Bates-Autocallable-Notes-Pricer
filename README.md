# Bates Autocallable Notes Pricer

[![Python 3.7+](https://img.shields.io/badge/python-3.7+-blue.svg)](https://www.python.org/downloads/)
[![QuantLib](https://img.shields.io/badge/QuantLib-1.25+-green.svg)](https://www.quantlib.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A professional pricing engine for autocallable structured products using the **Bates stochastic volatility model** (Heston + jumps), simulated with Monte Carlo methods.

## Overview

This project implements a robust `BatesPricer` class that enables pricing of complex autocallable notes by:

* Calibrating the Bates model to market implied volatility data
* Generating Monte Carlo paths with stochastic volatility and jumps
* Pricing various autocallable structures with early redemption features
* Supporting advanced features like coupon memory, protection barriers, jump sensitivity, and custom payoff logic

## Features

* **Bates Model Calibration**: Fit to market data with implied volatility surface
* **Monte Carlo Simulation**: Joint simulation of volatility and jump processes
* **Autocallable Product Support**: Including Phoenix and Athena-style notes
* **Advanced Payoff Handling**: Memory effect, barrier conditions, callable logic
* **Greeks Estimation**: Delta, Gamma, Vega, and jump sensitivity via bumping
* **QuantLib Integration**: Accurate date handling, pricing, and discounting

## Installation

```bash
pip install quantlib-python numpy scipy
```

## Example Usage

```python
from bates_model_pricer import BatesPricer
import QuantLib as ql
import numpy as np

# Market setup
valuation_date = ql.Date(20, 7, 2024)
spot_price = 79.98
strike_price = 82
risk_free_rate = 0.02
dividend_yield = 0.028

pricer = BatesPricer(valuation_date, spot_price, strike_price,
                     risk_free_rate, dividend_yield)

# Calibration market data
market_data = {
    'expiration_dates': [ql.Date(20, 1, 2025), ql.Date(20, 7, 2025)],
    'strikes': [65.6, 73.8, 82.0, 90.2, 98.4],
    'volatilities': [[0.26, 0.23, 0.21, 0.24, 0.27], [0.25, 0.22, 0.20, 0.23, 0.26]]
}

initial_params = [0.04, 2.0, 0.04, 0.3, -0.6, 0.5, -0.1, 0.2]
bounds = [
    (0.01, 0.5), (0.1, 10.0), (0.01, 0.5), (0.01, 1.0),
    (-0.99, 0.99), (0.0, 5.0), (-0.5, 0.5), (0.01, 1.0)
]

bates_params = pricer.calibrate_bates_model(market_data, initial_params, bounds)

# Product definition
product_specs = {
    'coupon_dates': np.array([...]),
    'notional': 1_000_000.0,
    'autocall_barrier': 1.0,
    'coupon_barrier': 0.7,
    'protection_barrier': 0.6,
    'coupon_rate': 0.06,
    'has_memory': True,
    'past_fixings': {},
    'final_payoff_formula': lambda x: x / strike_price
}

price = pricer.price_autocallable_note(product_specs, bates_params, num_paths=10000)
print(f"Note Price: ${price:,.0f}")
```

## Greeks Calculation

```python
greeks = pricer.calculate_greeks(product_specs, bates_params, num_paths=10000)
print(greeks)
```

## Architecture

### BatesPricer Class

```python
class BatesPricer:
    def __init__(...)
    def calibrate_bates_model(...)
    def generate_paths_with_jumps(...)
    def price_autocallable_note(...)
    def calculate_greeks(...)
```

### Bates Model Parameters

* **v0**: Initial variance
* **kappa**: Mean reversion speed
* **theta**: Long-term variance
* **sigma**: Vol of vol
* **rho**: Correlation
* **lambda**: Jump intensity
* **mu\_j**: Mean jump size
* **sigma\_j**: Jump size volatility

## Notes

* All simulations use `QuantLib` day-count conventions (TARGET calendar, Actual360)
* Simulation includes full drift compensation for jump risk
* Greeks are calculated via central differences
* Paths include past fixings for accurate valuation of live products

## References

* Bates, D. (1996). *Jumps and Stochastic Volatility: Exchange Rate Processes...*
* Merton, R. (1976). *Option pricing when underlying stock returns are discontinuous*
* Gatheral, J. (2006). *The Volatility Surface*
* QuantLib documentation: [https://www.quantlib.org/](https://www.quantlib.org/)

## License

MIT License
