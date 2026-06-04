# NBA Live Win Probability Model

Real-time win probability estimation for NBA games using play-by-play data. This repo contains the full data pipeline and feature engineering — ingesting raw game events from the NBA API, reconstructing game state at every play, and building a labeled dataset ready for model training.

---

## What This Does

Most sports win probability models treat each game as a snapshot at a single point in time. This project reconstructs the **full timeline** of every game as a sequence of probabilistically-relevant states — so the model sees how a game evolved, not just where it ended up.

The pipeline covers ~5,000+ games across the 2022–2026 NBA regular seasons and playoffs.

---

## Pipeline

```
NBA API (live + historical)
        │
        ▼
  Raw play-by-play JSON
        │
        ▼
  Game state reconstruction    ← score, time, possession, fouls, momentum
        │
        ▼
  Team rolling statistics      ← last 10 / last 20 / last season windows
        │
        ▼
  Rest & fatigue features      ← back-to-back, 3-in-4 schedule flags
        │
        ▼
  Differential feature layer   ← home minus away for all rating columns
        │
        ▼
  nba_win_prob_final.csv       ← labeled dataset, one row per play
```

---

## Features

### In-game state (play level)
| Feature | Description |
|---|---|
| `seconds_remaining` | Total seconds left in regulation or OT |
| `score_diff` | Home score minus away score |
| `home_possession` | Binary: home team has possession |
| `momentum_5` / `momentum_10` | Score differential change over last 5 / 10 plays |
| `home_fouls` / `away_fouls` | Running foul totals |
| `is_overtime` | Binary flag |
| `period` | Current period (1–4, or 5+ for OT) |

### Team context (game level)
Rolling windows computed over last 10 games, last 20 games, and the previous season for each team:

| Stat | Description |
|---|---|
| `off_rtg` | Offensive rating (points per 100 possessions) |
| `def_rtg` | Defensive rating |
| `net_rtg` | Net rating differential |
| `efg` | Effective field goal % |
| `tov_pct` | Turnover rate |
| `ft_rate` | Free throw attempt rate |
| `oreb_pct` | Offensive rebounding % |

All rolling stats are also computed as **home minus away differentials** (e.g. `diff_net_rtg_last10`).

### Rest & schedule
| Feature | Description |
|---|---|
| `home_rest_days` / `away_rest_days` | Days since last game |
| `home_b2b` / `away_b2b` | Back-to-back flag |
| `home_3in4` / `away_3in4` | Third game in four days flag |
| `is_playoffs` | Playoff game flag |
| `day_of_week` / `is_weekend` | Schedule context |

---

## Target Variable

`home_team_won` — binary label from final score. Each row in the dataset is a single play event during the game, labeled with the eventual game outcome. The model learns to estimate P(home team wins | current game state).

---

## Status

- [x] Data ingestion (NBA API, 2022–2026)
- [x] Game state reconstruction
- [x] Team rolling statistics
- [x] Rest/fatigue features
- [x] Differential feature engineering
- [x] Final merged dataset (~5,000+ games, regular season + playoffs)
- [ ] Model training (logistic regression baseline + gradient boosting)
- [ ] Calibration and evaluation
- [ ] Live inference script

---

## Usage

```bash
pip install -r requirements.txt
```

Run `LiveProbabilityNBA.ipynb` sequentially. Sections are organized as:
1. Data fetching (pulls from NBA API — rate limiting handled)
2. Game state builder
3. Team statistics and rolling windows
4. Rest features
5. Final merge and save

The final dataset is saved to `nba_win_prob_final.csv`.

---

## Data Source

Play-by-play data pulled from the [NBA Stats API](https://www.nba.com/stats) via direct endpoint access. No third-party library required.

---

## Requirements

```
pandas
numpy
requests
tqdm
scikit-learn
```
