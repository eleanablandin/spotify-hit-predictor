 Spotify Hit Predictor

Can a song's hit potential be predicted from its audio features alone, before release?

Over 99,000 tracks are uploaded to Spotify every day, yet 87% never reach 1,000 streams. Editorial teams curate high-impact playlists manually, but the volume makes it impossible to evaluate every track — songs with hit potential go unnoticed. This project tests whether audio features can identify them.

**[View the interactive dashboard →](https://public.tableau.com/app/profile/eleana.pena/viz/SPOTIFYHITpredictor/Dashboard1)**

---

## Hypotheses

The project was built around one central hypothesis and three testable sub-hypotheses.

> **Central:** A hit, regardless of genre, shares a measurable audio pattern that generates an emotional response and encourages repeated listening.

| | Hypothesis | Verdict |
|---|---|---|
| **H1** | A measurable audio pattern exists among hits | Partially supported |
| **H2** | Genre is secondary to the audio itself | **Not supported** |
| **H3** | Combined features predict better than individual variables | Not supported by the approach tested |

---

## Key Findings

**No single audio feature predicts popularity.** Across 89,023 unique tracks, every correlation sits below 0.13 — danceability, the strongest candidate, reaches only 0.065.

**Genre carries the signal audio doesn't.** Adding `track_genre` to the model more than doubled every metric (F1: 0.153 → 0.327, AUC 0.846), contradicting H2. Yet audio features still account for 73% of the model's total decision weight — genre fills a specific gap rather than dominating.

**The top predictor turned out to be a proxy.** `instrumentalness` ranked #1 in feature importance despite a weak correlation. Investigating it revealed why: the highest-instrumentalness genres are functional audio (study, sleep, ambient), the lowest are vocal-driven genres that dominate the hit rankings. The model had found genre information through a side door before genre was ever provided.

**The ceiling belongs to the data, not the algorithm.** Random Forest and XGBoost converged on similar performance (F1 0.327 vs. 0.301, AUC 0.846 vs. 0.881). The limitation is that audio features describe how a song sounds, not the conditions under which it succeeds.

---

## The Proposal

The deliverable is triage, not prediction. On a test set of 17,805 tracks containing 625 real hits, the model flags 1,251 tracks and captures about half of the hits within them — the editorial team reviews 7% of the catalog and finds half of what it is looking for.

With a precision of 0.245, three out of four flagged tracks are not hits. That rules the model out for allocating marketing budget, but not for allocating listening hours. It should only promote candidates for review, never filter them out.

---

## Repository Structure

```
notebooks/
  01_exploration.ipynb        Data profiling and cleaning
  02_sql_queries.ipynb        SQL exploration (SQLite)
  03_eda_visualization.ipynb  Correlation analysis and feature engineering
  04_modeling.ipynb           Random Forest, XGBoost, hypothesis testing
data/
  dataset.csv                 Raw dataset (114,000 rows)
  tracks_clean.csv            Cleaned output of notebook 01
```

---

## Method

**Cleaning** (114,000 → 112,782 rows) removed index columns, one row missing identifiers, 450 exact duplicates, and 767 rows with invalid ranges — tempo detected as zero, tracks under 30 seconds, and tracks over 10 minutes that turned out to be DJ mixes and functional audio.

**Deduplication** (112,782 → 89,023 tracks) collapsed cross-genre duplicates. 21% of rows are the same recording tagged under multiple genres; without this step, a song could appear in both the training and test sets.

**Modeling** used a stratified 80/20 split to preserve the 3.5% hit rate, threshold tuning instead of the default 0.5 cutoff, and precision/recall/F1 rather than accuracy — with this imbalance, a model that never predicts a hit still scores 96.5%.

---

## Notes on the Data

`popularity` is a live score, recalculated continuously. During deduplication, the same track appeared with values differing by one point across its genre tags, because the original extraction queried the API at slightly different moments. `GROUP BY track_id` with `MAX(popularity)` was used instead of `DISTINCT`, which failed to collapse those rows.

Spotify deprecated its audio-feature endpoints in November 2024. Only applications approved before that date retain access, so this dataset cannot be extended through the API.

---

## Tools

Python (pandas, NumPy, scikit-learn, XGBoost, matplotlib, seaborn) · SQLite · Tableau Public · Jupyter

---

## Data Source

[Spotify Tracks Dataset](https://www.kaggle.com/datasets/yashdev01/spotify-tracks-dataset) — Kaggle, CC0 1.0 Universal
