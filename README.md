# DM1 Project — Board Games Dataset (a.a. 2025/2026)

Analysis of the Board Games dataset (~22k games rated by an online community):
data understanding & preparation, clustering, classification, regression and
pattern mining.

## Structure

```
dataset/     DM1_game_dataset.csv (extracted from dm1_25_26_dataset.zip)
data/        df_clean2.2.csv, produced by notebook 02
notebooks/
  01_data_understanding.ipynb
  02_data_preparation.ipynb
  03_clustering.ipynb
  04_classification.ipynb
  05_regression.ipynb
  06_pattern_mining.ipynb
```

## How to run

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/jupyter lab notebooks/
```

Run the notebooks in order: notebook 02 generates `data/df_clean2.2.csv`, used
by all the following modules. Every stochastic step uses `random_state=42`.

## Main findings

**Data understanding & preparation.** The raw table hides most of its quality
problems behind zeros and sentinel values rather than proper NaNs: `NumComments`
is always zero, the `Rank:*` columns use a "dataset length + 1" sentinel for
unranked games, `MfgAgeRec` uses 0 where `ComAgeRec` uses NaN (in the same
rows), and `LanguageEase` is out of its nominal 1–5 scale in most records.
Manufacturer and community attributes are largely duplicated
(`GameWeight`/`ComWeight` and `MfgPlaytime`/`ComMaxPlaytime` are perfectly
correlated) and the popularity counts form one strongly correlated block. The
preparation phase therefore compresses the table into a handful of engineered
features: `AvgGameweight`, `ComAvgPlaytime` (with a `log1p` version, since the
popularity and duration variables are extremely right-skewed), a composite
`PopularityScore` and a compact `CategoryLabel`.

**Clustering.** In the space of complexity, recommended age, log-duration and
popularity, board games form continuous gradients rather than well-separated
segments. K-Means with k=3 is retained as the most informative solution
(silhouette ≈ 0.32, slightly below the coarse k=2 optimum): roughly half of the
games are accessible/short, a smaller cluster groups popular mid-complexity
games, and the rest are complex/longer games with the highest recommended age.
DBSCAN degenerates into one macro-cluster containing >96% of the games under
every tested configuration — direct evidence of the absence of density islands —
while hierarchical Ward clustering retrieves the same broad profiles with weaker
separation.

**Classification.** Predicting the three-class `Rating` from gameplay and
popularity features, the tuned Decision Tree (max depth 8, pre-pruned via
randomized search) is the best model with accuracy ≈ 0.65, ahead of KNN (≈ 0.58)
and Gaussian Naive Bayes (≈ 0.53). Errors concentrate between adjacent rating
levels — the `Medium` class overlaps with both neighbours — and the moderate
ceiling suggests the available features capture only part of what drives the
community rating. The tree splits first on complexity, publication year and
popularity.

**Regression.** Game complexity alone explains about half of the variance of
log-playtime (simple linear regression, R² ≈ 0.48); adding the community age
recommendation raises it to ≈ 0.53. Non-linear models (KNN, pruned Decision
Tree) plateau around R² ≈ 0.55: playtime is only partially explained by
complexity, age, popularity and player counts. The tree complexity curves show
the classic overfitting picture, with test R² peaking around depth 5–6.

**Pattern mining.** With games encoded as transactions of discretized items
(only positive binary properties, multi-label categories preserved), Apriori and
FP-Growth agree on every threshold. Frequent patterns confirm plausible domain
profiles (light ∧ short, family audience ∧ medium complexity) plus more specific
ones: the strongest rule links long, pre-2006, low-capacity games to the War
category (lift ≈ 5); recent long games are strongly associated with high ratings,
older light low-popularity games with low ratings. A compact set of ~20 rating
rules mined on a training split covers about half of the held-out games and
beats the majority baseline by a wide margin (≈ 0.58 vs 0.44 overall accuracy),
showing that the extracted associations can be operationalized, not just
described.
