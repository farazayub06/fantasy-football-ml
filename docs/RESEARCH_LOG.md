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

### 2026-11-26 · Preston · Phase 5 (feature importance: how we tested)

**Goal:** Measure which features and feature groups the model relies on, by position, to test H2 and H3, without using the 2025 test season.

**Data:**
- Models trained on 2020-2023, importance measured on 2024 (the tuning/validation season). 2025 is untouched, because tomorrow's feature changes are decided from these results, and decisions must not come from the test set
- Same rows as all other results: only games with history (`history_flag == "ok"`), played games only (`is_upcoming == 0`)
- Training uses all players; importance is measured on real-role players only (`offense_pct_last3` ≥ 0.25), the headline group

**Model:** Identical to the final model: linear regression, one per position, blanks filled with the training median plus "was missing" flags; features that are 100% blank for a position are skipped (EPA/att for non-QBs). Using the same model ensures we measure the importance of the model we actually report. Check: baseline 2024 MAEs matched notebook 05 (QB 6.27, RB 5.49, TE 3.86, WR 5.22).

**Method: permutation importance**
1. Score the model on 2024 (MAE)
2. Shuffle one feature's values randomly across rows, so it becomes noise but keeps its normal range of values
3. Predict again and score; importance = shuffled MAE − normal MAE (points of accuracy lost)
4. Repeat 10 times with different shuffles and average (fixed random seed 42, so results are reproducible); the std shows how stable the estimate is
- No retraining: only inputs are scrambled, so the result describes what this trained model depends on

**Group importance:** Shuffling one feature underestimates correlated features, because related features cover for it (e.g. targets, receptions, WOPR). So we also shuffled whole groups at once, keeping each row's group values together: history (3 features), usage (9), production (4), efficiency (4), matchup/context (6). Every feature is in exactly one group (checked in code). This is the version used to judge H2.

**Why permutation importance instead of regression coefficients:** coefficients depend on each feature's scale (snap % is 0-1, passing yards in the hundreds), so they can't be compared directly; permutation importance is in one unit for every feature, points of MAE.

**Limitations:**
- Importance is measured on the 2020-2023 model, not the final model trained on 2020-2024 (expected to be very similar)
- Shows what the model uses, not what causes fantasy points
- Values near 0 can be slightly negative from random shuffle noise; treat anything within about one std of 0 as "not used"

### 2026-11-26 · Preston · Phase 5 (feature importance: results)

**What I did:**
- Created `notebooks/06_feature_importance.ipynb`: permutation importance for each feature and for each feature group, per position, on 2024 real-role players (models trained on 2020-2023)
- Saved `reports/tables/feature_importance.csv` and `reports/tables/group_importance.csv`

**Numbers / results (MAE increase in PPR points when scrambled):**

Group importance:

| Group | QB | RB | TE | WR |
|---|---|---|---|---|
| usage | 0.157 | 0.446 | 0.987 | 1.023 |
| history | 0.128 | 0.438 | 0.151 | 0.362 |
| production (yards/TDs/INTs) | 0.155 | 0.299 | 0.002 | -0.002 |
| matchup/context | 0.167 | 0.085 | 0.041 | 0.016 |
| efficiency | 0.094 | 0.004 | 0.013 | 0.014 |

Top features by position:
- QB: season avg (0.182), rushing yards (0.164), carry share (0.098), carries (0.091), receptions (0.088), implied total (0.079), snap % (0.069), yards per carry (0.047), EPA/att (0.041), last-3 avg (0.034)
- RB: season avg (0.527), carry share (0.376), rushing yards (0.273), snap % (0.160), targets (0.154), last game (0.125), carries (0.105), implied total (0.064)
- TE: season avg (0.379), snap % (0.228), targets (0.134), receptions (0.098), WOPR (0.078), carry share (0.077), air yards share (0.039)
- WR: season avg (0.668), WOPR (0.262), snap % (0.189), receptions (0.130), target share (0.064), last-3 avg (0.054), last game (0.028)

