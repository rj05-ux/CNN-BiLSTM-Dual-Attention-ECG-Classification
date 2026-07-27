# CNN-BiLSTM-Dual-Attention-ECG-Classification

- Part A — Setup [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/rj05-ux/CNN-BiLSTM-Dual-Attention-ECG-Classification/blob/main/notebooks/PartA_Setup.ipynb)
- Part B — Preprocessing [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/rj05-ux/CNN-BiLSTM-Dual-Attention-ECG-Classification/blob/main/notebooks/PartB_Preprocessing.ipynb)
- Part C — EMD Features [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/rj05-ux/CNN-BiLSTM-Dual-Attention-ECG-Classification/blob/main/notebooks/PartC_EMD_Features.ipynb)
- Part D — Split & MRMR [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/rj05-ux/CNN-BiLSTM-Dual-Attention-ECG-Classification/blob/main/notebooks/PartD_Split_MRMR.ipynb)
- Part E — Deep Models [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/rj05-ux/CNN-BiLSTM-Dual-Attention-ECG-Classification/blob/main/notebooks/PartE_DeepModels.ipynb)
- Part F — Ensemble/Optuna/SHAP [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/rj05-ux/CNN-BiLSTM-Dual-Attention-ECG-Classification/blob/main/notebooks/PartF_Ensemble_Optuna_SHAP_DL.ipynb)
- Part G — INCART External Validation [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/rj05-ux/CNN-BiLSTM-Dual-Attention-ECG-Classification/blob/main/notebooks/PartG_INCART_External_Validation.ipynb)

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://python.org)
[![TensorFlow](https://img.shields.io/badge/TensorFlow-2.x-orange)](https://tensorflow.org)
[![Dataset](https://img.shields.io/badge/Dataset-MIT--BIH%20%7C%20INCART-red)](https://physionet.org/content/mitdb/1.0.0/)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)


This is the code behind our paper, **"An Efficient Multi-Scale CNN-BiLSTM with Dual Attention Mechanism for Inter-Patient ECG Arrhythmia Classification: Evaluation on MIT-BIH and INCART Databases,"** published in JETIR (Vol. 13, Issue 7, July 2026). [Read it here](http://www.jetir.org/view?paper=JETIR2607122).
[Publication certificate](docs/JETIR2607122_Certificate_page-0001.jpg)

## What this actually does

Most papers on ECG beat classification split their data randomly, which lets the same patient's heartbeats leak into both the train and test sets; this inflates accuracy and doesn't reflect how the model would perform on a new patient. This project uses the stricter **inter-patient protocol** (DS1/DS2 split from MIT-BIH), so training and testing sets never share a patient.

The pipeline classifies beats into the 5 AAMI classes — N, S, V, F, Q and goes through:

- EMD-based feature extraction from raw beats, with MRMR feature selection
- S-class targeted SMOTE to handle the class imbalance (S and F beats make up less than 3% of the data combined)
- Three deep models trained on raw waveforms: a 1D-CNN, a BiLSTM, and the proposed multi-scale CNN-BiLSTM with dual attention
- Ensembling across those three (soft-voting, Optuna-weighted voting, stacking) and Optuna hyperparameter tuning of the proposed model
- SHAP and saliency-based explainability, so it's not just a black box
- External validation on INCART, a completely different 12-lead dataset, to see how well it actually generalizes

## Results

On MIT-BIH, inter-patient (DS1 → DS2 test set):

| Model / Strategy | Accuracy | Macro-F1 | Macro-AUC |
|---|---|---|---|
| CNN | 93.27% | 0.7533 | 0.9492 |
| BiLSTM | 72.43% | 0.5161 | 0.9366 |
| CNN-BiLSTM (proposed) | 82.62% | 0.5801 | 0.9379 |
| CNN-BiLSTM (Optuna-tuned) | 85.82% | 0.6712 | 0.9242 |
| Soft-voting ensemble | 88.12% | 0.6910 | 0.9540 |
| Weighted-voting ensemble | 93.62% | 0.7578 | 0.9533 |
| **Stacking ensemble (best overall)** | **94.08%** | **0.7611** | **0.9648** |

On INCART (external, out-of-distribution): accuracy 82.00%, **macro-F1 = 0.3578**.

I'm reporting that drop as-is rather than glossing over it. It's a real and expected consequence of domain shift, different lead setup, different patient population, different sampling rate (resampled 257→360 Hz), and different annotators, and it's exactly the kind of honest generalization gap that gets left out of a lot of papers claiming "cross-dataset validation."

## Repo structure
```
CNN-BiLSTM-Dual-Attention-ECG-Classification/
├── notebooks/
│   ├── PartA_Setup.ipynb                        # env setup, dataset download, AAMI mapping
│   ├── PartB_Preprocessing.ipynb                # denoising, R-peak detection, beat segmentation
│   ├── PartC_EMD_Features.ipynb                 # EMD decomposition, feature extraction
│   ├── PartD_Split_MRMR.ipynb                   # inter-patient DS1/DS2 split, MRMR, S-SMOTE
│   ├── PartE_DeepModels.ipynb                   # CNN, BiLSTM, CNN-BiLSTM w/ dual attention
│   ├── PartF_Ensemble_Optuna_SHAP.ipynb         # ensembling, Optuna tuning, SHAP, PTB-XL screening
│   └── PartG_INCART_External_Validation.ipynb   # full external validation on INCART
│
├── results/          # saved metrics
├── figures/          # confusion matrices, SHAP plots, training curves
├── requirements.txt
└── LICENSE
```

Notebooks are meant to be run in order each one picks up saved features/splits/models from the one before it.

## Running it

```bash
git clone https://github.com/rj05-ux/CNN-BiLSTM-Dual-Attention-ECG-Classification.git
cd CNN-BiLSTM-Dual-Attention-ECG-Classification
pip install -r requirements.txt
```

Best run on Google Colab with a GPU (Part E onward needs one) — `Runtime → Change runtime type → T4 GPU`.

## Datasets

- **MIT-BIH Arrhythmia Database** — 48 records, 47 subjects, 360 Hz. Used for training and the inter-patient test. Downloaded automatically in Part A via `wfdb`.
- **INCART Database** — St. Petersburg 12-lead Arrhythmia Database, 75 records from 32 patients. Used only in Part G, for external validation.

AAMI class mapping:

| Class | MIT-BIH symbols | Meaning |
|---|---|---|
| N | N, L, R, e, j | Normal + bundle branch |
| S | A, a, J, S | Supraventricular ectopic |
| V | V, E | Ventricular ectopic |
| F | F | Fusion beats |
| Q | /, f, Q | Unknown / paced |

## Requirements

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

## Citation

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

## License

MIT — see [LICENSE](LICENSE).

## Acknowledgements

- MIT-BIH Arrhythmia Database: Moody GB, Mark RG (PhysioNet)
- INCART Database: St. Petersburg Institute of Cardiological Technics (PhysioNet)
- AAMI EC57 standard for arrhythmia classification
