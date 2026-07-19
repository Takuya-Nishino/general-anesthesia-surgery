# General Anesthesia Surgery: Multicenter Perioperative Machine Learning Analysis

This repository contains the analysis code accompanying the manuscript:

> **Pragmatic Multicenter Development and Validation of Machine Learning Models Using Routinely Available EHR Data to Predict Major Postoperative Complications After General Anesthesia**

The study used routinely collected electronic health record, Diagnosis Procedure Combination, and anesthesia information system data from three tertiary hospitals to develop and validate machine learning models for 30-day major postoperative complications after noncardiac surgery under general anesthesia.

## Repository Scope

The repository provides code for:

- leave-one-institution-out internal-external cross-validation (IECV);
- stratified random hold-out validation;
- temporal validation;
- logistic regression, XGBoost, and LightGBM models;
- single-imputation primary analyses and multiple-imputation sensitivity analyses;
- postoperative infection and non-infection composite sensitivity analyses;
- discrimination, calibration, overall performance, and decision curve analysis;
- out-of-sample SHAP analyses;
- SHAP interaction analyses for XGBoost and LightGBM;
- manuscript and supplementary tables and figures.

The repository does **not** contain clinical data, patient-level predictions, fitted model objects, or other potentially identifiable outputs.

## Study Overview

| Item | Description |
|---|---|
| Design | Retrospective multicenter observational cohort study |
| Setting | Three tertiary hospitals in Japan |
| Study period | January 2021 to March 2025 |
| Population | Adults undergoing noncardiac surgery under general anesthesia |
| Primary outcome | Composite of 30-day all-cause mortality, unplanned reoperation, prolonged mechanical ventilation or early reintubation, new renal replacement therapy, and postoperative infection |
| Primary validation | Leave-one-institution-out IECV |
| Secondary validation | Stratified 80:20 random hold-out validation |
| Temporal validation | Development: January 2021–December 2023; validation: January 2024–March 2025 |
| Algorithms | Logistic regression, XGBoost, and LightGBM |
| Feature sets | Preoperative model: 36 predictors; perioperative model: 43 predictors |
| Probability calibration | Isotonic regression fitted without access to the evaluation data |
| Main performance measures | AUROC, PR-AUC (average precision), Brier score, scaled Brier score, calibration slope, calibration intercept, and decision curve analysis |
| Model interpretation | Out-of-sample IECV SHAP values and SHAP interaction values |

## Files

The commands below assume that the following files are stored in the repository root. They may instead be placed under an `analysis/` directory if the paths are adjusted accordingly.

### Main analysis

`JMIR_Revision_SubmissionReady_Full_Analysis_3Model_SHAP_TemporalValidation_ReviceData20260714.py`

Runs the complete analysis pipeline from the raw, nonimputed analytic data set:

1. feature dictionary generation;
2. primary IECV;
3. random hold-out validation;
4. temporal validation;
5. outcome sensitivity analyses;
6. multiple-imputation sensitivity analysis;
7. performance estimation and bootstrap confidence intervals;
8. publication-table generation;
9. out-of-sample SHAP analysis;
10. JMIR-compliant figure generation;
11. run manifests and output inventories.

### Patient-characteristics tables

`JMIR_Patient_Characteristics_Standalone.py`

Generates descriptive tables from the raw, nonimputed data without fitting prediction models:

- main patient-characteristics table;
- full institution-specific characteristics;
- temporal development-versus-validation characteristics;
- random hold-out balance diagnostics;
- missingness summaries.

### Corrected pooled IECV Table 2

`JMIR_Build_Submission_Table2_Pooled_IECV_CorrectedInference.ipynb`

Rebuilds the pooled IECV submission table from existing patient-level out-of-sample predictions and a calibration-corrected metric table. It does not refit any prediction model.

The notebook applies paired institution-by-outcome-stratified bootstrap inference to differences between preoperative and perioperative models.

### SHAP interaction analysis

`JMIR_SHAP_Interaction_Offline_Reanalysis_20260714_corrected.py`

Loads the final fitted IECV model bundles and calculates pooled out-of-sample SHAP interaction values for:

- preoperative XGBoost;
- preoperative LightGBM;
- perioperative XGBoost;
- perioperative LightGBM.

The script does not repeat model fitting, imputation, hyperparameter optimization, or probability calibration.

