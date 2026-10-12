# Project Roadmap: Fantasy Football ML

**Goal:** Predict every QB, RB, WR and TE's **PPR fantasy points** each week, test whether machine learning beats simple baselines, and present the findings on a research poster at the gala.

**Team roles**

| | Partner A: Data & Features | Partner B: Modeling & Evaluation |
|---|---|---|
| Owns | Downloading, cleaning, merging data; building features | Baselines, models, metrics, error analysis |
| Repo folders | `data/`, `src/features/`, `notebooks/01-03` | `src/models/`, `notebooks/04-07`, `predictions/` |
| Poster sections | Data, Features, Methods | Results, Discussion, Limitations |

Replace "Partner A" and "Partner B" with your names. Both of you share the research log, the weekly sync and the final poster.

**How to read this file**

- `- [ ]` tasks are checkboxes. Tick them as you finish, or turn each phase into a GitHub Issue.
- 🔬 **Research checkpoint** = stop and save evidence for the poster. Each one says exactly what to save and where.
- Dates assume you start the week of **Oct 12, 2026** and work weekly. Shift them if your schedule differs.

---

## Research questions

Write these on the first page of the research log. Every phase answers part of them.

1. **Main question:** Can a machine learning model using usage, matchup and Vegas features predict weekly PPR points more accurately than a simple recent-form baseline?
2. Which features matter most, and does that differ by position (QB, RB, WR, TE)?
3. Where does the model fail? (Injuries, touchdowns, rookies, early-season weeks?)
4. *(Stretch)* How does the model compare with expert consensus projections?

---

## Phase 0: Setup (Week 1 · Oct 12-18)

**Both together**
- [ ] Create the GitHub repo `fantasy-football-ml` and add each other as collaborators
- [ ] Copy in this file, `docs/RESEARCH_LOG.md`, `docs/POSTER_EVIDENCE.md` and `docs/DECISIONS.md`
- [ ] Set up the folder structure (bottom of this file)
- [ ] Agree on a weekly sync time (30 min) and a rule: **every work session ends with a commit and a research log entry**
- [ ] Write the research questions into the research log

**Partner A**
- [ ] Set up VS Code with the Python and Jupyter extensions, clone the repo, create a `.venv` and `pip install -r requirements.txt` (steps in `README.md`)
- [ ] Test loading data with `nflreadpy` (`pip install nflreadpy`, then `.to_pandas()` on results)

