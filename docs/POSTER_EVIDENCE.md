# Poster Evidence Tracker

Every section of the final poster and the evidence it needs. Fill in the file path when an item is saved, so poster week is assembly, not scrambling. Checkpoint numbers match `ROADMAP.md`.

## Title & research question
| Item | Checkpoint | Owner | Saved at | Done |
|---|---|---|---|---|
| One-sentence research question | 0 | Both | `docs/RESEARCH_LOG.md` (Research questions) | [x] |
| Starting hypothesis | 0 | Both | `docs/RESEARCH_LOG.md` (Starting hypothesis, H1-H4) | [x] |
| 3-sentence summary (question, method, result with its number) | 7 | Both | | [ ] |

## Data
| Item | Checkpoint | Owner | Saved at | Done |
|---|---|---|---|---|
| Data sources (nflverse player stats, schedules, snap counts) | 1 | A | `data/DATA_DICTIONARY.md` (sections 1-2) | [x] |
| Table: seasons, rows, players, columns after cleaning | 1 | A | `reports/tables/` (numbers so far in `docs/RESEARCH_LOG.md`, 2026-10-29 Phase 1 entry; table not saved yet) | [ ] |
| Figure: PPR points distribution by position | 1 | B | `reports/figures/ppr_by_position.png` | [x] |
| Figure: last week vs this week scatter + correlation | 1 | B | `reports/figures/` (correlations in `docs/RESEARCH_LOG.md`, 2026-10-29 exploration entry; figure not saved yet) | [ ] |
| Data issues found (missing values, junk rows) | 1 | A | `docs/RESEARCH_LOG.md` (2026-10-15, 2026-10-29 entries); `docs/DECISIONS.md` (missing snap counts) | [x] |

## Methods
| Item | Checkpoint | Owner | Saved at | Done |
|---|---|---|---|---|
| Table: features, groups, formulas, positions | 2 | A | `data/DATA_DICTIONARY.md` (section 3, Features) | [x] |
| Pipeline diagram (data → features → model → weekly predictions) | 2 | A | `reports/figures/` | [ ] |
| How leakage was prevented | 2 | B | `docs/RESEARCH_LOG.md` (leakage columns + Jefferson check); `data/DATA_DICTIONARY.md` (section 5) | [x] |
| Train/test/live split (2020-24 / 2025 / 2026) | 2 | B | `docs/DECISIONS.md` (2026-11-19 Train/test split) | [x] |
| Models and baselines used | 3 | B | `notebooks/05_baselines_models.ipynb`; `docs/DECISIONS.md` (2026-11-20 Final model) | [x] |

## Results
| Item | Checkpoint | Owner | Saved at | Done |
|---|---|---|---|---|
| **Main table: MAE by model and position** | 3 | B | `reports/tables/model_comparison.csv`, `reports/tables/model_comparison_wide.csv` | [x] |
| Figure: MAE by position, baseline vs best model | 3 | B | `reports/figures/mae_by_position.png` | [x] |
| Figure: live 2026 MAE week by week, model vs baseline | 4 | B | `reports/figures/` | [ ] |
| Figure: top features by position | 5 | B | `reports/figures/` | [ ] |
| Table: with vs without injury features | 5 | A | `reports/tables/` | [ ] |
| *(Stretch)* Breakout detector precision | 6 | Both | | [ ] |

## Discussion & limitations
| Item | Checkpoint | Owner | Saved at | Done |
|---|---|---|---|---|
| Hypothesis vs actual results | 5 | Both | `docs/RESEARCH_LOG.md` | [ ] |
| 2-3 player examples (big hits and misses, with reasons) | 5 | B | | [ ] |
| Where the model fails (error analysis) | 5 | B | | [ ] |
| 3 limitations | 7 | Both | | [ ] |
| 3 future directions | 7 | Both | | [ ] |

## Gala extras
| Item | Owner | Done |
|---|---|---|
| QR code to the GitHub repo | A | [ ] |
| 60-second pitch, practiced | Both | [ ] |
| Answers to likely questions | Both | [ ] |
| Live demo (optional) | A | [ ] |
