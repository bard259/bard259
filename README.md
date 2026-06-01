# Peijun Xu — Portfolio

Quantitative engineer and researcher working at the intersection of machine learning, optimization, and financial systems.

---

## Active Projects

| Project | Description | Stack | Status |
|---------|-------------|-------|--------|
| [casual-trading](#casual-trading) | AI-driven equity signal generation + automated P&L evaluation | Python, OpenAI, Supabase, GitHub Actions | Active |
| [broker-agent](#broker-agent) | Rule-based stock consultant combining technical signals with Reddit sentiment | Python, yfinance, VADER, PRAW | Active |
| [causal-impact](#causal-impact) | Causal inference toolkit for measuring the effect of strategy launches | Python, CausalImpact | Active |
| [nba-fantasy](#nba-fantasy) | Linear programming optimizer for salary-cap fantasy basketball drafts | Python, PuLP, pandas | Active |
| [mindmap](#mindmap) | LLM-powered interactive knowledge mind-map for any topic | Node.js, React Native, Expo, OpenAI | Active |

## Research & Academic Work

| Project | Description | Methods |
|---------|-------------|---------|
| [Robust-Mendelian-Randomization](#robust-mendelian-randomization) | Robust confidence intervals for MR-SPI causal inference | R, Robust Optimization, Two-Sample IV |
| [BlueBike-Project](#bluebike-project) | Optimal bike-share rebalancing under demand uncertainty | Adaptive Robust Optimization |
| [Chess-RL](#chess-rl) | Reinforcement learning agent for chess | Python, RL |
| [TF-Capstone](#tf-capstone) | Graduate capstone project | Quantitative Methods |

---

## Project Deep-Dives

### casual-trading

An automated pipeline that runs every trading day at market close. GPT-4o-mini generates directional signals (`buy`/`sell`/`hold`) for a configurable watchlist, persists them to Supabase, and then evaluates each signal at 1-day and 7-day horizons using actual closing prices from yfinance.

**Architecture**
```
GitHub Actions (cron: 4pm ET)
  └── run_market_close_pipeline.py
        ├── generate_actions.py   → OpenAI → Supabase (agent_actions table)
        └── evaluate_actions.py   → yfinance → Supabase (agent_evals table)
```

**Roadmap**
- [ ] **Backtesting module** — replay historical signals against actual prices to compute hit-rate, Sharpe ratio, and drawdowns before live deployment
- [ ] **Signal enrichment** — inject live fundamentals (P/E, earnings date), news headlines, and options flow into the prompt context
- [ ] **Multi-model comparison** — run identical prompts through GPT-4o, Claude Sonnet, and a rule-based baseline; track each in a separate column so you can A/B at the eval layer
- [ ] **Portfolio-level P&L tracker** — aggregate eval returns into a paper-trading ledger with position sizing (e.g., Kelly criterion) and daily NAV curve
- [ ] **Risk controls** — add max concentration per ticker, hard stop-loss threshold, and a "no-trade" circuit breaker on high-VIX days
- [ ] **Dashboard** — Supabase dashboard or lightweight Streamlit app showing signal history, hit rates, and cumulative return by model

---

### broker-agent

A lower-level trading consultant that constructs decisions from raw signals rather than asking an LLM directly. It pulls 90 days of daily OHLCV data via yfinance, computes RSI and momentum, scrapes Reddit (r/stocks, r/wallstreetbets, etc.) for recent posts, scores them with VADER sentiment, and applies a transparent rule tree to emit `buy`/`hold`/`short`.

**Architecture**
```
consult(ticker)
  ├── _build_features()
  │     ├── _fetch_prices()      → yfinance OHLCV
  │     ├── _rsi()               → 14-day RSI
  │     └── _fetch_reddit_texts() → PRAW → VADER sentiment
  └── _decide_with_explain()     → rule-based decision + rationale
```

**Roadmap**
- [ ] **Replace PSAW** — PushShift is deprecated; migrate historical Reddit data pipeline to the official Reddit API v2 or a paid alternative (Apify, Coresignal)
- [ ] **Add MACD and Bollinger Bands** to the feature set alongside RSI
- [ ] **Backtesting harness** — run `consult()` over historical dates and compare decisions against actual price moves using the same return framework as `casual-trading/evaluate_actions.py`
- [ ] **Unify with casual-trading** — expose `broker-agent` signals as a context input to `casual-trading`'s GPT prompt, creating a hybrid rule + LLM pipeline
- [ ] **CLI and packaging** — ship a proper `pip install broker-agent` with a `broker consult NVDA --explain` CLI entry point
- [ ] **Confidence calibration** — replace the binary rule tree with a logistic regression or gradient boosted model trained on the backtested signal history

---

### causal-impact

A thin but properly packaged Python library that wraps Google's `causalimpact` library for measuring the causal effect of interventions (strategy launches, A/B tests, policy changes). Given treated time-series data and synthetic controls, it runs a Bayesian structural time-series model over the pre/post-intervention period.

**Current gaps**
- `run_causal_impact_analysis` is defined twice (duplicate in `core.py`)
- `causalimpact` is missing from `pyproject.toml` dependencies
- No tests, no examples with real data

**Roadmap**
- [ ] **Fix the duplicate function definition** in `core.py`
- [ ] **Add `causalimpact` to `pyproject.toml`** dependencies
- [ ] **Add a Jupyter notebook example** using a real publicly available event (e.g., a product launch or policy change on an open dataset)
- [ ] **Multiple control selection strategies** — implement automated synthetic control selection using correlation-based ranking or elastic net
- [ ] **Test suite** — pytest covering the data preparation, alignment, and return-value contract
- [ ] **CI workflow** — GitHub Actions to run tests and publish to PyPI on tag push
- [ ] **Integration with casual-trading** — use causal-impact to measure the incremental return of switching from one signal model to another mid-season

---

### nba-fantasy

A salary-cap fantasy basketball draft optimizer. Given player projections and team schedule data, it builds a linear programming model (PuLP) that selects a 13-man roster maximizing projected season points under position constraints and a \$200 budget. It also computes "fair value" for any player via binary search and simulates competitive pressure pricing from other managers.

**Architecture**
```
sim(df)
  ├── run_proposal()   → PuLP LP solve → optimal 13-man roster
  ├── fair_value(name) → binary search over price → last price where player is included
  └── market_price()   → propagate price signal across remaining available players
```

**Roadmap**
- [ ] **Remove duplicate `df_load`** — `data.py` is a copy of `functions.py`; delete `data.py` and import from `functions.py`
- [ ] **2025-26 season data pipeline** — automate projection ingestion from a public source (Basketball-Reference, ESPN API, or HashtagBasketball scraper)
- [ ] **CLI interface** — `python -m draft_supporter` interactive draft session that accepts `my_pick PLAYER PRICE` and `other_pick PLAYER PRICE` commands
- [ ] **Live injury/news integration** — pull beat-reporter tweets or official NBA injury reports to downgrade or flag players at draft time
- [ ] **Robust draft proposals** — instead of a single optimal solution, produce a set of K near-optimal rosters to hedge against the uncertainty of which players will actually be available
- [ ] **Web UI** — a simple React app where you type picks in real-time and get a refreshed optimal proposal after each nomination

---

### mindmap

A cross-platform (web + iOS + Android) interactive mind-mapping app. The user types any topic, and an Express backend calls `gpt-4.1-mini` with a structured JSON schema to generate 3 subtopics with descriptions. Clicking any node expands it recursively, building a navigable knowledge tree. SVG rendering in React Native via an Expo frontend.

**Architecture**
```
Frontend (Expo / React Native)
  └── MindMap.js (SVG component)
        └── services/mindmap.js  → POST /api/mindmap

Backend (Express + Node.js)
  └── server.js
        └── OpenAI Responses API (gpt-4.1-mini, strict JSON schema)
```

**Roadmap**
- [ ] **Fix API key security** — `EXPO_PUBLIC_` prefix exposes the OpenAI key to the browser bundle; move all OpenAI calls to the server and remove the client-side key
- [ ] **Save and export** — let users download the current tree as JSON (for re-import), PNG screenshot, or a formatted PDF outline
- [ ] **Edit and delete nodes** — long-press to rename or prune any node in the tree
- [ ] **Persistence** — store trees in localStorage (web) or AsyncStorage (mobile) so sessions survive page reload; optionally sync to a backend
- [ ] **Perspective and audience controls** — the backend already accepts `perspective` and `purpose` fields; surface them in the UI as configurable settings before generation
- [ ] **Sharing** — generate a read-only shareable URL for any saved tree
- [ ] **Switch to Claude** — the backend is LLM-agnostic at the API boundary; swap `gpt-4.1-mini` for `claude-haiku-4-5` and compare explanation quality and depth

---

### Robust-Mendelian-Randomization

Graduate research applying the philosophy of robust optimization to Mendelian Randomization (MR-SPI). Instead of finding the single best causal effect estimate, the method finds the tightest confidence interval that remains valid under a worst-case subset of invalid instrumental variables. Published as a preprint.

**Roadmap**
- [ ] Add a `README.md` with abstract, methodology summary, and link to preprint
- [ ] Package the R code as a proper R package or at minimum a standalone `.R` script with documented function signatures
- [ ] Add an end-to-end reproducibility script that installs dependencies and replicates the paper's main table/figure

---

### BlueBike-Project

Academic project applying Adaptive Robust Optimization (ARO) to the BlueBike (Boston) station rebalancing problem. The model minimizes worst-case rebalancing cost under an uncertainty set for ridership demand. Results are in the PDF.

**Roadmap**
- [ ] Add a `README.md` with problem statement, methodology, and key results summary
- [ ] Add the optimization model code (Python + Gurobi/CVXPY or MATLAB)
- [ ] Add a data pipeline to pull current BlueBike trip data from the public GBFS feed

---

### Chess-RL

A reinforcement learning agent trained to play chess. The approach and results are documented in the PDF.

**Roadmap**
- [ ] Add a `README.md` with model architecture, training approach, and evaluation results
- [ ] Add the training code and pre-trained model weights
- [ ] Add a playable demo (CLI or simple web interface)

---

### TF-Capstone

Graduate capstone project. Contains a poster presentation.

**Roadmap**
- [ ] Add a `README.md` with abstract, methodology, and key findings
- [ ] Export poster to PDF for in-browser viewing on GitHub
- [ ] Add underlying analysis code and data

---

## Skills & Stack

**Languages:** Python, R, JavaScript / TypeScript  
**ML / Optimization:** PyTorch, scikit-learn, PuLP, CVXPY, Robust Optimization  
**Data:** pandas, numpy, yfinance, Supabase, PostgreSQL  
**AI / LLMs:** OpenAI API, Anthropic API, LangChain  
**Web / Mobile:** React Native, Expo, Express, Node.js  
**Infrastructure:** GitHub Actions, Docker  

---

*Last updated: June 2026*