**Partner B**
- [ ] Write the first version of `README.md`: one paragraph on the project, the research questions, how to run it
- [ ] Add a `.gitignore` that excludes `data/raw/` and `.venv/` (large files don't belong in Git)
- [ ] Do the same VS Code setup as Partner A, and add any new package you use to `requirements.txt`

> 🔬 **Research checkpoint 0: Starting point**
> - Log entry: why you chose this project, what you expect to find, your current ML experience
> - Save your **hypothesis** in writing now, e.g. "We expect usage stats like target share to matter more than touchdowns." Being right or wrong is a poster finding either way.

---

## Phase 1: Data collection & exploration (Weeks 1-2 · Oct 12-25)

**Partner A**
- [ ] Download weekly player stats for **2020-2026** (`stats_player_week`)
- [ ] Download **schedules** (one file) and **snap_counts** for 2020-2026
- [ ] Combine all seasons into one table; filter to QB, RB, WR, TE
- [ ] Remove junk rows (blank `player_id`: one appears per week)
- [ ] Write `data/DATA_DICTIONARY.md`: every column you keep, what it means, where it came from

**Partner B**
- [ ] Explore the combined data in `notebooks/02_exploration.ipynb`: `shape`, `info()`, `isnull().sum()`, `duplicated().sum()`, `value_counts()`
- [ ] Plot the distribution of weekly PPR points by position
- [ ] Measure how well last week predicts this week (correlation by position; an early half-PPR check on 2026 weeks 1-4 gave about **0.5** overall)
- [ ] List every column that would cause **target leakage** (same-week stats)

> 🔬 **Research checkpoint 1: The data**
> Save to `reports/figures/` and `reports/tables/`, then log it:
> - **Table:** number of seasons, rows, players and columns after cleaning
> - **Figure:** PPR points distribution by position (box plot or histogram)
> - **Figure:** last week vs this week scatter plot, with the correlation
> - **Note:** missing values found and why they exist; the junk rows you removed
> - **Note:** the leakage columns and why they can't be features
>
> *Poster use: "Data" section and your first figure.*

---

## Phase 2: Feature engineering (Weeks 3-4 · Oct 26-Nov 8)

**Partner A** builds every feature in `src/features/build_features.py`. **Every feature uses previous weeks only.**
- [ ] History: PPR points, last-3-week average and season-to-date average
- [ ] Usage: targets, receptions, carries, target share, rush share, snap %
- [ ] Opportunity: air yards share, WOPR
- [ ] Efficiency: catch rate, yards per catch, yards per carry, QB EPA per attempt
- [ ] Opponent: PPR points and yards allowed to each position (season to date, before this week)
- [ ] Game context: home/away, rest days, game total, implied team total (from schedules)
- [ ] Team: team pass attempts per game
- [ ] Save the final table to `data/processed/features.csv`

**Partner B**
- [ ] Write a **leakage test**: for a few players, check by hand that each feature only uses earlier weeks
- [ ] Decide how to handle early-season weeks with little history (e.g. drop weeks 1-2, or fill with last season's averages); record it in `docs/DECISIONS.md`
- [ ] Set up the train/test split: **train 2020-2024, test 2025, live 2026**. Never a random split.

> 🔬 **Research checkpoint 2: Features**
> - **Table:** every feature, its group, its formula, and which positions use it (this becomes a poster table)
> - **Figure:** correlation of each feature with next week's points, by position
> - **Decision log:** how you handled leakage, early weeks, and missing snap counts
> - **Log entry:** which features you expect to matter most (compare later with the results)
>
> *Poster use: "Methods" section and the feature table.*

---

## Phase 3: Baselines & first models (Weeks 5-6 · Nov 9-22)

**Partner B**
- [ ] Baseline 1: **last week's points**
- [ ] Baseline 2: **last-3-week average**
- [ ] Model 1: linear regression
- [ ] Model 2: random forest or gradient boosting (e.g. `HistGradientBoostingRegressor`)
- [ ] Score all four on the 2025 test season with **MAE** (average points missed), by position
- [ ] Save the results to `reports/tables/model_comparison.csv`

**Partner A**
- [ ] Fix any data problems Partner B finds
- [ ] Build the weekly pipeline script `src/pipeline/weekly_update.py`: download newest data, rebuild features, ready for predictions
- [ ] Add the FLEX ranking: combine RB, WR and TE predictions, sorted by predicted points

> 🔬 **Research checkpoint 3: Does ML beat the baseline?** *(the core result)*
> - **Table:** MAE of every model and baseline, by position. **This is the main table on your poster.**
> - **Figure:** bar chart of MAE by position, baseline vs best model
> - **Note:** how much better (or worse) the model is, in points and in percent
> - **Log entry:** surprises; if the model *didn't* beat the baseline, write down why you think so (that's a valid finding)
>
> *Poster use: "Results" section.*

**Gate:** the best model beats the last-3-week baseline on 2025 before you go live. If not, spend one more week on features.

---

## Phase 4: Live predictions (Week 7 to end of NFL regular season · Nov 23-Jan 3)

Runs **every week** alongside Phases 5 and 6.

**Partner A (Monday night / Tuesday)**
- [ ] Run the weekly pipeline after Monday Night Football
- [ ] Commit the week's predictions to `predictions/2026/week_XX.csv` **before Thursday's game** (the commit timestamp proves you predicted in advance)

**Partner B (after the games)**
- [ ] Score last week's predictions: MAE by position, model vs baseline
- [ ] Add the week to `reports/tables/live_season_record.csv`
- [ ] Note the 3 biggest misses and why they happened (injury? touchdown spike? blowout?)

> 🔬 **Research checkpoint 4: Weekly live record** *(every week)*
> - One research log line: week, model MAE, baseline MAE, biggest miss and its cause
> - At the end of the season: **Figure:** model vs baseline MAE week by week (line chart)
>
> *Poster use: "real-world test" figure, the most convincing evidence you have.*

---

## Phase 5: Analysis & improvement (Weeks 7-10 · Nov 23-Dec 20)

**Partner B**
- [ ] Feature importance for the best model, by position (permutation importance or SHAP)
- [ ] Error analysis: where is the model worst? By position, by week of season, by player type
- [ ] Try one improvement and measure it: tune the model, or remove useless features

**Partner A**
- [ ] Add injury features: is a teammate at the same position out? How many targets/carries were "vacated"?
- [ ] Test whether the injury features improve MAE (compare with and without)

> 🔬 **Research checkpoint 5: What matters and where it fails**
> - **Figure:** top 10 features by importance, one panel per position
> - **Table:** with vs without injury features (MAE)
> - **Examples:** 2-3 specific players where the model was very right or very wrong, with the reason
> - **Log entry:** compare the important features with your Phase 0 hypothesis
>
> *Poster use: "Discussion" section and the feature-importance figure.*

---

## Phase 6 (optional stretch): Breakout detector (Weeks 10-12 · Dec 7-Jan 3)

Only if Phases 3-5 are done.

**Partner A**
- [ ] Define a breakout (e.g. 15+ PPR points from a player averaging under 8) and label past weeks
- [ ] Add Sleeper API trending adds as a feature

**Partner B**
- [ ] Train a classifier (logistic regression, then gradient boosting); score precision and recall
- [ ] Track flagged players during the live season

> 🔬 **Research checkpoint 6:** precision of flagged breakouts, and a list of real hits and misses.

---

## Phase 7: Final results & poster (Weeks 12-14 · early-mid January, adjust to your gala date)

**Both**
- [ ] Freeze the model; rerun final numbers on the full 2026 season
- [ ] Fill every row of `docs/POSTER_EVIDENCE.md`
- [ ] Draft the poster together; each partner writes their own sections first
- [ ] Have someone outside the project read the poster and ask questions

**Partner A**
- [ ] Clean up `src/` and the data pipeline so anyone can rerun it
- [ ] Write the Data and Methods sections of the poster

**Partner B**
- [ ] Make final, poster-quality versions of all figures (large fonts, consistent colors)
- [ ] Write the Results, Discussion and Limitations sections

> 🔬 **Research checkpoint 7: The story**
> - Write a 3-sentence summary: question, method, main result with its number
> - List 3 limitations and 3 future directions
> - Update the README with the final results and figures

---

## Phase 8: Research gala prep (final week)

- [ ] Write a 60-second pitch and a 3-minute walkthrough of the poster; practice both
- [ ] Prepare answers to likely questions: *How do you know you didn't cheat (leakage)? Why not just use ESPN's projections? What would you do with more time?*
- [ ] Put a QR code on the poster linking to the GitHub repo
- [ ] Optional: a laptop or tablet showing the live predictions or dashboard

---

## Weekly routine (both partners, every week)

1. Work on your tasks; commit with clear messages (`Add opponent points-allowed feature`)
2. Write a research log entry before you close the laptop (5 minutes, template in `docs/RESEARCH_LOG.md`)
3. Save any figure or table you might want later to `reports/`, even rough ones
4. 30-minute sync: show each other what you did, update checkboxes, plan next week

---

## Repo structure

```
fantasy-football-ml/
├── README.md                  # project summary, results, how to run
├── ROADMAP.md                 # this file
├── requirements.txt           # Python packages for the notebooks
├── docs/
│   ├── RESEARCH_LOG.md        # dated entries, both partners
│   ├── DECISIONS.md           # every important choice and why
│   └── POSTER_EVIDENCE.md     # checklist mapping poster sections to saved evidence
├── data/
│   ├── raw/                   # downloaded files (gitignored)
│   ├── processed/             # cleaned tables and features
│   └── DATA_DICTIONARY.md
├── notebooks/
│   ├── 01_load_data.ipynb
│   ├── 02_exploration.ipynb
│   ├── 03_features.ipynb
│   ├── 04_baselines.ipynb
│   ├── 05_models.ipynb
│   ├── 06_feature_importance.ipynb
│   └── 07_error_analysis.ipynb
├── src/
│   ├── features/build_features.py
│   ├── models/train.py
│   └── pipeline/weekly_update.py
├── predictions/2026/          # week_XX.csv, committed before games
└── reports/
    ├── figures/
    └── tables/
```
