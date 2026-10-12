# Fantasy Football ML

A machine learning research project that predicts weekly **PPR fantasy points** for NFL quarterbacks, running backs, wide receivers and tight ends, and tests whether ML beats simple recent-form baselines.

**Team:** Partner A (data & features) · Partner B (modeling & evaluation)

## Research questions

1. Can a machine learning model using usage, matchup and Vegas features predict weekly PPR points more accurately than a simple recent-form baseline?
2. Which features matter most, and does that differ by position?
3. Where does the model fail?

## Data

Free data from [nflverse](https://github.com/nflverse/nflverse-data/releases): weekly player stats, schedules (Vegas lines, rest days) and snap counts, 2020-2026.

- Train: 2020-2024 · Test: 2025 · Live predictions: 2026 season

## Results

*Coming soon.* Main results table and figures will go here.

## Project docs

- [Roadmap](ROADMAP.md): phases, tasks per partner, research checkpoints
- [Research log](docs/RESEARCH_LOG.md)
- [Decisions](docs/DECISIONS.md)
- [Poster evidence tracker](docs/POSTER_EVIDENCE.md)

## How to run

Notebooks run locally in **VS Code** (Jupyter notebooks), Python 3.10+.

1. Install [VS Code](https://code.visualstudio.com/) with the **Python** and **Jupyter** extensions
2. From the repo folder, create a virtual environment and install the packages:
   ```bash
   python -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```
3. Open a notebook in VS Code, pick the `.venv` kernel (top right), and run the notebooks in order (`notebooks/01_...` onward)
