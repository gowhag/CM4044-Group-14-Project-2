# Model Card: Molecular Graph + GCN

## Model Overview
- Project: CM4044 Project 2 — One Molecule, Many Faces.
- Model role: main graph-based model.
- Task: regression of aqueous solubility using ESOL.
- Input: molecular graphs containing atom features and bond connectivity.
- Output: predicted measured log solubility, using the ESOL target scale.
- Implementation: GCN class using PyTorch Geometric in
  [ESOL]_CM4044_Prj2.ipynb.

## Molecular Representation
Atoms form graph nodes. Each atom has a 38-dimensional feature vector
encoding:
- Element.
- Total degree.
- Formal charge.
- Total hydrogen count.
- Hybridisation.
- Aromaticity.
- Ring membership.

Categorical features include an additional category for other values.

Each bond creates edges in both directions. The implementation uses
bond connectivity without explicit bond-type feature vectors or
3D coordinates.

## Architecture and Hyperparameters

| Component or parameter | Value |
|---|---|
| Graph convolution layers | 3 GCNConv layers |
| Hidden dimension | 128 |
| Normalisation | BatchNorm1d after each convolution |
| Activation | ReLU |
| Dropout | 0.1 |
| Pooling | Concatenated global mean and sum pooling |
| Pooled dimension | 256 |
| Regression head | Linear(256,128), ReLU, Dropout(0.1), Linear(128,1) |
| Batch size | 64 |
| Optimiser | AdamW |
| Learning rate | 0.001 |
| Weight decay | 0.0001 |
| Loss function | Mean squared error on standardised targets |
| Maximum epochs | 150 |
| Early-stopping patience | 20 epochs |
| Validation improvement threshold | 0.0001 RMSE |
| Gradient clipping | Maximum norm 5.0 |
| Random seeds | 42, 43, 44, 45, 46 |

Targets are standardised using the training-set mean and population
standard deviation. Predictions are transformed back to the original
target scale before evaluation.

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
sanitisation and canonicalisation. Duplicate groups with sample target
standard deviation above 0.5 are removed. Retained groups use averaged
targets. No subsampling is performed.

The original download date and dataset hashes were not recorded.
Preserve the submitted curated CSV and record its SHA-256 hash to
identify the exact training-data version.

## Training and Evaluation
All core pipelines use the same saved Murcko-scaffold assignments.

Recorded split sizes per seed:
- Training: 892 molecules.
- Validation: 111 molecules.
- Test: 113 molecules.

The notebook checks that splits contain no scaffold overlap.
The model state with the best validation RMSE is restored before
test evaluation. Test metrics do not select the checkpoint.

## Recorded Test Performance

Mean ± sample standard deviation across five seeds:

| Metric | Value |
|---|---|
| RMSE | 0.9276 ± 0.1008 |
| MAE | 0.7251 ± 0.0541 |
| R² | 0.7970 ± 0.0567 |

Standard deviations use ddof=1. These are recorded notebook outputs,
not results of a newly verified run.

Results:
- outputs/gcn_esol_metrics_per_seed.csv
- outputs/gcn_esol_summary.csv
- outputs/gcn_esol_test_predictions.csv

## Intended Use
Academic evaluation of graph-based molecular representations for
ESOL solubility prediction.

Inference requires the same atom-feature definitions, model
architecture, trained weights and training-target scaling parameters.

## Limitations
- Explicit bond-type features and 3D geometry are absent.
- Generalisation depends on the training chemistry and scaffolds.
- Acyclic molecules share an empty Murcko scaffold and are grouped
  together by the splitting procedure.
- Weak feature–error correlations do not establish causality.
- Performance on other molecular properties is unvalidated.
- Changing architecture and representation together prevents
  representation-only attribution.

## Saved Model Status
The uploaded notebook retains the best state inside fit_gcn() and
restores it for prediction, but does not export a GCN checkpoint
to disk.

Before submitting trained models, save a checkpoint inside fit_gcn()
after restoring best_state. Include:
- Model state dictionary.
- Input feature dimension.
- Hidden dimension, number of layers and dropout.
- Random seed.
- Best epoch.
- Training-target mean, y_mean.
- Training-target standard deviation, y_std.

The scaling parameters are necessary to convert predictions back
to the original target scale. Metrics and prediction CSVs are not
a saved trained GCN.

## Reproducibility
Seeds: 42, 43, 44, 45 and 46.

The notebook records RDKit version 2026.09.1. Original PyTorch,
PyTorch Geometric and other package versions were not recorded.

Preserve the actual environment, data, splits and checkpoint files.
Fixed seeds alone do not guarantee identical results across hardware
or software versions.
