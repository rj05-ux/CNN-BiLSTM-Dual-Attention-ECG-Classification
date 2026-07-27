# 🫀 ECG Arrhythmia Classification

## Multi-Scale CNN-BiLSTM with Dual Attention — Inter-Patient Evaluation on MIT-BIH & INCART

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://python.org)
[![TensorFlow](https://img.shields.io/badge/TensorFlow-2.x-orange)](https://tensorflow.org)
[![Dataset](https://img.shields.io/badge/Dataset-MIT--BIH%20%7C%20INCART-red)](https://physionet.org/content/mitdb/1.0.0/)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

---

## 📌 Overview

This repository contains the full implementation behind our published paper:

> **"An Efficient Multi-Scale CNN-BiLSTM with Dual Attention Mechanism for Inter-Patient ECG Arrhythmia Classification: Evaluation on MIT-BIH and INCART Databases"**
> Published in JETIR, Vol. 13, Issue 7 (July 2026) — [Read the paper](http://www.jetir.org/view?paper=JETIR2607122)

The pipeline classifies ECG heartbeats into the **5 AAMI classes (N, S, V, F, Q)** under the strict **inter-patient protocol** (DS1/DS2 split, zero patient overlap between train and test — the evaluation standard that most published work gets wrong). It combines:

- **EMD-based feature extraction** from raw beats
- **SMOTE** to handle severe class imbalance (S and F classes are <3% of the dataset)
- **Classical ML baselines** (Random Forest, SVM) and **deep models** (1D-CNN, LSTM, multi-scale CNN-BiLSTM with dual attention)
- **Ensembling** (soft-voting, Optuna-weighted voting, stacking) + **Optuna hyperparameter tuning**
- **SHAP explainability** and gradient-based temporal saliency
- **External validation on INCART** (St. Petersburg 12-lead database) to test cross-dataset generalization

---

## 📊 Results

### MIT-BIH, inter-patient (DS1 train / DS2 test), 5-class AAMI

| Model | Accuracy | Macro-F1 |
|---|---|---|
| Random Forest | 92.93% | 0.520 |
| SVM | — | 0.710 (best S-class sensitivity) |
| LSTM | 64.83% | — |
| 1D-CNN | **88.25%** | 0.530 |
| CNN-BiLSTM (Optuna-tuned) | — | 0.672 (up from 0.630 pre-tuning) |
| **Stacking Ensemble (best)** | **94.45%** | **76.66%** |

### External validation — INCART (out-of-distribution)

| Metric | Value |
|---|---|
| Macro-F1 on INCART | **35.79%** |

The drop from 76.66% (MIT-BIH) to 35.79% (INCART) is expected and reported honestly — it reflects genuine domain shift (different lead configuration, population, and annotation protocol), not a pipeline error.

---

## 📁 Repository Structure

```
ecg-arrhythmia-classification/
│
├── notebooks/
│   ├── PartA_Setup.ipynb                          ← Environment, dataset download, AAMI mapping
│   ├── PartB_Preprocessing.ipynb                  ← Denoising, R-peak detection, beat segmentation
│   ├── PartC_EMD_Features.ipynb                   ← EMD decomposition, feature extraction
│   ├── PartD_Split_MRMR.ipynb                     ← Inter-patient DS1/DS2 split, MRMR selection, SMOTE
│   ├── PartE_DeepModels.ipynb                     ← 1D-CNN, LSTM, multi-scale CNN-BiLSTM w/ dual attention
│   ├── PartF_Ensemble_Optuna_SHAP.ipynb           ← Ensembling, Optuna tuning, SHAP + saliency, PTB-XL screening check
│   └── PartG_INCART_External_Validation.ipynb     ← Full external validation on INCART
│
├── results/            ← Saved metrics, results tables
├── figures/             ← Confusion matrices, SHAP plots, training curves
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

---

## 🚀 Quick Start

### 1. Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/ecg-arrhythmia-classification.git
cd ecg-arrhythmia-classification
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Run on Google Colab (recommended — GPU needed for Part E onward)
Open notebooks in order with **Runtime → Change runtime type → T4 GPU** enabled.

### 4. Run notebooks in order
```
PartA → PartB → PartC → PartD → PartE → PartF → PartG
```
Each notebook picks up saved artifacts from the previous one (features, splits, trained models) via a shared `PROJECT_DIR`.

---

## 📦 Datasets

**MIT-BIH Arrhythmia Database** (primary, training + inter-patient test)
- 48 records, 47 subjects, 360 Hz, ~30 min/record
- Downloaded automatically in Part A via `wfdb`

**INCART Database** (external validation only, Part G)
- St. Petersburg 12-lead Arrhythmia Database
- Used to test generalization to an independently-annotated, differently-collected dataset

### AAMI EC57 Class Mapping

| Class | MIT-BIH Symbols | Clinical Meaning |
|---|---|---|
| N | N, L, R, e, j | Normal + bundle branch |
| S | A, a, J, S | Supraventricular ectopic |
| V | V, E | Ventricular ectopic |
| F | F | Fusion beats |
| Q | /, f, Q | Unknown / paced |

---

## ⚙️ Requirements

```
wfdb>=4.1.0
EMD-signal>=1.9.0
imbalanced-learn>=0.11.0
mrmr-selection>=0.2.8
antropy>=0.1.6
scikit-learn>=1.3.0
tensorflow>=2.12.0
optuna>=3.0.0
shap>=0.44.0
numpy>=1.24.0
pandas>=2.0.0
matplotlib>=3.7.0
seaborn>=0.12.0
scipy>=1.11.0
pywavelets>=1.4.1
```

---

## 📖 Citation

If you use this code in your research, please cite:

```bibtex
@article{rutuja2026ecg,
  title   = {An Efficient Multi-Scale CNN-BiLSTM with Dual Attention Mechanism for
             Inter-Patient ECG Arrhythmia Classification: Evaluation on MIT-BIH
             and INCART Databases},
  journal = {JETIR},
  volume  = {13},
  number  = {7},
  year    = {2026},
  url     = {http://www.jetir.org/view?paper=JETIR2607122}
}
```

---

## 📄 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgements

- MIT-BIH Arrhythmia Database: Moody GB, Mark RG (PhysioNet)
- INCART Database: St. Petersburg Institute of Cardiological Technics (PhysioNet)
- AAMI EC57 standard for arrhythmia classification
