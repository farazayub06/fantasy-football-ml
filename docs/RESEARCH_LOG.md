# Research Log

Both partners add an entry at the end of **every** work session, newest at the top. Five minutes is enough. These entries become the raw material for the poster, so write down numbers and surprises while you still remember them.

## Research questions

1. Can a machine learning model using usage, matchup and Vegas features predict weekly PPR points more accurately than a simple recent-form baseline?
2. Which features matter most, and does that differ by position?
3. Where does the model fail?
4. *(Stretch)* How does it compare with expert consensus projections?

## Starting hypothesis

*Written 2026-10-15, before any models were trained.*

- **H1 (main):** A gradient boosting model using usage, matchup and Vegas features will have lower MAE than the last-3-week-average baseline on the 2025 test season, for every position.
  - *Null:* the model's MAE is no better than the baseline's.
  - *Test:* `reports/tables/model_comparison.csv` (checkpoint 3), reported in points and percent.
- **H2 (features):** Usage features (target share, snap %, carries, air yards share) will matter more than touchdowns or efficiency stats, because a player's role is more stable week to week than his touchdowns.
  - *Test:* feature importance (checkpoint 5).
- **H3 (by position):** WR/TE: target share and air yards matter most. RB: carries, snap % and implied team total. QB: implied team total and EPA per attempt.
  - *Test:* per-position feature importance panels (checkpoint 5).
- **H4 (failures):** The largest errors will be in weeks 1–3 (little history), multi-touchdown games, and weeks when a teammate at the same position is injured.
  - *Test:* error analysis by week and the weekly "biggest misses" log (checkpoints 4–5).

- 

---

## Entry template (copy for each session)

```
### YYYY-MM-DD · Name · Phase X

**What I did:**
-

**Numbers / results:**
-

**What surprised me:**
-

**Problems or open questions:**
-

**Saved for the poster:** (file paths in reports/)
-

**Next step:**
-
```

---

## Weekly live record (Phase 4)

| Week | Model MAE | Baseline MAE | Biggest miss | Why |
|---|---|---|---|---|
| | | | | |

---

## Entries

### 2026-10-16 · Preston · Phase 1

**What I did:**
- Downloaded schedules and snap counts for 2020-2026, plus the player ID table (`load_players`)
- Saved to `data/raw/` as Parquet: `schedules_2020_2026`, `snap_counts_2020_2026`, `player_ids` (gitignored)

**Numbers / results:**
- Schedules: 1,965 games (includes unplayed 2026 games, which already have Vegas lines)
- Snap counts: 164,986 rows; latest 2026 week: 5
- `offense_pct` is on a 0-1 scale
- Snap rows with no PFR ID: 0
- Players with both a GSIS and a PFR ID: 22,689

**What surprised me:**
- Snap counts use Pro Football Reference IDs (`pfr_player_id`), not the NFL GSIS IDs (`player_id`) used in player stats, so the tables can't be merged directly. The player ID table works as a translator. However it looks like all players have both a PFR and GSIS. Will have to see when doing merging.

**Problems or open questions:**
- Some snap rows may not match a GSIS ID. Count them after the merge and record how we handle them (DECISIONS.md: missing snap counts)
- Snap counts include defense and special teams; filter to QB/RB/WR/TE

**Saved for the poster:**
- Nothing yet (raw data only)

**Next step:**
- Combine the tables, filter to QB/RB/WR/TE, remove junk rows, merge snap counts through the ID table

### 2026-10-15 · Preston · Phase 1

**What I did:**
- Created `notebooks/01_load_data.ipynb`
- Downloaded weekly player stats for 2020-2026 with `nflreadpy.load_player_stats(summary_level="week")`
- Saved to `data/raw/player_stats_week_2020_2026.parquet` (gitignored)
- Essentially work flow is for parquet files --> run script to load data and save as parquet if parquet files dont exist --> save data as parquet files locally --> refer to locally saved data in code to use data when data is not saved as variable in memory.
- Parquet file is constantly updated to hold most recent data, since data is saved as a parquet file each time load data script is ran, so no need to worry about old data bieng the only kind avaliable, all data within the range should be availible even when its brand new.

**Numbers / results:**
- 117,956 rows x 150 columns, 7 seasons (2020-2026), 4,301 unique players
- Latest 2026 week available: 5
- Regular-season rows: 112,865 · playoff rows: 5091
- Rows with a blank `player_id` (junk rows): 136

