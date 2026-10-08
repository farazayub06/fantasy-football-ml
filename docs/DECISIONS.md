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

### (to decide) · Train/test split
### (to decide) · Handling early-season weeks with little history
### (to decide) · Handling missing snap counts
