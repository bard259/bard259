# causal-impact

Causal inference toolkit for measuring the effect of interventions — strategy launches, A/B tests, policy changes, product releases. Wraps Google's `causalimpact` Python library with data preparation utilities and a synthetic data generator for experimentation.

## How it works

Given a treated time series `y` and a set of synthetic control series `X`, the library fits a Bayesian structural time-series model on the pre-intervention period and extrapolates it as a counterfactual into the post-intervention period. The gap between actual and predicted values (with credible intervals) is the estimated causal effect.

```
run_causal_impact_analysis(y, X, treatment_time)
  ├── prepare_data_for_causal_impact()   → align treated + control series
  ├── CausalImpact(data, pre, post)      → fit BSTS model
  ├── impact.summary()                   → print effect estimate + p-value
  └── impact.plot()                      → visualise actual vs. counterfactual
```

## Installation

```bash
pip install git+https://github.com/bard259/causal-impact.git
```

## Quick start

```python
import pandas as pd
from causal_impact_mvp.core import run_causal_impact_analysis, generate_causal_data

# Generate synthetic data (100 points, intervention at t=70, effect size=5)
data = generate_causal_data(n_points=100, treatment_time=70, causal_effect=5)

y = data["y"]
X = data[["x1", "x2", "x3"]]

summary = run_causal_impact_analysis(y, X, treatment_time=70)
```

## With your own data

```python
import pandas as pd
from causal_impact_mvp.core import run_causal_impact_analysis

# y: pandas Series — the metric you want to measure (e.g., daily revenue)
# X: pandas DataFrame — control series (e.g., revenue of similar markets)
# Both must share the same DatetimeIndex

y = pd.read_csv("treated_market.csv", index_col=0, parse_dates=True)["revenue"]
X = pd.read_csv("control_markets.csv", index_col=0, parse_dates=True)

# treatment_time: integer index of the first post-intervention row
treatment_time = y.index.get_loc("2024-06-01")

run_causal_impact_analysis(y, X, treatment_time)
```

## API

### `run_causal_impact_analysis(y, X, treatment_time)`

| Parameter | Type | Description |
|-----------|------|-------------|
| `y` | `pd.Series` | Treated time series |
| `X` | `pd.DataFrame` | Synthetic control time series |
| `treatment_time` | `int` | Integer index of first post-treatment row |

Returns: summary string; prints summary and plots the impact chart.

### `generate_causal_data(n_points, treatment_time, causal_effect, n_covariates)`

Generates synthetic time series with a known causal effect for testing and demos.

## Roadmap

- [ ] **Fix duplicate function definition** in `core.py` (second `run_causal_impact_analysis` shadows the first)
- [ ] **Add `causalimpact` to `pyproject.toml`** dependencies — currently missing
- [ ] **Automated control selection** — implement correlation-based or elastic-net ranking to choose the best control series from a larger pool
- [ ] **Jupyter notebook example** — real-world demonstration using a public event dataset
- [ ] **Test suite** — pytest covering data alignment, treatment_time bounds, and return value contract
- [ ] **CI workflow** — GitHub Actions running tests on every push, publish to PyPI on tag
- [ ] **Date-indexed treatment** — accept a date string or `pd.Timestamp` for `treatment_time` in addition to an integer index

## License

MIT
