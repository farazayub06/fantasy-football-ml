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
