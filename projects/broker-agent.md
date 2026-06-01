# broker-agent

Rule-based stock consultant that combines technical indicators with Reddit sentiment analysis to emit `buy` / `hold` / `short` decisions with a transparent rationale. No LLM required — every decision is traceable to a specific rule.

## How it works

```
consult(ticker)
  ├── _fetch_prices()          → yfinance: 90-day OHLCV
  ├── _rsi()                   → 14-day RSI
  ├── r1, r5                   → 1-day and 5-day momentum
  ├── _fetch_reddit_texts()    → PRAW: recent posts from r/stocks, r/wallstreetbets, etc.
  ├── _score_sentiment()       → VADER compound scores
  └── _decide_with_explain()   → rule tree → decision + rationale
```

**Decision rules (in priority order):**

| Condition | Signal |
|-----------|--------|
| Sentiment mean ≥ 0.10, RSI < 70, momentum ≥ −2% | `buy` |
| Sentiment mean ≤ −0.10, RSI > 30, momentum ≤ 2% | `short` |
| No sentiment data, r5 > 2%, RSI 40–68 | `buy` |
| No sentiment data, r5 < −2%, RSI 32–60 | `short` |
| Otherwise | `hold` |

## Installation

```bash
pip install git+https://github.com/bard259/broker-agent.git
```

## Usage

```python
from broker_agent.mvp.consultant import consult

# Simple decision
decision = consult("NVDA")
print(decision)  # 'buy' | 'hold' | 'short'

# With full feature breakdown
decision, details = consult("NVDA", explain=True)
print(details["features"])   # RSI, momentum, sentiment scores
print(details["rationale"])  # which rule fired
```

## Setup (for Reddit sentiment)

Create a Reddit app at https://www.reddit.com/prefs/apps and set these environment variables:

```bash
REDDIT_CLIENT_ID=your_client_id
REDDIT_CLIENT_SECRET=your_client_secret
REDDIT_USER_AGENT=broker-agent/1.0 by u/your_username
```

Without these, the agent falls back to technicals-only mode.

## Environment variables

| Variable | Required | Description |
|----------|----------|-------------|
| `REDDIT_CLIENT_ID` | Optional | Reddit app client ID |
| `REDDIT_CLIENT_SECRET` | Optional | Reddit app client secret |
| `REDDIT_USER_AGENT` | Optional | Reddit API user agent string |

## Roadmap

- [ ] **Replace PSAW** — PushShift is deprecated; migrate historical Reddit data to the official Reddit API v2 or an alternative (Apify, Coresignal)
- [ ] **Expand technical indicators** — add MACD signal line crossover and Bollinger Band squeeze to the feature set
- [ ] **Backtesting harness** — run `consult()` over historical dates and compare decisions against actual price moves using the same evaluation framework as `casual-trading/evaluate_actions.py`
- [ ] **Hybrid pipeline** — expose `broker-agent` signals as structured context in `casual-trading`'s GPT prompt for a combined rule + LLM approach
- [ ] **Confidence model** — replace the binary rule tree with a logistic regression or GBDT trained on the backtested signal history
- [ ] **CLI** — `broker consult NVDA --explain` and `broker backtest NVDA --start 2024-01-01`

## License

MIT
