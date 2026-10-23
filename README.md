# CM4044-Group-14-Project-2
# MP2-01 One Molecule, Many Faces: Does the Representation Matter?

**CM4044 Group 14, Project 2**
Members: Wan Jeng Hor, Yap Kang Lin, Serene Ann Bravin

**Research question:** For the same property-prediction task, how much of the performance comes from the molecular representation rather than the model?

We compare three representations of the same molecules on identical curated data, identical scaffold splits and identical seeds:

| Representation | Model | Role |
|---|---|---|
| RDKit 2D descriptors (217) | XGBoost | Baseline |
| SMILES characters | LSTM | Main model |
| Molecular graph | GCN (PyTorch Geometric) | Main model |
| SELFIES tokens | LSTM | Stretch |
| Mol2Vec embedding | XGBoost | Stretch |

Primary dataset: **ESOL** (aqueous solubility, logS). Confirmation dataset: **Lipophilicity** (logD).

## Repository layout

```
.
├── README.md
├── requirements.txt
├── notebooks/
│   ├── ESOL_CM4044_Prj2.ipynb
│   └── LIPO_CM4044_Prj2.ipynb
├── data/
│   ├── esol_curated.csv
│   ├── esol_scaffold_splits.csv      # split manifest: row_idx, smiles, seed, split
│   ├── esol_curation_log.json
│   ├── lipo_curated.csv
│   ├── lipo_scaffold_splits.csv
│   └── lipo_curation_log.json
├── models/
│   ├── MODEL_CARD_xgb.md
│   ├── MODEL_CARD_lstm.md
│   ├── MODEL_CARD_gcn.md
│   ├── xgb_models/                   # xgb_<dataset>_seed_<seed>.json
│   ├── lstm_checkpoints/             # lstm_seed_<seed>.pt (weights + vocab)
│   └── gcn_checkpoints/              # gcn_<dataset>_seed_<seed>.pt (weights + target mean/std)
└── results/                          # per-seed metrics, predictions, error analysis, ensemble CSVs
```

## Setup

Python 3.10+ (tested on Google Colab). Install dependencies:

```
pip install -r requirements.txt
```

A GPU is optional. The notebooks run on CPU (ESOL has only 1,116 molecules after curation).

## How to reproduce

1. Open `notebooks/ESOL_CM4044_Prj2.ipynb` in Google Colab (or Jupyter).
2. Run all cells from top to bottom. The notebook downloads the dataset from the DeepChem S3 bucket and the pretrained Mol2Vec model from the Mol2Vec GitHub repository, so internet access is needed.
3. Outputs are written to `outputs/` (curated data, split manifest, per-seed metrics, predictions, checkpoints, error analysis, ensemble results).
4. Repeat with `notebooks/LIPO_CM4044_Prj2.ipynb`.

Approximate run time on Colab CPU: TODO (fill in after your last run).

## Reproducibility

- **Seeds:** 42, 43, 44, 45, 46. Each seed sets Python `random`, NumPy and PyTorch (CPU and CUDA) via `set_seed`, and also controls the scaffold split and the XGBoost `random_state`.
- **Splits:** randomised scaffold split (80/10/10) with Murcko scaffolds. Splits are generated once, saved to `data/*_scaffold_splits.csv`, and reloaded by every model, so all models see exactly the same train/validation/test molecules for each seed.
- **No leakage:** vocabularies, imputers and target scaling are fitted on training molecules only. Early stopping and checkpoint selection use the validation set; the test set is used only for the final report.
- **Curation:** see `data/*_curation_log.json` (parse, keep largest fragment, neutralise, canonical SMILES, merge duplicates, drop groups with conflicting labels, std > 0.5).
- **Note:** GPU runs of PyTorch models may differ slightly from CPU runs because some CUDA operations are non-deterministic.

## Results (ESOL, test set, mean over 5 seeds, scaffold split)

| Model | RMSE | MAE | R² |
|---|---|---|---|
| XGB-RDKit (baseline) | 0.811 ± 0.113 | 0.617 ± 0.070 | 0.845 ± 0.042 |
| GCN | 0.928 ± 0.101 | 0.725 ± 0.054 | 0.797 ± 0.057 |
| LSTM-SMILES | 1.026 ± 0.138 | 0.772 ± 0.094 | 0.754 ± 0.057 |
| LSTM-SELFIES (stretch) | 1.007 | 0.784 | 0.764 |
| XGB-Mol2Vec (stretch) | 1.090 | 0.854 | 0.724 |

(± is the sample standard deviation across seeds. Stretch rows show means only.)

Lipophilicity results: TODO (paste from the LIPO notebook).

## Deliverables map

| Brief item | Where |
|---|---|
| 1. Code | `notebooks/`, `README.md`, `requirements.txt` |
| 2. Trained models | `models/` (weights for baseline and main models, plus model cards) |
| 3. Curated dataset | `data/` (curated CSVs, split manifests, curation logs) |

## Known limitations

- ESOL and Lipophilicity are small, single-source benchmarks; scaffold splits give high variance between seeds (see the standard deviations above).
- The LSTM maps unseen test tokens to the padding index (no separate `<unk>` embedding).
- Results are from five seeds with one hyperparameter setting per model; no extensive tuning was done.
