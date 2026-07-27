# ECG Arrhythmia Classification

Inter-patient ECG arrhythmia classification using a multi-scale CNN-BiLSTM with dual attention, evaluated on MIT-BIH and INCART.

This is the code behind our paper, **"An Efficient Multi-Scale CNN-BiLSTM with Dual Attention Mechanism for Inter-Patient ECG Arrhythmia Classification: Evaluation on MIT-BIH and INCART Databases,"** published in JETIR (Vol. 13, Issue 7, July 2026). [Read it here](http://www.jetir.org/view?paper=JETIR2607122).

## What this actually does

Most papers on ECG beat classification split their data randomly, which lets the same patient's heartbeats leak into both the train and test sets — that inflates accuracy and doesn't reflect how the model would perform on a new patient. This project uses the stricter **inter-patient protocol** (DS1/DS2 split from MIT-BIH), so training and testing sets never share a patient.

The pipeline classifies beats into the 5 AAMI classes — N, S, V, F, Q — and goes through:

- EMD-based feature extraction from raw beats
- SMOTE to handle the class imbalance (S and F beats make up less than 3% of the data combined)
- A few baselines (Random Forest, SVM, LSTM, 1D-CNN) for comparison
- The main model: a multi-scale CNN-BiLSTM with dual attention
- Ensembling (soft-voting, Optuna-weighted voting, stacking) and Optuna hyperparameter tuning
- SHAP and saliency-based explainability, so it's not just a black box
- External validation on INCART, a completely different 12-lead dataset, to see how well it actually generalizes

## Results

On MIT-BIH, inter-patient (DS1 → DS2):

| Model | Accuracy | Macro-F1 |
|---|---|---|
| Random Forest | 92.93% | 0.520 |
| SVM | — | 0.710 (best S-class sensitivity) |
| LSTM | 64.83% | — |
| 1D-CNN | 88.25% | 0.530 |
| CNN-BiLSTM (Optuna-tuned) | — | 0.672 |
| **Stacking ensemble (best overall)** | **94.45%** | **76.66%** |

On INCART (external, out-of-distribution): macro-F1 drops to **35.79%**.

I'm reporting that drop as-is rather than glossing over it. It's a real and expected consequence of domain shift — different lead setup, different patient population, different annotation process — and it's exactly the kind of honest generalization gap that gets left out of a lot of papers claiming "cross-dataset validation."

## Repo structure

```
notebooks/
  PartA_Setup.ipynb                       – env setup, dataset download, AAMI mapping
  PartB_Preprocessing.ipynb               – denoising, R-peak detection, beat segmentation
  PartC_EMD_Features.ipynb                – EMD decomposition, feature extraction
  PartD_Split_MRMR.ipynb                  – inter-patient DS1/DS2 split, MRMR, SMOTE
  PartE_DeepModels.ipynb                  – 1D-CNN, LSTM, CNN-BiLSTM w/ dual attention
  PartF_Ensemble_Optuna_SHAP.ipynb        – ensembling, Optuna tuning, SHAP, PTB-XL screening
  PartG_INCART_External_Validation.ipynb  – full external validation on INCART

results/    – saved metrics
figures/    – confusion matrices, SHAP plots, training curves
requirements.txt
LICENSE
```

Notebooks are meant to be run in order — each one picks up saved features/splits/models from the one before it.

## Running it

```bash
git clone https://github.com/YOUR_USERNAME/ecg-arrhythmia-classification.git
cd ecg-arrhythmia-classification
pip install -r requirements.txt
```

Best run on Google Colab with a GPU (Part E onward needs one) — `Runtime → Change runtime type → T4 GPU`.

## Datasets

- **MIT-BIH Arrhythmia Database** — 48 records, 47 subjects, 360 Hz. Used for training and the inter-patient test. Downloaded automatically in Part A via `wfdb`.
- **INCART Database** — St. Petersburg 12-lead Arrhythmia Database. Used only in Part G, for external validation.

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