**Hypotheses:**
- H2 (usage > touchdowns/efficiency): **supported for WR, TE and RB.** Usage is the top group (WR 1.02, TE 0.99, RB 0.45, tied with history), and efficiency is near zero (≤ 0.014). **Not supported for QB:** all five groups are between 0.09 and 0.17.
- H3 (different top features by position):
  - WR: supported (WOPR, i.e. target and air-yards share, plus snap %)
  - RB: mostly supported (carry share, rushing yards, snap %; implied total only #8)
  - TE: partly supported (snap % and targets lead; air yards share small)
  - QB: not as predicted. Rushing (yards, carry share, carries) matters most after season average; implied total #6, EPA/att #9

**What surprised me:**
- QB rushing is the most important QB signal after season average
- Season average beats last-3 average at every position; recent form adds little once the season average is known
- In the single-feature table, usage features each look small, but as a group they are the largest. They're correlated and cover for each other when only one is scrambled

**Problems or open questions:**
- `receptions_last3` ranks #5 for QBs (0.088), which is suspicious since QBs rarely catch passes. Check before trusting it
- Efficiency features are near zero outside QB: candidates to drop (test on 2024)
- Importance here is for the 2020-2023 model on 2024, not the final model trained through 2024

**Saved for the poster:**
- `reports/tables/feature_importance.csv`, `reports/tables/group_importance.csv`

**Next step:**
- Top-10 features figure (one panel per position), then error analysis (H4)
- v3 candidates from this step: QB designed-run/rush-share feature; test dropping efficiency features

### 2026-11-25 · Preston · Phase 4 (week 12 predictions)

**What I did:**
- Retrained linear regression (one model per position) on every played game, 2020 through 2026 week 11
- Predicted week 12 for every player who played in the last 3 weeks; saved `predictions/2026/week_12.csv` with our prediction, the Baseline 2 (last-3 average) prediction, and a real-role flag
- Committed at Wed Nov, 25 at 7:46 pm ET (before the first week 12 game, Wed 8:00 pm ET)

**Numbers / results:**
- 

**What surprised me:**
- 

**Problems or open questions:**
- Some predicted players may end up inactive; only players who actually play will be scored
- Live model trains on 2026 games too (updates Decision 1's "retrain on 2020-2025"); add to DECISIONS.md

**Saved for the poster:**
- `predictions/2026/week_12.csv` (first live prediction; commit timestamp proves it was made in advance)

**Next step:**
- After week 12 finishes: score it (MAE by position, model vs Baseline 2, real-role and all players), start `reports/tables/live_season_record.csv`, note the 3 biggest misses
- Before week 13: move the weekly steps into `src/pipeline/weekly_update.py`

### 2026-11-25 · Preston · Phase 4 prep (upcoming-week rows)

**What I did:**
- Refreshed data (notebooks 01 → 02) through 2026 week 11
- Added a cell to `04_features` (after cell 0) that adds a placeholder row for each player's next game: finds the next unplayed week automatically (earliest 2026 regular-season week with no final score), takes every player who played in the last 3 weeks, attaches his team's upcoming game (opponent, home/away, rest days, Vegas total, spread, implied total), and marks it `is_upcoming = 1`
- The existing feature cells fill these rows automatically, because every feature uses only earlier games (`shift(1)`)
- Added `is_upcoming` to the saved identifier columns

**Numbers / results:**
- Predicting week 12 · 457 upcoming player-games added (QB 51 · RB 115 · WR 188 · TE 103)
- Spot check: upcoming rows have `last3`, `implied_total` and opponent features filled; `fantasy_points_ppr` blank (not played yet)

**What surprised me:**
- No new feature code was needed; the shift-based features handle future games on their own

**Problems or open questions:**
- Players who end up inactive or injured still get a prediction; only players who actually play will be scored
- Must run on Tuesday/Wednesday after Monday Night Football; mid-weekend, part of the week is already played

**Saved for the poster:**
- Nothing new yet (this is groundwork for the live 2026 test)

**Next step:**
- Notebook 05: retrain on all played games, predict the upcoming week, save `predictions/2026/week_12.csv`, commit before Wednesday's (Thanksgiving schedule) first game

### 2026-11-20 · Preston · Phase 3 (baselines and models)

**What I did:**
- Created `notebooks/05_baselines_models.ipynb`: dropped first games of each season (`history_flag == "ok"`), split by season (train 2020-2024, test 2025), scored all models with one function on the same rows
- Baselines: last game's points; last-3-game average
- Models (one per position): linear regression (median fill + missing flags), gradient boosting (default settings)
- Checked model choice on 2024 (trained on 2020-2023) before trusting the 2025 result
- Saved `reports/tables/model_comparison.csv`, `model_comparison_wide.csv`, `reports/figures/mae_by_position.png`

**Numbers / results (2025 test, MAE in PPR points, real-role players):**
- Baseline 1 (last game) 6.55 · Baseline 2 (last 3) 5.55 · Gradient boosting 5.21 · **Linear regression 5.11**
- Linear vs Baseline 2: QB 7.38 → 6.65 (−9.9%) · RB 6.27 → 5.78 (−7.8%) · WR 5.39 → 5.01 (−7.1%) · TE 4.26 → 3.91 (−8.2%) · ALL −7.9%
- All players: Baseline 2 4.72 → linear 4.44
- 2024 check (real-role): linear beat boosting at every position (QB 6.27 vs 6.58, RB 5.49 vs 5.55, TE 3.86 vs 3.97, WR 5.22 vs 5.27)

**What surprised me:**
- Linear regression beat gradient boosting everywhere; most of the signal is simple and additive
- QBs improved the most (−9.9%), possibly from the passing / EPA-per-attempt features
- Averaging 3 games beats last game alone by about 15%

**Problems or open questions:**
- Gradient boosting was untuned; tuning on 2024 could close the gap (Phase 5 improvement)
- Gains are modest (~8%) because fantasy points are noisy; check that the gain isn't luck (how often the model beats the baseline on the same game)
- Gradient boosting crashed on features that are 100% blank for a position (EPA/att for non-QBs); fixed by skipping those columns per position

**Saved for the poster:**
- `reports/tables/model_comparison_wide.csv` (main results table)
- `reports/figures/mae_by_position.png` (Results figure)

**Next step:**
- Gate passed (model beats Baseline 2 at every position): retrain on 2020-2025 and start live predictions (Phase 4)

### 2026-11-16 · Preston · Phase 2 (features v2)

**What I did:**
- Replaced total EPA with EPA per attempt over the last 3 games (`epa_per_att_last3`): sum of EPA ÷ sum of attempts; blank under 20 attempts (see DECISIONS.md)
- Added history features: last game's points (`fantasy_points_ppr_last1`, needed for Baseline 1) and season-to-date average (`fantasy_points_ppr_season`)
- Added efficiency features over the last 3 games: catch rate, yards per catch, yards per carry (sum ÷ sum, blank when nothing to divide by)
- Added team pass attempts per game (`team_att_avg`, earlier games this season only) and the Vegas game total (`total_line`)
- Reorganized `feature_cols` into groups: history, usage, role, passing, efficiency, team/opponent, game context
- Feature set v2: 26 features, saved to `data/processed/features.parquet`

**Numbers / results:**
- `epa_per_att_last3` for QBs: mean 0.037, range -1.655 to 0.851;
- Leakage check (Justin Jefferson, 2020 week 6): last1 = 5.3 (week 5 score 5.3), last3 = 16.7 , season = 12.4 
- `team_att_avg`: mean 33.7 pass attempts per game

**What surprised me:**
- ___

**Problems or open questions:**
- Linear regression can't use blank values; fill them (e.g. EPA per attempt → 0) before Phase 3
- Still to decide with Faraz: early-season weeks with no history; whether to score only real-role players
- A backup QB who suddenly starts has almost no history, so expect big misses there (H4)

**Saved for the poster:**
- Feature list by group (for the checkpoint 2 feature table)

**Next step:**
- Decide the open questions at the weekly sync, then start baselines and first models (Phase 3)

### 2026-10-29 · Preston · Phase 2 (Things to consider)

**What I did:**
- Did a run through of our code base to see where things stand. Before progressing to phase 3. 

**Where things stand:**
- In phase 1 we loaded and cleaned the data, filtered it to the positions we wanted, and merged player id's and snap counts. Then we explored the data and founds correlations between a players previous week points and the amount of points they had the next week =. 

- In phase 2 we built 13 features using 3-game averages, an opponent features and game context. The leakage test also passed.

**What surprised me:**
- We still need to add some more features, but at the same time we dont want to over parameterize the model.

- We need to save some of the new computed metrics into parquet files.

- Roadmap features stil missing: season-to-date average, efficiency stats (catch rate, yards per catch, yards per carry), team pass attempts and game total.

**Problems or open questions:**
- Train/test split: still “to decide”.

- Early weeks: history resets every season, which is why 11% of rows are blank. One option is to carry over last season’s averages, however this can't be done for rookies.

- Which players get scored: decide this before Phase 3. A quarter of RB/WR/TE games score 1.4 points or less. Those backups are easy to predict, so they make every model’s MAE look good. Either score only real-role players (e.g. last-3 snap % ≥ 25%) or report both groups.

**Housekeeping:**
- DATA_DICTIONARY.md still lists only 1 column.

- No ROADMAP.md boxes are ticked.

- The checkpoint 1 table and the last-week scatter plot aren’t saved in reports/.


### 2026-10-29 · Faraz · Phase 2 (features)

**What I did:**
- Created `notebooks/04_features.ipynb` and saved feature set v1 to `data/processed/features.parquet` (gitignored)
- 13 features: 3-game averages of PPR points, targets, receptions, carries, snap %; 3-game averages of target share, air yards share, WOPR, carry share; opponent PPR allowed to the position (earlier weeks this season); implied team total, home/away, rest days

**Numbers / results:**
- Leakage test passed: checked Justin Jefferson 2020 by hand; each week's feature uses only earlier games (week 6: scored 39.6, feature = 16.7 from weeks 3-5)
- 4,141 rows (each player's first game of a season) have no history → blank `_last3` features
- Opponent PPR allowed per game, all players at the position combined: QB 16.9 · RB 22.3 · WR 34.1 · TE 12.2

**What surprised me:**
- Early-season opponent averages are extreme (WR range 6 to 79) because they're based on 1-2 games

**Problems or open questions:**
- How to handle week-1 rows with no history (drop, or fill with last season's averages)
- Vegas lines, home/away and rest days are known before kickoff, so they need no lagging

**Saved for the poster:**
- Feature list (above) for the Methods section

**Next step:**
- Train baselines and a first model (Phase 3)

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

