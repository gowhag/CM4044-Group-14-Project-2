# Model Card: SMILES + LSTM

## Model Overview
- Project: CM4044 Project 2 — One Molecule, Many Faces.
- Model role: main sequence-based model.
- Task: regression of aqueous solubility using ESOL.
- Input: character-level tokens from curated canonical SMILES.
- Output: predicted measured log solubility, using the ESOL target scale.
- Implementation: LSTMRegressor in [ESOL]_CM4044_Prj2.ipynb.

The implemented LSTM is unidirectional. The notebook's introductory
reference to BiLSTM does not describe the implemented architecture.

## Architecture and Hyperparameters

| Component or parameter | Value |
|---|---|
| Embedding dimension | 64 |
| Hidden dimension | 128 |
| LSTM layers | 2 |
| Direction | Unidirectional |
| Dropout | 0.2 |
| Regression head | Dropout, Linear(128,64), ReLU, Linear(64,1) |
| Batch size | 32 |
| Optimiser | Adam |
| Learning rate | 0.001 |
| Weight decay | 0.00001 |
| Loss function | Mean squared error |
| Maximum epochs | 100 |
| Early-stopping patience | 12 epochs |
| Gradient clipping | Maximum norm 5.0 |
| Random seeds | 42, 43, 44, 45, 46 |

Sequences are processed using packed sequences and their true lengths.
The final hidden state of the last LSTM layer feeds the regression head.

The vocabulary is constructed from training SMILES only. The executed
SMILES pipeline uses a vocabulary containing a padding token without
a separate unknown token. Unseen characters therefore map to the
padding index. Changing this behaviour requires retraining and
reevaluation.

## Training Data and Version
- Dataset: ESOL.
- Source:
  https://deepchemdata.s3-us-west-1.amazonaws.com/datasets/delaney-processed.csv
- Raw dataset: 1,128 entries.
- Curated dataset: 1,116 molecules.
- Training-data file: outputs/esol_curated.csv.
- Curation record: outputs/esol_curation_log.json.
- Split manifest: outputs/esol_scaffold_splits.csv.

Curation includes parsing, largest-fragment selection, neutralisation,
sanitisation and canonicalisation. Duplicate targets are averaged
unless the group's sample standard deviation exceeds 0.5, in which
case the group is removed. No subsampling is performed.

The original download date and dataset hashes were not recorded.
Preserve the actual curated CSV and record its SHA-256 hash to
identify the exact training-data version.

## Training and Evaluation
The model uses the same saved Murcko-scaffold assignments as the
other core pipelines.

Recorded split sizes per seed:
- Training: 892 molecules.
- Validation: 111 molecules.
- Test: 113 molecules.

Training batches use a seeded PyTorch generator. The checkpoint
with the lowest validation RMSE is restored before test evaluation.

## Recorded Test Performance

Mean ± sample standard deviation across five seeds:

| Metric | Value |
|---|---|
| RMSE | 1.0257 ± 0.1384 |
| MAE | 0.7718 ± 0.0936 |
| R² | 0.7542 ± 0.0571 |

Standard deviations use ddof=1. These are recorded notebook outputs,
not results of a newly verified run.

Results:
- outputs/lstm_esol_5_seeds.csv
- outputs/lstm_esol_test_predictions.csv

## Intended Use
Academic comparison of character-level SMILES learning with descriptor
and graph pipelines for ESOL solubility prediction.

Inference requires the saved training vocabulary, model weights,
canonicalisation procedure and sequence-processing implementation.

## Limitations
- The small dataset limits learning directly from character sequences.
- Sequence ordering influences the representation.
- Unseen characters map to padding in the current implementation.
- The model uses no SMILES augmentation or explicit 3D information.
- Performance outside the ESOL chemical domain is unvalidated.
- Changing representation and model together prevents
  representation-only attribution.

## Saved Model Status
The notebook writes per-seed checkpoints to:
outputs/lstm_checkpoints/lstm_seed_42.pt
through
outputs/lstm_checkpoints/lstm_seed_46.pt

Each checkpoint contains:
- model_state_dict
- vocab
- seed
- best_epoch

Verify that these files exist and include them in the trained-model
submission. Instantiate the documented architecture and restore the
corresponding vocabulary and weights for inference.

## Reproducibility
Seeds: 42, 43, 44, 45 and 46.

The notebook records RDKit version 2026.09.1. Original PyTorch and
other package versions were not recorded.

Preserve the environment, data and split manifest. Fixed seeds do
not guarantee identical results across CPU/GPU hardware or different
software versions.
