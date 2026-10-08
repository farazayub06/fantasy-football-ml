# Research Log

Both partners add an entry at the end of **every** work session, newest at the top. Five minutes is enough. These entries become the raw material for the poster, so write down numbers and surprises while you still remember them.

## Research questions

1. Can a machine learning model using usage, matchup and Vegas features predict weekly PPR points more accurately than a simple recent-form baseline?
2. Which features matter most, and does that differ by position?
3. Where does the model fail?
4. *(Stretch)* How does it compare with expert consensus projections?

## Starting hypothesis

*(Write this in Phase 0, before seeing any results.)*

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
