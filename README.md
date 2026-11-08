# ML-WGCNA-DEG
Combination of WGNA and DEG to train cancer dataset on ML
# ML-WGCNA-DEG

Feature selection by intersecting WGCNA modules with differentially
expressed genes, followed by machine learning on METABRIC breast cancer
data (Basal vs non-Basal).

This repository contains the code only. Raw data is not included.

## Pipeline

| # | Script | Language | Input | Output |
|---|--------|----------|-------|--------|
| 1 | `scripts/01_wgcna.R` | R | `METABRIC_RNA_Mutation.csv` | `metabric_gene_modules_R.csv` |
| 2 | `scripts/02_dge_basal_vs_rest.py` | Python | `METABRIC_RNA_Mutation.csv` | DEG table (CSV) |
| 3 | `scripts/03_overlap.py` | Python | module table + DEG table | `deg_wgcna_overlap.csv` |
| 4 | `scripts/04_ml_shap_n42.py` | Python | the three files above | `ml_n42_results.csv`, SHAP tables and plots |

Run the steps in order. Step 1 runs once, globally, on all samples.

## Dataset

METABRIC expression and clinical data (Curtis et al., 2012, *Nature*
486:346-352), accessed through a Kaggle re-upload. Download
`METABRIC_RNA_Mutation.csv` yourself and place it in your working
folder. The PAM50 label used for classification is the column
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

1. Put the input CSV files in one Drive folder.
2. Set `BASE_DIR` at the top of each Python script to that folder.
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
| `ml_n42_results.csv` | Balanced accuracy, null mean and std, permutation p-value per model |
| `shap_importance_RF.csv`, `shap_importance_XGB.csv` | Mean abs SHAP per gene |
| `shap_consensus_genes.csv` | Genes in the top 15 of both models |
| `shap_bar_basal_vs_rest.png` | Bar plot of mean abs SHAP, coloured by direction |
| `shap_beeswarm_rf.png` | SHAP beeswarm for Random Forest |

## Repository structure

```
ML-WGCNA-DEG/
├── scripts/
├── results/
├── requirements.txt
├── R_sessionInfo.txt
├── .gitignore
├── LICENSE
└── README.md
```

## Notes

- Hyperparameters are fixed, not tuned, to keep leave-one-out CV
  affordable on 42 samples.
- SVM is excluded from SHAP because it was unstable across seeds.
- Steps 1 to 3 use all samples, including the 42 later evaluated, so
  cross-validated scores may be optimistic.
- The MIT license covers the code only, not the METABRIC data.
