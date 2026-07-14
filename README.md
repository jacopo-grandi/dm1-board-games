# DM1 Project — Board Games Dataset (a.a. 2025/2026)

Analisi del dataset Board Games (~22k giochi): data understanding & preparation,
clustering, classificazione, regressione e pattern mining.

## Struttura

```
dataset/     DM1_game_dataset.csv (estratto da dm1_25_26_dataset.zip)
data/        df_clean2.2.csv, prodotto dal notebook 02
notebooks/
  01_data_understanding.ipynb
  02_data_preparation.ipynb
  03_clustering.ipynb
  04_classification.ipynb
  05_regression.ipynb
  06_pattern_mining.ipynb
```

## Esecuzione

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/jupyter lab notebooks/
```

I notebook vanno eseguiti in ordine: il 02 genera `data/df_clean2.2.csv`,
usato da tutti i moduli successivi. Tutti i passaggi stocastici usano
`random_state=42`.
