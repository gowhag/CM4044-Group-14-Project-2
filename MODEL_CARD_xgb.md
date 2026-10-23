# Model Card: RDKit Descriptors + XGBoost

## Model Overview
- Project: CM4044 Project 2 — One Molecule, Many Faces.
- Model role: baseline.
- Task: regression of aqueous solubility using ESOL.
- Input: RDKit molecular descriptors from curated molecules.
- Output: predicted measured log solubility, using the ESOL target scale.
- Implementation: XGBRegressor in [ESOL]_CM4044_Prj2.ipynb.

## Architecture and Hyperparameters

| Parameter | Value |
|---|---|
| Maximum number of estimators | 500 |
| Maximum tree depth | 5 |
| Learning rate | 0.03 |
| Subsample ratio | 0.8 |
| Column subsample ratio | 0.8 |
| Objective | reg:squarederror |
| Early-stopping rounds | 30 |
| Random seeds | 42, 43, 44, 45, 46 |
| CPU threads | n_jobs=-1 |

Non-finite descriptor values and values with absolute magnitude above
1e8 are treated as missing. Descriptor columns entirely missing in
the training set are removed.

Median imputation is fitted on training data only and applied to the
validation and test sets. The recorded run retains 217 descriptors
for each seed.

## Training Data and Version
- Dataset: ESOL.
- Source:
  https://deepchemdata.s3-us-west-1.amazonaws.com/datasets/delaney-processed.csv
- Raw dataset: 1,128 entries.
- Curated dataset: 1,116 molecules.
- Training-data file: outputs/esol_curated.csv.
- Curation record: outputs/esol_curation_log.json.
- Split manifest: outputs/esol_scaffold_splits.csv.

Curation includes SMILES parsing, largest-fragment selection,
neutralisation, sanitisation and canonicalisation. Duplicate canonical
SMILES are grouped. Targets are averaged for groups with sample
standard deviation no greater than 0.5; groups above this threshold
are removed. No subsampling is performed.

The notebook does not record the dataset download date or file hash.
The submitted curated CSV identifies the data used. Its SHA-256 hash
should be recorded to provide an immutable training-data version.

## Training and Evaluation
Randomised Murcko-scaffold splits target an 80/10/10 division.
The recorded sizes per seed are:
- Training: 892 molecules.
- Validation: 111 molecules.
- Test: 113 molecules.

All core pipelines use the same saved split manifest.
Validation performance controls early stopping. Test data are used
for final evaluation.

## Recorded Test Performance

Mean ± sample standard deviation across five seeds:

| Metric | Value |
|---|---|
| RMSE | 0.8114 ± 0.1134 |
| MAE | 0.6174 ± 0.0698 |
| R² | 0.8452 ± 0.0415 |

Standard deviations use ddof=1. These values are recorded notebook
outputs, not results of a newly verified run.

Results:
- outputs/xgb_esol_5_seeds.csv
- outputs/xgb_esol_test_predictions.csv

## Intended Use
Academic comparison of molecular representation–model pipelines for
ESOL solubility prediction.

Inference requires the same molecular curation, descriptor ordering,
descriptor-column selection and fitted imputation procedure.

## Limitations
- Generalisation is limited by the size and chemical coverage of ESOL.
- Descriptor definitions can differ between RDKit versions.
- Performance on other properties or unrelated chemical domains
  has not been established.
- Changing both representation and architecture prevents the
  comparison from isolating representation's contribution.
- Predictions do not replace experimental solubility measurements.

## Saved Model Status
The uploaded notebook saves metrics and predictions but does not
export the fitted XGBoost model.

For the trained-model submission, save each seed's fitted estimator,
descriptor names, retained-column mask and fitted imputer.
A prediction CSV is not a saved trained model.

## Reproducibility
Seeds: 42, 43, 44, 45 and 46.

The notebook records RDKit version 2026.09.1. Other original package
versions were not recorded. Preserve the actual environment,
curated data, split manifest and trained artifacts.

Fixed seeds alone do not guarantee identical results across
different hardware and software versions.