## Recommended Repository Layout

```text
general-anesthesia-surgery/
├── README.md
├── JMIR_Revision_SubmissionReady_Full_Analysis_3Model_SHAP_TemporalValidation_ReviceData20260714.py
├── JMIR_Patient_Characteristics_Standalone.py
├── JMIR_Build_Submission_Table2_Pooled_IECV_CorrectedInference.ipynb
├── JMIR_SHAP_Interaction_Offline_Reanalysis_20260714_corrected.py
├── requirements.txt
├── .gitignore
└── data/
    └── README.md
```

The `data/README.md` file should explain the access restrictions and expected local data structure. The clinical data set itself must not be committed.

## Data Availability

The clinical data cannot be made publicly available because they are subject to Japanese privacy regulations, institutional policy, and ethics approval conditions.

Deidentified data may be considered for legitimate academic purposes upon reasonable request to the corresponding author, subject to approval by the institutional review board and participating institutions.

The public repository therefore supports **code transparency**, not unrestricted data replication.

## Required Input Data

The principal scripts expect an Excel workbook containing one row per surgical admission.

The default local filename used in the scripts is:

```text
Revice_data_20260714.xlsx
```

This file is not included in the repository.

### Required identifier, grouping, outcome, and period columns

| Column | Role | Required values |
|---|---|---|
| `INDEX` | Unique analytic episode identifier | Unique, nonmissing |
| `付属_1` | Institution code | Exactly 3 distinct institutions |
| `Event` | Primary composite outcome | 0 or 1; nonmissing |
| `PI` | Postoperative infection outcome | 0 or 1; nonmissing |
| `Hard_endpoint` | Non-infection composite outcome | 0 or 1; nonmissing |
| `Period24_25` | Temporal validation indicator | 0=2021–2023; 1=2024–March 2025 |

The analysis verifies the following logical relationship:

```text
Event = 1 when PI = 1 or Hard_endpoint = 1
```

### Preoperative predictors

```text
ASA
Age
Male
BMI
Dialysis
CHF
Malig
Alb
BUN
CRP
Cre
Hb
K
Na
PLT
T-Bil
WBC
DeliMed
β-blocker
Oral steroids
Antiplatelet
Anticoag
AntiCa
Opioid
Proc-Eye
Proc-Face/Neck
Proc-Thorax
Proc-MSK
Proc-ENT
Proc-Neuro
Proc-Genital
Proc-Urinary
Proc-Skin
Proc-Abd
ResectNum
HighRiskProc
```

### Additional perioperative predictors

```text
OpTime
RBC Tx
FFP Tx
PLT Tx
FluidBal
HR at 6h
MAP at 6h
```

The perioperative model includes all preoperative predictors plus these seven additional variables.

## Software Environment

The analysis was developed in Python 3.11 on Windows. The principal tested package versions were:

```text
Python              3.11.11
numpy               1.26.4
pandas              2.2.3
scikit-learn        1.6.1
scipy               1.15.2
xgboost             3.0.1
lightgbm            4.6.0
optuna               4.3.0
matplotlib           3.10.0
shap                 0.47.2
```

Additional dependencies include:

```text
cloudpickle
joblib
openpyxl
Pillow
IPython
```

The main analysis writes an `analysis_manifest.json` file containing the Python version, operating system information, analysis settings, random seeds, and installed package versions for each run.

## Installation

Create and activate a Python 3.11 environment, then install the required packages.

```bash
python -m pip install \
  numpy==1.26.4 \
  pandas==2.2.3 \
  scikit-learn==1.6.1 \
  scipy==1.15.2 \
  xgboost==3.0.1 \
  lightgbm==4.6.0 \
  optuna==4.3.0 \
  matplotlib==3.10.0 \
  shap==0.47.2 \
  cloudpickle \
  joblib \
  openpyxl \
  Pillow \
  IPython \
  jupyter
```

CPU execution is supported. GPU acceleration is not required.

## Configuration

Before execution, edit the user-settings section near the beginning of each file.

### Main analysis

Set:

```python
RAW_DATA_PATH = Path(r"path\to\Revice_data_20260714.xlsx")
OUTPUT_DIR = Path(r"path\to\analysis_outputs")
```

The final analysis settings use:

