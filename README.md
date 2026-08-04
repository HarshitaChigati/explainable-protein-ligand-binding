\# Explainable Structure-Based Protein-Ligand Binding Affinity Prediction



An explainable machine learning framework for protein–ligand binding affinity prediction that uses an ablation-style comparison of ligand-only, protein-surface-only, and combined feature sets to quantify how much predictive signal each contributes individually and jointly under the real-world constraint that atom-level protein representations were unavailable.

!\[Python](https://img.shields.io/badge/Python-3.12-blue)

!\[scikit-learn](https://img.shields.io/badge/scikit--learn-GradientBoosting-orange)

!\[Bioinformatics](https://img.shields.io/badge/Bioinformatics-Protein--Ligand-green)

!\[Interpretability](https://img.shields.io/badge/Explainable-SHAP-purple)

!\[License](https://img.shields.io/badge/License-MIT-lightgrey)



\---



\## Project Overview



This project asks a decomposition question rather than a pure accuracy question: of the

predictive signal available for protein-ligand binding affinity, how much comes from

ligand chemistry alone, how much from protein-pocket surface structure alone, and how

much only emerges when the two are combined? Three models trained on identical data with

an identical algorithm — differing only in which feature group they receive — make this

an ablation study by construction, not just a model comparison. Results are then

interpreted with SHAP and a shell-level structural analysis to identify which specific

features drive the combined model's predictions.



\*\*Why this project exists:\*\* Most structure-based scoring functions, including RF-Score, assume atom-resolved protein structures from which atom-pair interaction features can be computed. This project instead investigates how much predictive information can be recovered from surface-based protein representations when atom-level protein coordinates are unavailable. Here, the protein pocket is only available as a surface mesh with per-vertex biophysical descriptors — a common situation when working with pre-processed or third-party structural data. This project demonstrates that meaningful binding affinity prediction remains possible using engineered surface-derived structural descriptors, even when atom-level protein information is unavailable.

\---



\## Pipeline



```text

&#x20;                   CASF-2016 Core Set (285 Crystal Complexes)

&#x20;                                   │

&#x20;                                   ▼

&#x20;                 Merge Experimental pKa Labels → 284 Matched Complexes

&#x20;                                   │

&#x20;                                   ▼

&#x20;                     Independent Feature Engineering

&#x20;              ┌────────────────────┼────────────────────┐

&#x20;              ▼                    ▼                     ▼

&#x20;      Ligand Features      Structure Features      Combined Features

&#x20;       (41 features)         (17 features)           (58 features)

&#x20;     atom/bond composition,  ligand-pocket distance   Model A + Model B

&#x20;     atom \& bond counts,     shells (0-4 / 4-6 / 6-8 Å),

&#x20;     radius of gyration,     pocket surface descriptors

&#x20;     bounding-box shape      per shell

&#x20;              └────────────────────┼────────────────────┘

&#x20;                                   ▼

&#x20;                GradientBoostingRegressor (scikit-learn)

&#x20;                                   │

&#x20;                                   ▼

&#x20;                        5-Fold Cross-Validation

&#x20;                                   │

&#x20;                ┌──────────────────┼──────────────────┐

&#x20;                ▼                  ▼                  ▼

&#x20;         Pearson Correlation      RMSE       Out-of-Fold Predictions

&#x20;                                   │

&#x20;                                   ▼

&#x20;                         Model Interpretation

&#x20;                ┌──────────────────┴──────────────────┐

&#x20;                ▼                                     ▼

&#x20;        SHAP Feature Analysis            Shell-Level Correlation Analysis

&#x20;                                                       │

&#x20;                                                       ▼

&#x20;                                         0-4 Å shell shows the strongest

&#x20;                                         correlation with binding affinity

```



\---



\## Data



\- \*\*PDBbind v2016 core set / CASF-2016 benchmark\*\* — 285 high-resolution protein-ligand

&#x20; crystal complexes, the standard scoring-function benchmark used across the field.

\- Structures are provided as pre-processed ligand and protein-pocket graphs rather than

&#x20; raw PDB files. The protein pocket is represented as a molecular surface mesh (\~750

&#x20; vertices per complex, MaSIF-style) with 4 biophysical descriptors per point, not as

&#x20; discrete atoms — see Limitations.

\- Binding affinity (pKa) labels were sourced from the public CASF-2016 core-set benchmark

&#x20; values (via `zhenglz/onionnet`, an open-source repository redistributing the same

&#x20; PDBbind-derived labels used across the scoring-function literature) and merged by PDB

&#x20; ID. 284 of 285 complexes matched (`1g2k` had no label available and was dropped).



\---



\## Setup (Google Colab)



This notebook is written to run in Google Colab with the data files stored on Google

Drive.



1\. Upload `dataset\_CASF-2016\_285` and `casf2016\_pka\_labels.csv` to a folder in your

&#x20;  Google Drive.

2\. Open the notebook in Colab.

3\. In the first code cell, update the paths to point at your Drive folder:



&#x20;  ```python

&#x20;  from google.colab import drive

&#x20;  drive.mount('/content/drive')



&#x20;  DATA\_PATH = "/content/drive/MyDrive/<your-folder-name>/dataset\_CASF-2016\_285"

&#x20;  LABELS\_PATH = "/content/drive/MyDrive/<your-folder-name>/casf2016\_pka\_labels.csv"

&#x20;  ```



4\. Run all cells top to bottom (`Runtime → Run all`).



> \*\*Note:\*\* the structure file has no file extension (`dataset\_CASF-2016\_285`, not

> `.tar`) — this is intentional; `torch.load` doesn't require a specific extension.



\*\*Running locally instead of Colab?\*\* Skip the `drive.mount(...)` cell and set the two

paths to wherever the files live on your machine, e.g. `data/dataset\_CASF-2016\_285`.



\---



\## Method



An ablation design: three models share the same Gradient Boosted Trees algorithm and

the same 284 complexes, evaluated with 5-fold cross-validation so every prediction is

out-of-fold. Only the feature group changes; the learning algorithm, data, and

evaluation protocol remain identical. Consequently, any difference in predictive performance can be attributed to the information contained in the feature sets rather than differences in the learning algorithm.



| Model | Feature Type | Number of Features |

|---|---|---:|

| A | Ligand composition + molecular geometry | 41 |

| B | Protein-pocket structural descriptors | 17 |

| C | Combined feature set (A + B) | 58 |



\---



\## Results



| Model | Pearson R | RMSE |

|---|---:|---:|

| A: Ligand-only | 0.571 | 1.801 |

| B: Structure-only | 0.591 | 1.760 |

| \*\*C: Combined\*\* | \*\*0.615\*\* | \*\*1.725\*\* |

```text

&#x20;                   Pearson Correlation with Experimental pKa



Ligand-only        ███████████████████████████            0.571

Structure-only      ████████████████████████████           0.591

Combined            █████████████████████████████          0.615

```



<p align="center">

<img src="images/model\_comparison.png" width="800">

</p>



The combined model consistently outperformed both individual models, demonstrating

that ligand-derived and protein-surface-derived features capture complementary

aspects of protein–ligand binding affinity.



\*(These numbers reflect 5-fold cross-validation on a 284-complex set, not the

literature-standard train-on-refined-set / test-on-core-set protocol, so they are not

directly comparable to published RF-Score / OnionNet / Pafnucy benchmark figures — see

Limitations.)\*



\### Exploratory Data Analysis



<p align="center">

<img src="images/eda\_overview.png" width="800">

</p>



\---



\## Interpretability



SHAP analysis on the combined model shows which individual features — ligand

composition and structural shell features alike — drive predictions, and in which

direction:



<p align="center">

<img src="images/shap\_summary.png" width="800">

</p>



Shell-level correlation analysis: the innermost contact shell (0-4Å) shows a strong

positive correlation with affinity (R=0.55) — complexes where the ligand is more tightly

enclosed by pocket surface at close range tend to bind more strongly, consistent with

better steric/shape complementarity.



<p align="center">

<img src="images/shell\_correlation.png" width="700">

</p>



\---



\## Scientific Contributions



Unlike binding-affinity prediction work that optimizes primarily for predictive accuracy,

this project runs a controlled ablation to explicitly separate ligand-derived and

protein-derived information and quantify each feature group's independent predictive

contribution — then goes a step further with SHAP and shell-level structural analysis to

identify which specific features drive predicted affinity, not just how accurate

the prediction is. This is also demonstrated under a realistic data constraint

(surface-derived, not atom-resolved, protein features), rather than assuming ideal input

data.



\---



\## Key Takeaways



\- Ligand chemistry alone contains substantial predictive information.

\- Protein-surface descriptors independently capture meaningful structural information.

\- Combining both feature groups yields the strongest predictive performance.

\- SHAP identifies the engineered features driving predictions.

\- Close-range (0–4 Å) protein–ligand interactions show the strongest association with binding affinity.



\---



\## Repository Structure



```

├── binding\_affinity\_prediction.ipynb   # full analysis notebook

├── dataset\_CASF-2016\_285                # ligand + pocket-surface graphs (285 complexes)

├── casf2016\_pka\_labels.csv              # PDB ID → pKa labels (CASF-2016 core set)

├── images/                              # figures referenced in this README

│   ├── eda\_overview.png

│   ├── model\_comparison.png

│   ├── shap\_summary.png

│   └── shell\_correlation.png

└── README.md

```



\---



\## Limitations \& Future Work



\- No independent train/test split — only the 284-complex core set was available, so all

&#x20; evaluation is via cross-validation within it.

\- No atom-level protein features — the pocket is a surface mesh, not discrete atoms, so a

&#x20; true RF-Score-style element-pair (C-C, N-O, ...) fingerprint wasn't possible here.

\- \*\*Future work:\*\* re-run this pipeline on raw PDB structures to recover atom-level

&#x20; element-pair contact fingerprints; train on the full PDBbind refined/general set for a

&#x20; literature-comparable evaluation; explore per-target-class (e.g. kinases vs. proteases)

&#x20; breakdowns given a larger, better-balanced dataset.



\---



\## Citation



Ballester, P. J., \& Mitchell, J. B. O. (2010).

A machine learning approach to predicting protein–ligand binding affinity with applications to molecular docking.

Bioinformatics, 26(9), 1169–1175.

https://doi.org/10.1093/bioinformatics/btq112

