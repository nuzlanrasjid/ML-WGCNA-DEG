# WGCNA-DEG-ML

Feature selection by intersecting WGCNA modules with differentially
expressed genes, followed by machine learning on METABRIC breast cancer
data (Basal vs non-Basal).

This repository contains the code only. Raw data is not included.

## Repository structure

```
ML-WGCNA-DEG/
├── data/
│   └── input/                      # put METABRIC_RNA_Mutation.csv here (not tracked)
├── wgcna-dge-ml-workflow-script
├── outputs/
│   ├── 01_wgcna/                   # gene-module table
│   ├── 02_dge/                     # DEG table
│   ├── 03_overlap/                 # DEG x WGCNA overlap
│   └── 04_ml/
│       ├── tables/                 # model metrics, SHAP tables
│       └── figures/                # SHAP plots
├── requirements.txt
├── R_sessionInfo.txt
├── .gitignore
├── LICENSE
└── README.md
```

`data/input/` holds raw data only and is never modified by the scripts.
Everything the scripts generate goes to `outputs/`, one subfolder per step.
Each step reads the previous step's files from `outputs/`.

## Pipeline

| # | Script | Language | Input | Output |
|---|--------|----------|-------|--------|
| 1 | `scripts/01_wgcna.R` | R | `data/input/METABRIC_RNA_Mutation.csv` | `outputs/01_wgcna/metabric_gene_modules_R.csv` |
| 2 | `scripts/02_dge_basal_vs_rest.py` | Python | `data/input/METABRIC_RNA_Mutation.csv` | `outputs/02_dge/deg_basal_vs_rest.csv` |
| 3 | `scripts/03_overlap.py` | Python | module table + DEG table | `outputs/03_overlap/deg_wgcna_overlap.csv` |
| 4 | `scripts/04_ml_shap_n42.py` | Python | raw CSV + the two tables above + overlap | `outputs/04_ml/` (tables and figures) |

Run the steps in order. Step 1 runs once, globally, on all samples.

## Dataset

METABRIC expression and clinical data ( (Pereira et al., 2016, *Nature*), 
accessed through a Kaggle re-upload. Download `METABRIC_RNA_Mutation.csv` 
yourself from this link [Kaggle](https://www.kaggle.com/datasets/raghadalharbi/breast-cancer-gene-expression-profiles-metabric) and place it in `data/input/`. The PAM50 label used for classification is the column
`pam50_+_claudin-low_subtype`.

## Requirements

**R:** `WGCNA`

**Python:**

```
numpy
pandas
scikit-learn
xgboost
shap
joblib
matplotlib
```

Install with `pip install -r requirements.txt`. R session details are
in `R_sessionInfo.txt`.

## How to run

The scripts were written for Google Colab with files on Google Drive.

1. Copy the repository folder to Drive and put the raw CSV in `data/input/`.
2. Set `BASE_DIR` at the top of each script to the repository root. Input
   and output paths are derived from it:
   `INPUT_DIR = BASE_DIR/data/input`, `OUTPUT_DIR = BASE_DIR/outputs/<step>`.
3. Run step 1 in R, then steps 2 to 4 in Python.

Settings at the top of `04_ml_shap_n42.py`:

| Parameter | Default | Meaning |
|-----------|---------|---------|
| `TARGET_CLASS` | `"Basal"` | Positive class (Basal vs non-Basal) |
| `RANDOM_STATE` | `42` | Seed for the subsample and models |
| `N_MINORITY`, `N_MAJORITY` | `6`, `36` | Samples drawn per class |
| `N_PERM` | `50` | Label permutations (minimum p = 1/51) |
| `TOP_K_SHAP` | `15` | Top genes per model kept for the consensus list |

## What step 4 does

1. Rebuilds the sample table, then draws a 6 vs 36 subsample.
2. Evaluates Random Forest, XGBoost and SVM with leave-one-out CV and
   balanced accuracy.
3. Builds a permutation null (50 shuffles, run in parallel with joblib).
4. Reports sensitivity and specificity from the LOO predictions.
5. Repeats the subsampling with 10 seeds to check stability.
6. Computes SHAP values for RF and XGBoost, then a consensus gene list
   (top 15 in both models) and a direction check.
7. Saves bar and beeswarm plots.

## Outputs

| File | Content |
|------|---------|
| `outputs/01_wgcna/metabric_gene_modules_R.csv` | Gene-to-module assignment |
| `outputs/02_dge/deg_basal_vs_rest.csv` | Differential expression, Basal vs rest |
| `outputs/03_overlap/deg_wgcna_overlap.csv` | Genes in both DEG list and WGCNA modules |
| `outputs/04_ml/tables/ml_n42_results.csv` | Balanced accuracy, null mean and std, permutation p-value per model |
| `outputs/04_ml/tables/shap_importance_RF.csv`, `shap_importance_XGB.csv` | Mean abs SHAP per gene |
| `outputs/04_ml/tables/shap_consensus_genes.csv` | Genes in the top 15 of both models |
| `outputs/04_ml/figures/shap_bar_basal_vs_rest.png` | Bar plot of mean abs SHAP, coloured by direction |
| `outputs/04_ml/figures/shap_beeswarm_rf.png` | SHAP beeswarm for Random Forest |

## Notes

- Hyperparameters are fixed, not tuned, to keep leave-one-out CV
  affordable on 42 samples.
- SVM is excluded from SHAP because it was unstable across seeds.
- Steps 1 to 3 use all samples, including the 42 later evaluated, so
  cross-validated scores may be optimistic.
- The MIT license covers the code only, not the METABRIC data.
