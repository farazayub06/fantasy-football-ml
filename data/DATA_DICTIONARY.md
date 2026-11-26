# Data Dictionary

Every column we keep: name, meaning, where it came from. Partner A maintains this.

Data files are **not in Git** (`data/raw/` and `data/processed/*.parquet` are gitignored). Rebuild them by running the notebooks in order: `01_load_data` → `02_clean_data` → `04_features` (`03_exploration` is optional).

---

## 1. Sources (nflverse, loaded with `nflreadpy`)

| Raw file (`data/raw/`) | Loaded with | One row per | Player ID | Notes |
|---|---|---|---|---|
| `player_stats_week_2020_2026.parquet` | `load_player_stats(seasons, summary_level="week")` | player per week | `player_id` (GSIS, e.g. `00-0019596`) | 150 columns; all positions, regular season + playoffs |
| `schedules_2020_2026.parquet` | `load_schedules(seasons)` | game | none | Teams, scores, Vegas lines, rest days; includes unplayed 2026 games |
| `snap_counts_2020_2026.parquet` | `load_snap_counts(seasons)` | player per game | `pfr_player_id` (Pro Football Reference, e.g. `BradTo00`) | All positions incl. defense and special teams |
| `player_ids.parquet` | `load_players()`, columns `gsis_id`, `pfr_id` | player | both | Translator between GSIS and PFR IDs |

