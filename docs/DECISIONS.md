# Decisions Log

Every important choice, why you made it, and what else you considered. Judges at the gala will ask "why did you do it that way?", and this file is your answer.

## Template

```
### YYYY-MM-DD · Decision title · Name

**Decision:**
**Why:**
**Alternatives considered:**
**Effect on results (if measured):**
```

---

### 2026-10-08 · Project: NFL fantasy football, PPR

**Decision:** Predict weekly PPR points for QB, RB, WR and TE (FLEX = RB, WR and TE combined).
**Why:** Rich free data (nflverse), thousands of player-weeks per season, live weekly testing during the 2026 season.
**Alternatives considered:** Premier League match outcome prediction (less detailed free data); NFL game winners (only 272 games a season).

### 2026-10-08 · Data source: nflverse

**Decision:** Use nflverse data (weekly player stats, schedules, snap counts) via `nflreadpy` or the release CSVs.
**Why:** Free, documented, updated weekly, covers 1999 to today.
**Alternatives considered:** Paid APIs; R packages `nflreadr` / `nflfastR` (same data, but we're using Python).

### 2026-10-12 · Notebooks: Jupyter in VS Code, not Google Colab

**Decision:** Run the notebooks locally in VS Code (Jupyter extension) with a shared `.venv` built from `requirements.txt`.
**Why:** Notebooks live in the repo and are committed with normal Git; no re-installing packages every session; same package versions for both partners.
**Alternatives considered:** Google Colab (free GPU we don't need, but packages reset each session and syncing notebooks back to GitHub is clumsy).

### (to decide --> Done - 11/20) · Train/test split
### (to decide --> Done - 11/20) · Handling early-season weeks with little history
### 2026-10-29 · Regular season only · Faraz

**Decision:** Keep only regular-season games; drop playoffs.
**Why:** Fantasy football is scored on the regular season, and playoff games include only some teams.
**Alternatives considered:** Keeping playoffs as extra training data.

### 2026-10-29 · Missing snap counts · Faraz

**Decision:** Leave the 38 player-weeks (0.1%) with no snap data blank for now; decide at modeling time.
**Why:** Too few to affect results; some models handle blanks on their own.
**Alternatives considered:** Dropping those rows; filling with the player's average snap %.

### 2026-11-16 · QB efficiency: EPA per attempt instead of total EPA · Preston

**Decision:** Replace `passing_epa_last3` (average total EPA over the last 3 games) with `epa_per_att_last3` = total passing EPA over the last 3 games ÷ total pass attempts over those games. Leave it blank when the player has fewer than 20 attempts in those games.

**Why:**
- Total EPA mixes volume (how many passes) with efficiency (how good each pass was). Volume is already covered by `attempts_last3`, so per-attempt EPA gives the model a clean efficiency signal instead of repeating volume.
- Sums are divided instead of averaging each game's EPA/attempt, so a 3-attempt game doesn't count as much as a 40-attempt game.
- The 20-attempt cutoff removes tiny samples (e.g. a backup going 2-for-3 with one long completion), which produce extreme, meaningless values. 20 attempts is less than one full start, so starting QBs always qualify.

**Alternatives considered:**
- Keep total EPA: simpler, but confuses volume with efficiency.
- Keep both: the two are strongly related, so the model would split importance between them and the feature-importance chart would be harder to read.
- Average per-game EPA/attempt: lets low-attempt games swing the value.
- Different cutoff (10 or 30 attempts): 20 chosen as a middle ground; can revisit if results are sensitive to it.

**Effect on results (if measured):** Not measured yet. Compare QB MAE with total EPA vs. EPA per attempt in Phase 3.

### 2026-11-19 · Train/test split · Preston

**Decision:** Split by season: train 2020-2024, test 2025, live predictions 2026. The 2025 test set is scored once, after models are chosen. For tuning, train on 2020-2023 and validate on 2024. Before live predictions, retrain on 2020-2025; the reported benchmark stays the 2025 test score.

**Why:** A random split lets the model learn from games after the ones it's tested on (leakage) and doesn't match real use, where only the past is known. Tuning on 2025 would make the test score too optimistic.

**Alternatives considered:** Random 80/20 split (leaks future data); rolling week-by-week evaluation (more realistic but more work; possible later).

### 2026-11-19 · Which players are scored · Preston

**Decision:** Report MAE for (1) all players and (2) real-role players, defined as `offense_pct_last3` ≥ 0.25. Real-role MAE is the headline number.
**Why:** About a quarter of RB/WR/TE games are backups scoring ~1 point, which are easy to predict and make every model look better. Real-role players are the ones fantasy managers actually decide between. The filter uses past snap %, so it's known before the game (no leakage).
**Alternatives considered:** All players only (flattering); this week's snap % as the filter (leakage); a fantasy-points threshold (also uses information not known before the game).
**Effect on results:** Rows kept per split: ___ (from the check cell).

### 2026-11-19 · Early-season rows with no history · Preston

**Decision:** Drop each player's first game played in each season (no `last1`/`last3`/`season` values) from training and scoring. Tag every row with `history_flag`:
- `ok`: has history
- `season_opener`: first game was the team's first game
- `limitation_returning`: first game came later, player appeared in an earlier season (injury return, suspension, backup's first stat)
- `limitation_new`: first game came later, never seen before (rookie or new player)

**Why:** Baselines can't predict rows with no history either, so dropping them keeps every model graded on the same games. Tagging late starters separately lets us count them and list them as a limitation instead of hiding them. In live predictions they appear with a blank prediction and their tag.
**Caveats:** 2020 has no earlier season in our data, so all 2020 late starters are tagged `limitation_new`. "Late start" includes backups whose first recorded stat came after week 1.
**Alternatives considered:** Fill with last season's averages (no help for rookies; possible future improvement); leave blanks for gradient boosting (unfair comparison with baselines and linear regression).
**Effect on results:** Rows per tag per season: ___ (from the count cell).

### 2026-11-20 · Final model: linear regression · Preston

**Decision:** Use linear regression (one model per position; blanks filled with the training median plus missing-value flags) as the main model for live 2026 predictions.
**Why:** Chosen on the 2024 validation season (trained on 2020-2023, real-role players): linear regression beat default gradient boosting at every position (MAE QB 6.27 vs 6.58, RB 5.49 vs 5.55, TE 3.86 vs 3.97, WR 5.22 vs 5.27). The 2025 test confirms it: linear was best in every row and beat the last-3 baseline by 7.1-9.9% on real-role players. It's also simpler and easier to explain.
**Alternatives considered:** Default gradient boosting (close for RB/WR, worse for QB); tuned gradient boosting not yet tried (planned for Phase 5, tuned on 2024 only).
**Effect on results:** Real-role MAE, 2025 test: linear 5.11 vs baseline 5.55 (7.9% better).

### 2026-11-25 · Building rows for upcoming games · Preston

**Decision:** Each week, add one row per player for the next unplayed week. Players included: anyone who played in the last 3 weeks of the season, on his most recent team. Next week = the earliest 2026 regular-season week with no final score. Teams on bye get no rows. Rows are marked `is_upcoming = 1`.
**Why:** The model needs feature rows for games that haven't happened. Adding placeholder rows before the feature cells lets the same leakage-safe code (only earlier games) build their features, so live predictions use exactly the features the model was tested on.
**Limitations:** Inactive or injured players still get predictions (no injury data yet); they're excluded when scoring. A player who hasn't played in 3+ weeks (e.g. returning from injury) gets no prediction.
**Alternatives considered:** A separate feature script for future games (duplicate code that could drift from the tested features); predicting only for real-role players (we keep everyone and filter when scoring, same as the 2025 test).