**What surprised me:**
- Low amount of blank player_id rows so most rows of data that fit the filter of offensive players can provide some value.

**Problems or open questions:**
- The table includes playoffs and every position. Decide during cleaning whether to keep regular season only (record in DECISIONS.md)

**Saved for the poster:**
- Nothing yet (raw data only)

**Next step:**
- Download schedules, snap counts and the player ID table

### 2026-10-08 · (Name) · Phase 1

**What I did:**

- Explored `stats_player_week_2026.csv` (weeks 1-4): 4,449 rows, 150 columns; 1,436 rows are QB/RB/WR/TE (474 players)

**Numbers / results:**
- Average PPR points per game differs a lot by position; QBs score most
- Last week's points correlate about **0.5** with this week's points (0.52, measured on half-PPR before we switched to PPR; recheck with PPR)

**What surprised me:**
- Each week has one blank junk row with no player ID; a season-totals file had a fake "64 games" row

**Problems or open questions:**
- 4 weeks isn't enough to train on; need 2020-2025

**Saved for the poster:**
- (add screenshots of the exploration outputs)

**Next step:**
- Download 2020-2025 weekly stats, schedules and snap counts

### 2026-10-29 · Faraz · Phase 1

**What I did:**
- Created `notebooks/02_clean_data.ipynb`: loads the raw files, cleans player stats, merges in snap counts and schedules
- Saved the combined table to `data/processed/player_weeks_clean.parquet` (gitignored)

**Numbers / results:**
- 119,897 raw rows → **37,313** QB/RB/WR/TE regular-season player-weeks (WR 15,606 · RB 9,764 · TE 7,800 · QB 4,143)
- Snap merge: 185 snap rows had no GSIS ID match; **38 player-weeks (0.1%) have no snap data**
- Schedule merge: 0 rows missing Vegas lines; home share 0.50 (team abbreviations match)
- Final table: 37,313 rows × 161 columns, with new columns `is_home`, `rest_days`, `implied_total`

**What surprised me:**
- The ID translation worked almost perfectly; only 0.1% of player-weeks are missing snap %

**Problems or open questions:**
- Decide at modeling time whether to drop or fill the 38 rows with no snap %

**Saved for the poster:**
- Row counts for the Data section table (above)

**Next step:**
- Exploration notebook: points by position, last week vs this week correlation, leakage columns

### 2026-10-29 · Faraz · Phase 1 (exploration)

**What I did:**
- Created `notebooks/03_exploration.ipynb` using the clean table (37,313 player-weeks)
- Summarized PPR points by position and made a box plot
- Measured how well last week's points predict this week's, by position
- Checked same-week stats for target leakage

**Numbers / results:**
- Average PPR points per game: QB 13.9 · RB 7.8 · WR 7.4 · TE 5.5
- Spread (std): QB 9.2 · RB 8.0 · WR 7.8 · TE 6.0
- 25% of RB/WR/TE games score 1.4 points or less (lots of backups with tiny roles)
- Highest single games: WR 57.9 · RB 56.2 · QB 51.9 · TE 45.6
- Last-week correlation (within season): QB 0.40 · RB 0.50 · WR 0.44 · TE 0.40 · all 0.50

**What surprised me:**
- QBs are the LEAST predictable from last week (0.40), even though they score the most
- RBs are the most predictable (0.50), likely because workload (carries, snaps) is steady
- Pooling all positions inflates correlations (all = 0.50) and can flip signs: interceptions correlate +0.13 with points only because QBs throw them and QBs score more. Always report by position.

**Key insight:**
- A player's role drives his points: WR1s outscore WR3s, starting QBs outscore backups. Past snap % and target share show the role a player has had; depth charts show role changes before they happen (e.g. a backup moving up after an injury). Role features should be the core of the model.

**Leakage:**
- All same-week stats are leakage because fantasy points are calculated from them: `fantasy_points` (0.96), same-week snaps (0.63), receiving yards (0.59), receptions (0.58), targets (0.50), etc. Only past values (last week, 3-week averages) can be features.

**Problems or open questions:**
- Should we filter out low-role players (e.g. under ~25% snap share) or report them separately?
- Depth charts must be the version known before each game, to avoid leakage

**Saved for the poster:**
- `reports/figures/ppr_by_position.png`
- Last-week correlation table (above)

**Next step:**
- Feature engineering: 3-week averages of points, targets, snap %, carries