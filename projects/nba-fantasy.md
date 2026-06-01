# nba-fantasy

Linear programming optimizer for salary-cap fantasy basketball auctions. Given player projections and team schedule data, it selects the optimal 13-man roster maximising projected season points under position constraints and a $200 budget. Includes a fair-value price calculator and draft simulation engine.

## How it works

The `sim` class models an auction draft in progress:

- `run_proposal()` — solves an LP (via PuLP) to find the optimal roster given current market prices and available players
- `fair_value(name)` — binary-searches the highest price at which a player still appears in the optimal proposal
- `market_price(name, price)` — propagates a price signal: if a player goes for more than expected, the remaining budget redistributes across all other available players
- `my_pick(name, price)` / `other_pick(name, price)` — update market state after each nomination is won

```
sim(df)
  ├── run_proposal()     → PuLP LP solve → optimal 13-man roster
  ├── fair_value(name)   → binary search → max price to stay optimal
  └── market_price()     → propagate price signal across remaining players
```

## Installation

```bash
pip install git+https://github.com/bard259/nba-fantasy.git
```

## Usage

```python
from draft_supporter.functions import sim, df_load

df = df_load("players.csv", "teams.csv")
draft = sim(df=df)

# Get optimal roster at current prices
proposal = draft.run_proposal()
print(proposal[["player", "2025 total", "cur_price"]])

# Find fair value for a specific player
print(draft.fair_value("LeBron James"))

# Record a pick (player went for $45)
draft.my_pick("LeBron James", 45)

# Record an opponent's pick
draft.other_pick("Nikola Jokic", 60)

# Get updated optimal proposal with remaining budget
updated = draft.run_proposal()
```

## Data format

`players.csv` must include columns: `player`, `team-position`, `proj`, `avg`, `2025 proj`, `2024 avg`, `2024 total`, `2023 avg`, `2023 total`, `is_pg`, `is_sg`, `is_sf`, `is_pf`, `is_c`

`teams.csv` must include columns: `short` (team abbreviation), `before playoff` (games before playoff), `playoff week` (playoff weeks)

## LP constraints

- Exactly 13 players total
- Budget ≤ $200
- Roster slots: PG (1), SG (1), G (1), SF (1), PF (1), F (1), C (2), UTIL (3), BN (3)
- Minimum eligible players: ≥3 PG-eligible, ≥3 SG-eligible, ≥3 SF-eligible, ≥3 PF-eligible, ≥4 C-eligible
- Health + growth filter: ≥8 players with positive projected growth AND ≥65 games played last season

## Roadmap

- [ ] **Remove duplicate `df_load`** — `data.py` duplicates `functions.py`; delete `data.py`
- [ ] **2025-26 data pipeline** — automated projection ingestion from Basketball-Reference or HashtagBasketball
- [ ] **CLI interface** — interactive draft session: `python -m draft_supporter` accepting `my_pick PLAYER PRICE` commands
- [ ] **Robust proposals** — return K near-optimal rosters to hedge against player unavailability
- [ ] **Injury/news signals** — scrape official NBA injury reports to downgrade or flag players at draft time
- [ ] **Web UI** — React app for live draft sessions showing real-time optimal proposal after each nomination

## License

MIT
