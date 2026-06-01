# casual-trading

Automated AI-powered equity signal generation with closed-loop performance evaluation. Runs every trading day at market close via GitHub Actions.

## How it works

1. **Generate** — `generate_actions.py` asks GPT-4o-mini for a `buy`/`sell`/`hold` decision for each ticker in your watchlist and writes the result (action, confidence, reasoning) to Supabase.
2. **Evaluate** — `evaluate_actions.py` fetches actual closing prices via yfinance and computes realised 1-day and 7-day returns for every prior action, writing them to an `agent_evals` table.
3. **Orchestrate** — `run_market_close_pipeline.py` gates both scripts behind a trading-calendar check and a 4:00–4:20 pm ET window. Set `FORCE_RUN=1` to bypass for testing.
4. **Schedule** — `.github/workflows/market_close_pipeline.yml` triggers at 20:05 and 21:05 UTC on weekdays to cover both EST and EDT.

```
GitHub Actions (cron)
  └── run_market_close_pipeline.py
        ├── generate_actions.py   → OpenAI → Supabase: agent_actions
        └── evaluate_actions.py   → yfinance → Supabase: agent_evals
```

## Setup

### 1. Supabase schema

```sql
create table agent_actions (
  id          uuid primary key default gen_random_uuid(),
  run_date    date not null,
  ticker      text not null,
  action      text not null,        -- 'buy' | 'sell' | 'hold'
  confidence  float,
  reason      text,
  prompt      text,
  model       text,
  created_at  timestamptz default now(),
  unique (run_date, ticker)
);

create table agent_evals (
  id           uuid primary key default gen_random_uuid(),
  action_id    uuid references agent_actions(id),
  ticker       text,
  horizon      text,               -- '1d' | '7d'
  entry_time   timestamptz,
  exit_time    timestamptz,
  entry_price  float,
  exit_price   float,
  return       float,
  created_at   timestamptz default now(),
  unique (action_id, horizon)
);

-- Optional: ticker watchlist table
create table tickers (
  ticker    text primary key,
  is_active boolean default true
);
```

### 2. Configure your watchlist

Either set `WATCHLIST=AAPL,MSFT,NVDA` as an env var, or populate the `tickers` table and leave `WATCHLIST` unset.

### 3. GitHub Secrets

| Secret | Description |
|--------|-------------|
| `SUPABASE_URL` | Your Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Service role key (bypasses RLS) |
| `OPENAI_API_KEY` | OpenAI API key |

### 4. Local development

```bash
pip install -r requirements.txt
cp .env.example .env   # fill in your keys
FORCE_RUN=1 python run_market_close_pipeline.py
```

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SUPABASE_URL` | — | Required |
| `SUPABASE_SERVICE_ROLE_KEY` | — | Required |
| `OPENAI_API_KEY` | — | Required |
| `WATCHLIST` | `""` | Comma-separated tickers; falls back to `tickers` table |
| `OPENAI_MODEL` | `gpt-4o-mini` | Model used for signal generation |
| `MAX_TICKERS` | `50` | Cap on watchlist size per run |
| `ACTIONS_TABLE` | `agent_actions` | Supabase table for raw signals |
| `EVALS_TABLE` | `agent_evals` | Supabase table for realised returns |
| `HORIZON_MODE` | `calendar` | `calendar` or `trading` day horizon counting |
| `FORCE_RUN` | `0` | Set to `1` to bypass trading-session and time-window checks |

## Roadmap

- [ ] **Backtesting** — replay historical signals against actual prices to compute hit-rate, Sharpe, and max drawdown before committing to live generation
- [ ] **Signal enrichment** — inject fundamentals (P/E, earnings date), options flow, and news headlines into the prompt for richer context
- [ ] **Multi-model comparison** — run the same prompt through GPT-4o, Claude Sonnet, and a rule-based baseline; track model in `agent_actions` so evals can be segmented by model
- [ ] **Portfolio P&L tracker** — aggregate evaluated returns into a paper-trading ledger with position sizing and daily NAV curve
- [ ] **Risk controls** — max concentration per ticker, hard stop-loss, no-trade circuit breaker on high-VIX days
- [ ] **Dashboard** — Streamlit or Supabase dashboard showing signal history, hit rates, and cumulative return by model

## License

MIT
