# Matrix-Based and Spectral Molecular Descriptors for Heat of Vaporization

Code and data for the paper:

> **Saba Aroosh and Abdul Rauf Nizami.** *Matrix-Based and Spectral Molecular Descriptors for Machine Learning Prediction of the Heat of Vaporization of Aliphatic Hydrocarbons.* Department of Mathematics, University of Central Punjab (UCP), Lahore, Pakistan. Manuscript under review (2026).

This repository contains the dataset of 273 aliphatic hydrocarbons, one Jupyter notebook for each of the five proposed descriptors, and the full 5-fold cross-validation results reported in the paper.

## Repository contents

| Path | Contents |
| --- | --- |
| `data/Aliphatic_HV_with_SMILES.xlsx` | Dataset: 273 aliphatic hydrocarbons (C1–C10) with name, SMILES and heat of vaporization (kJ/mol) |
| `data/Aliphatic_HV_with_SMILES.csv` | The same dataset as CSV |
| `notebooks/Descriptor_1_Z_D_A_C_5foldCV.ipynb` | Descriptor 1: Z + D + A + C (2112 features) |
| `notebooks/Descriptor_2_Z_D_A2_C2_5foldCV.ipynb` | Descriptor 2: Z + D + A² + C² (2112 features) |
| `notebooks/Descriptor_3_Z_D_UT_A_UT_C_5foldCV.ipynb` | Descriptor 3: Z + D + triu(A) + triu(C) (1056 features) |
| `notebooks/Descriptor_4_Z_D_EigA_EigC_5foldCV.ipynb` | Descriptor 4: Z + D + λ(A) + λ(C) (128 features) |
| `notebooks/Descriptor_5_Z_D_WA_WC_5foldCV.ipynb` | Descriptor 5: Z + D + Aw + Cw (2112 features) |
| `results/HV_5fold_CV_results_all.xlsx` | All results: proposed descriptors, ECFP4/MACCS, fusion descriptors, size baselines, hybrid models, block ablation and extrapolation split |

## Descriptors

Each SMILES string is parsed with a small custom parser (no cheminformatics library needed), and hydrogens are added explicitly. Atoms are indexed as the carbons in SMILES order, followed by all hydrogens. All vectors and matrices are zero-padded to N_max = 32 atoms, which is the size of the largest molecules in the dataset (C10H22).

- **Z**: atomic number vector.
- **D**: vertex degree vector, D_i = Σ_j A_ij.
- **A**: adjacency matrix (1 if bonded, 0 otherwise).
- **C**: Coulomb-type matrix, C_ii = 0.5 Z_i^2.4 and C_ij = Z_i Z_j / |i − j|. The index difference replaces the inter-atomic distance. Because hydrogens are indexed after carbons, C depends only on the molecular formula.
- **Aw, Cw**: bond-order-weighted versions (single = 1, double = 2, triple = 3). Cw equals C except that bonded pairs are multiplied by the bond order.
- **λ(·)**: eigenvalues of the unpadded matrix, sorted in descending order and zero-padded to length 32.

## Evaluation protocol

- 5-fold cross-validation with `KFold(n_splits=5, shuffle=True, random_state=42)`. The same folds are used for every descriptor and model.
- Models: Random Forest (300 trees), XGBoost (400 rounds) and LightGBM (400 trees), all with `random_state=42` and otherwise default hyperparameters (no tuning).
- Metrics: R² and MSE, in (kJ/mol)², reported as mean ± SD over the five folds.

## Main results (5-fold CV, best model per descriptor)

| Descriptor | Features | Best model | R² (mean ± SD) | MSE (mean ± SD) |
| --- | ---: | --- | --- | --- |
| Desc-1 | 2112 | RF | 0.918 ± 0.041 | 5.06 ± 2.83 |
| Desc-2 | 2112 | RF | 0.925 ± 0.039 | 4.68 ± 2.75 |
| Desc-3 | 1056 | RF | 0.917 ± 0.040 | 5.10 ± 2.81 |
| Desc-4 | 128 | RF | 0.926 ± 0.029 | 4.52 ± 2.08 |
| Desc-5 | 2112 | RF | 0.917 ± 0.040 | 5.09 ± 2.78 |
| Size baseline (nC + nH + MW, linear) | 3 | Linear | 0.935 ± 0.023 | 4.01 ± 1.82 |
| Hybrid (size baseline + RF on Desc-4 residuals) | 131 | Linear + RF | 0.950 ± 0.027 | 3.15 ± 1.94 |

The complete results are in `results/HV_5fold_CV_results_all.xlsx`. They include all three models for every descriptor, the ECFP4 and MACCS benchmarks, the fusion descriptors, the Descriptor 4 block ablation and the extrapolation split.

## How to run

**Google Colab:** open a notebook, run all cells, and upload `data/Aliphatic_HV_with_SMILES.xlsx` when asked. The first cell installs the required packages.

**Locally:**

```bash
git clone <this repository URL>
cd HV-matrix-descriptors
pip install -r requirements.txt
cd notebooks
jupyter notebook
```

Run any notebook from top to bottom. It reads the dataset from `../data/`, prints R² and MSE per model and fold, and saves `CV_results_DescN.xlsx`. The published results were produced with Python 3, scikit-learn 1.9.1, XGBoost 3.4.1 and LightGBM 4.7.0.

## Notes on the data

The heat of vaporization values were compiled from standard thermochemical references (Majer and Svoboda, 1985; NIST Chemistry WebBook). They do not all refer to the same temperature. For compounds that are gaseous at 298 K (C1–C4), the values refer to lower temperatures. See the paper for details.

## Citation

If you use this code or data, please cite the paper above. The full reference will be added here when the paper is published.

## License

Released under the MIT License (see `LICENSE`).

## Contact

Abdul Rauf Nizami (corresponding author): a.nizami@ucp.edu.pk