```python
REUSE_EXISTING_OUTPUTS = False
FAST_TEST_MODE = False
```

For a clean final run, retain these settings. For resuming an interrupted run with unchanged data and settings:

```python
REUSE_EXISTING_OUTPUTS = True
```

This enables reuse of compatible preprocessing caches, tuned parameters, models, and predictions.

### Patient-characteristics script

Set:

```python
RAW_DATA_PATH = Path(r"path\to\Revice_data_20260714.xlsx")
MAIN_ANALYSIS_OUTPUT_DIR = Path(r"path\to\analysis_outputs")
```

The script reads the same raw data and reuses the fixed hold-out split created by the main analysis when available.

### Corrected Table 2 notebook

Set:

```python
ROOT_DIR = Path(r"path\to\analysis_outputs")
```

The notebook requires:

```text
04_predictions/Primary_all_predictions.csv
08_publication_tables/Table2_Primary_IECV_Validation_CalibrationFixed.xlsx
```

It writes the corrected submission table to `08_publication_tables/`.

### SHAP interaction script

Set:

```python
PROJECT_DIR = Path(r"path\to\local_project")
```

The project directory must contain:

```text
Revice_data_20260714.xlsx
03_models/
```

If the fitted model directory is stored elsewhere, specify:

```python
MODEL_DIR_OVERRIDE = Path(r"path\to\03_models")
```

## Execution

The main analysis file uses VSCode/Jupyter-compatible `# %%` cells. The preferred procedure is to open the file in VSCode or Jupyter and execute all cells from top to bottom.

### Recommended order

1. Run the main integrated analysis.
2. Run the patient-characteristics script.
3. Run the corrected pooled IECV Table 2 notebook.
4. Run the corrected SHAP interaction script.

### Command-line execution

After path configuration, the Python scripts may also be executed from the command line:

```bash
python JMIR_Revision_SubmissionReady_Full_Analysis_3Model_SHAP_TemporalValidation_ReviceData20260714.py
python JMIR_Patient_Characteristics_Standalone.py
python JMIR_SHAP_Interaction_Offline_Reanalysis_20260714_corrected.py
```

Run the Table 2 notebook interactively or execute it with Jupyter:

```bash
jupyter nbconvert \
  --to notebook \
  --execute JMIR_Build_Submission_Table2_Pooled_IECV_CorrectedInference.ipynb \
  --output JMIR_Build_Submission_Table2_Pooled_IECV_CorrectedInference_executed.ipynb
```

## Analysis Details

### Validation framework

The primary analysis uses leave-one-institution-out IECV. In each fold, two institutions form the development data and the remaining institution forms the held-out evaluation data.

Random hold-out and temporal validation are secondary assessments and should not replace the IECV results when interpreting transportability.

### Hyperparameter optimization

Optuna with a Tree-structured Parzen Estimator sampler is used for hyperparameter optimization.

Default final-analysis budgets are:

| Model | Trials per primary validation setting |
|---|---:|
| Logistic regression | 20 |
| XGBoost | 50 |
| LightGBM | 50 |

Hyperparameter optimization is performed within the development data only.

### Missing data

The primary analysis uses iterative multivariable single imputation with an `LGBMRegressor` conditional estimator.

The multiple-imputation sensitivity analysis uses 10 imputed data sets. Model fitting and probability calibration are performed separately within each imputed data set, and patient-level calibrated probabilities are pooled before evaluation.

### Calibration

Predicted probabilities are calibrated using isotonic regression without using the evaluation data.

For IECV, calibration is learned from the development institutions and applied unchanged to the held-out institution. Reported held-out calibration therefore reflects transportability without post hoc local recalibration.

### Statistical uncertainty

The integrated analysis uses bootstrap resampling for confidence intervals. The corrected pooled IECV Table 2 notebook uses 10,000 paired institution-by-outcome-stratified bootstrap resamples for preoperative-versus-perioperative model comparisons.

### SHAP interpretation

SHAP values are calculated using held-out observations from each IECV fold.

The explained output is the uncalibrated base-model output before isotonic calibration:

- logistic regression: log-odds scale;
- XGBoost and LightGBM: raw-margin scale.

Absolute SHAP magnitudes should not be compared directly across different algorithms. Within-algorithm rankings and directional patterns are the intended interpretation.

### SHAP interactions

