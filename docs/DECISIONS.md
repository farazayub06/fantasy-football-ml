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

### (to decide) · Train/test split
### (to decide) · Handling early-season weeks with little history
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