Full nflverse column definitions: [player stats](https://nflreadr.nflverse.com/articles/dictionary_player_stats.html) · [snap counts](https://nflreadr.nflverse.com/articles/dictionary_snap_counts.html) · [players](https://nflreadr.nflverse.com/articles/dictionary_players.html) (also in `useful_links.txt`).

## 2. Processed files (`data/processed/`)

| File | Made by | One row per | Contents |
|---|---|---|---|
| `player_weeks_clean.parquet` | `02_clean_data` | QB/RB/WR/TE player per regular-season game | All 150 player-stat columns + snap counts + schedule columns (section 4) |
| `features.parquet` | `04_features` | same rows, plus one placeholder row per player for the next unplayed week | 9 identifiers + 26 features + target (section 3) |

**Cleaning rules (notebook 02):** keep QB, RB, WR, TE only; regular season only (`season_type == "REG"`); drop rows with a blank `player_id` (junk rows). Snap counts are joined through `player_ids` (PFR ID → GSIS ID), then on `player_id` + `season` + `week`.

---

## 3. `features.parquet`

### Identifiers (never given to the model)

| Column | Meaning | Source |
|---|---|---|
| `player_id` | NFL GSIS player ID; the project's one player key | player stats |
| `player_display_name` | Player name, for reading results | player stats |
| `position` | QB, RB, WR or TE; one model is trained per position | player stats |
| `team` | Player's team that week (abbreviation, e.g. `MIN`) | player stats |
| `opponent_team` | Team he played against | player stats |
| `season` | NFL season (2020-2026); used for the train/test/live split | player stats |
| `week` | Week of the season | player stats |
| `history_flag` | `ok` (has earlier games this season) · `season_opener` (first game = team's first game) · `limitation_returning` (first game came later; seen in an earlier season) · `limitation_new` (first game came later; never seen before). Only `ok` rows are used for training and scoring | built in `04_features` |
| `is_upcoming` | 1 = game not played yet (a placeholder row for next week's prediction; target is blank), 0 = played | built in `04_features` |

### Target (what the model predicts)

| Column | Meaning | Source |
|---|---|---|
| `fantasy_points_ppr` | PPR fantasy points scored in this game (1 point per reception) | player stats |

### Features (26)

Every feature uses **only games before this one** (`shift(1)` in the code), except game context, which is known before kickoff. "Last 3" means the player's previous 3 games **this season** (fewer if he has played fewer); history resets each season. "Sum ÷ sum" means totals over those games are divided, so a small game doesn't count as much as a big one.

| Group | Column | Formula | Useful for | Built from |
|---|---|---|---|---|
| History | `fantasy_points_ppr_last1` | PPR points in his previous game | All | `fantasy_points_ppr` |
| History | `fantasy_points_ppr_last3` | Average PPR points, last 3 games | All | `fantasy_points_ppr` |
| History | `fantasy_points_ppr_season` | Average PPR points, all earlier games this season | All | `fantasy_points_ppr` |
| Usage | `targets_last3` | Average targets, last 3 games | RB, WR, TE | `targets` |
| Usage | `receptions_last3` | Average catches, last 3 games | RB, WR, TE | `receptions` |
| Usage | `carries_last3` | Average carries, last 3 games | RB, QB | `carries` |
| Usage | `offense_pct_last3` | Average share of team offensive snaps (0-1), last 3 games | All | `offense_pct` (snap counts) |
| Role | `target_share_last3` | Average share of team targets, last 3 games | RB, WR, TE | `target_share` (nflverse) |
| Role | `air_yards_share_last3` | Average share of team air yards, last 3 games | WR, TE | `air_yards_share` (nflverse) |
| Role | `wopr_last3` | Average WOPR = 1.5 × target share + 0.7 × air yards share, last 3 games | WR, TE | `wopr` (nflverse) |
| Role | `carry_share_last3` | Average of carries ÷ team carries that game, last 3 games | RB | `carries` |
| Passing | `attempts_last3` | Average pass attempts, last 3 games | QB | `attempts` |
| Passing | `passing_yards_last3` | Average passing yards, last 3 games | QB | `passing_yards` |
| Passing | `passing_tds_last3` | Average passing TDs, last 3 games | QB | `passing_tds` |
| Passing | `passing_interceptions_last3` | Average interceptions thrown, last 3 games | QB | `passing_interceptions` |
| Passing | `epa_per_att_last3` | Sum of passing EPA ÷ sum of attempts, last 3 games; blank if under 20 attempts | QB only | `passing_epa`, `attempts` |
| Passing | `rushing_yards_last3` | Average rushing yards, last 3 games | QB, RB | `rushing_yards` |
| Efficiency | `catch_rate_last3` | Catches ÷ targets (sum ÷ sum), last 3 games; blank if 0 targets | RB, WR, TE | `receptions`, `targets` |
| Efficiency | `yards_per_catch_last3` | Receiving yards ÷ catches (sum ÷ sum), last 3 games; blank if 0 catches | RB, WR, TE | `receiving_yards`, `receptions` |
| Efficiency | `yards_per_carry_last3` | Rushing yards ÷ carries (sum ÷ sum), last 3 games; blank if 0 carries | RB, QB | `rushing_yards`, `carries` |
| Team | `team_att_avg` | Team's pass attempts per game, average of earlier games this season | All | `attempts` (summed by team) |
| Opponent | `opp_ppr_allowed_avg` | PPR points this defense allowed to the player's position per game, average of earlier weeks this season | All | `fantasy_points_ppr` (summed by opponent + position) |
| Game context | `implied_total` | Points Vegas expects the player's team to score: home = (total + spread) ÷ 2, away = (total − spread) ÷ 2 | All | `total_line`, `spread_line` (schedules) |
| Game context | `total_line` | Vegas game total: points expected from both teams combined | All | schedules |
| Game context | `is_home` | 1 = home game, 0 = away | All | `team`, `home_team` |
| Game context | `rest_days` | Days since the team's previous game | All | `home_rest` / `away_rest` (schedules) |

**Blanks:** about 11% of rows have no history (each player's first game of a season) and are dropped by `history_flag`. Efficiency ratios are blank when there's nothing to divide by. `epa_per_att_last3` is blank for non-QBs (about 91% of rows), so RB/WR/TE models skip it. Linear regression fills remaining blanks with the training median plus a "was missing" flag; gradient boosting handles blanks itself.

---

## 4. Columns added in `02_clean_data`

Added to `player_weeks_clean.parquet` alongside the 150 player-stat columns.

| Column | Meaning | Source |
|---|---|---|
| `offense_snaps` | Offensive snaps played that game | snap counts |
| `offense_pct` | Share of team offensive snaps played, **0-1 scale** (1.0 = every snap). Blank for 38 player-games with no snap match (0.1%) | snap counts |
| `home_team`, `away_team` | Teams in the game | schedules |
| `total_line` | Vegas game total (over/under) | schedules |
| `spread_line` | Vegas spread from the home team's view: positive = home team favored by that many points | schedules |
| `home_rest`, `away_rest` | Days of rest for each team | schedules |
| `is_home` | 1 if `team == home_team`, else 0 | derived |
| `rest_days` | `home_rest` if home, else `away_rest` | derived |
| `implied_total` | See the features table | derived |

## 5. Raw player-stat columns used to build features

From `stats_player_week` (definitions from nflverse).

| Column | Meaning |
|---|---|
| `game_id` | Game key, e.g. `2020_01_TB_NO`; joins to schedules |
| `season_type` | `REG` (regular season) or `POST` (playoffs); only `REG` is kept |
| `targets` | Passes thrown to the player |
| `receptions` | Passes caught |
| `receiving_yards` | Yards gained on catches |
| `carries` | Rushing attempts |
| `rushing_yards` | Yards gained rushing |
| `attempts` | Pass attempts (QBs, plus rare trick plays) |
| `passing_yards`, `passing_tds`, `passing_interceptions` | Passing production and mistakes |
| `passing_epa` | Total expected points added (EPA) on the player's pass plays: how much each play raised or lowered the team's expected points |
| `target_share` | Player's targets ÷ team targets that game |
| `air_yards_share` | Player's air yards ÷ team air yards (air yards = how far the ball traveled past the line of scrimmage) |
| `wopr` | Weighted opportunity rating = 1.5 × `target_share` + 0.7 × `air_yards_share` |
| `fantasy_points` | Standard (non-PPR) fantasy points; **not** a feature, it's leakage (see below) |

**Leakage:** every same-game stat above is only known after the game and partly makes up the fantasy score, so none of them is ever a feature directly. Only their values from earlier games are used (see `docs/RESEARCH_LOG.md`, 2026-10-29 exploration entry).