The interaction script:

- samples up to 2000 held-out patients per institution;
- computes absolute SHAP interaction values in batches;
- pools interaction values across IECV held-out institutions;
- sets diagonal elements to zero;
- masks values below the within-model 80th percentile;
- reports predictive interactions, not causal effects.

The batch size can be reduced if memory is limited.

## Main Outputs

The integrated analysis creates the following directory structure:

```text
analysis_outputs/
├── 00_feature_dictionary/
├── 01_cache/
├── 02_hyperparameters/
├── 03_models/
├── 04_predictions/
├── 05_metrics/
├── 06_multiple_imputation/
├── 07_shap/
├── 08_publication_tables/
├── 09_publication_figures/
├── 10_logs/
├── 11_patient_characteristics/
├── fixed_holdout_split.csv
├── Core_Output_File_Manifest.xlsx
└── Output_File_Manifest.xlsx
```

Key outputs include:

- feature definitions, units, measurement windows, missingness, and preprocessing;
- patient-level out-of-sample predictions;
- institution-specific and pooled IECV performance;
- random hold-out and temporal validation performance;
- infection and non-infection outcome sensitivity analyses;
- multiple-imputation results;
- hyperparameter search spaces and selected values;
- SHAP values and feature-importance tables;
- publication-ready Excel tables;
- JMIR-compliant PNG figures;
- run manifests and checksums.

## JMIR Figure Constraints

The final figure-generation code applies the following submission constraints:

- PNG format;
- nontransparent white background;
- maximum dimensions of 1200 × 1200 pixels;
- maximum file size of 5 MB;
- no figure number or full caption embedded in the image;
- noncolor visual encodings in addition to color;
- captions exported separately for submission-system metadata.

## Reproducibility and Integrity Checks

The scripts include checks for:

- required columns;
- uniqueness of `INDEX`;
- binary and nonmissing outcome values;
- exactly three institutions;
- valid temporal-period coding;
- logical consistency among `Event`, `PI`, and `Hard_endpoint`;
- compatibility of cached preprocessing objects and fitted models;
- consistency between patient-level predictions and formatted performance tables;
- output-file dimensions, transparency, and file size;
- run settings, package versions, file signatures, and checksums.

Do not bypass these checks when generating results for publication.

## Files That Must Not Be Committed

At minimum, exclude the following from the public repository:

```gitignore
# Clinical data
*.xlsx
*.xls
data/*
!data/README.md

# Patient-level outputs
04_predictions/
06_multiple_imputation/
07_shap/
fixed_holdout_split.csv

# Fitted models and preprocessing objects
01_cache/
03_models/
*.pkl
*.pickle
*.joblib
*.cloudpickle

# Local output directories
analysis_outputs/
SHAP_interaction_outputs/
_figure_working_files/

# Local configuration and logs
.env
*.log
__pycache__/
.ipynb_checkpoints/
```

Publication tables and figures should be added only after confirming that they contain no patient identifiers or institution-specific confidential information.

## Intended Use

This code is provided for research transparency and methodological reproducibility.

The models were developed retrospectively and have not been prospectively validated as clinical decision-support systems. The code and model outputs must not be used for direct clinical decision-making without independent validation, local recalibration, governance review, and prospective safety evaluation.

## Citation

Until the final journal citation is available, cite the accompanying manuscript as:

```text
Nishino T, Mase H, Kim C, Sugita S, Kondo Y, Ishikawa M.
Pragmatic Multicenter Development and Validation of Machine Learning Models
Using Routinely Available EHR Data to Predict Major Postoperative Complications
After General Anesthesia. Manuscript submitted to JMIR Perioperative Medicine.
```

BibTeX placeholder:

```bibtex
@unpublished{nishino_perioperative_ml_2026,
  author = {Nishino, Takuya and Mase, Hiroshi and Kim, Chol and Sugita, Shinji and Kondo, Yukihiro and Ishikawa, Masashi},
  title = {Pragmatic Multicenter Development and Validation of Machine Learning Models Using Routinely Available EHR Data to Predict Major Postoperative Complications After General Anesthesia},
  year = {2026},
  note = {Manuscript submitted to JMIR Perioperative Medicine}
}
```

## Contact

For questions regarding the analytic code or data-access procedures, contact the corresponding author listed in the manuscript.
