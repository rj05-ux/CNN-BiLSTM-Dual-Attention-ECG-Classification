# Setting up this repo on GitHub

Quick notes on how this project ended up on GitHub, in case future-me (or
anyone else) needs to reproduce or extend the setup. Everything here was
done through the GitHub web UI — no local `git` CLI involved.

## The repo

- **Name:** `CNN-BiLSTM-Dual-Attention-ECG-Classification`
- **Owner:** `rj05-ux`
- **Link:** https://github.com/rj05-ux/CNN-BiLSTM-Dual-Attention-ECG-Classification
- Public, since the whole point is reproducibility for the paper (Multi-Scale
  CNN-BiLSTM with Dual Attention, MIT-BIH + INCART, AAMI EC57 inter-patient
  protocol)

## What's actually in the repo

```
CNN-BiLSTM-Dual-Attention-ECG-Classification/
├── notebooks/     — the 7 notebooks, Parts A through G
├── results/       — metrics only (csv/json/xlsx), no raw arrays
├── figures/       — plots and images
├── README.md      — was already there
├── LICENSE        — was already there
├── requirements.txt
├── .gitignore
└── docs/          — added if there's anything worth putting there
```

**notebooks/** has all seven parts: Setup, Preprocessing, EMD Features,
Split & MRMR, Deep Models, Ensemble/Optuna/SHAP, and INCART External
Validation.

**results/** only has the small stuff training histories, metrics
summaries, comparison tables. Left out all the raw `.npy` prediction and
feature arrays (proba files, beats, labels, record IDs) since those aren't
really "results," they're intermediate data, and one of them
(`G_incart_beats.npy`) is nearly 250MB anyway.

**src/** doesn't exist. Turns out there's no code living outside the
notebooks — it's all inline. Nothing to upload here. If I ever pull reusable
functions out into standalone `.py` files, this is where they'd go.

## What's deliberately NOT on GitHub

- `incart/` and `mitdb/` — raw datasets, way too big, and the README
  already tells people to grab them via `wfdb` automatically
- `models/` and `checkpoints/` — trained weights, often 50MB+, right up
  against GitHub's 100MB file cap. If I want to share a trained model
  later, Drive or HuggingFace with a README link makes more sense
- `.venv/` — never belongs in version control, obviously
- The raw `.npy` arrays mentioned above

## .gitignore

```
.venv/
incart/
mitdb/
models/
checkpoints/
__pycache__/
*.pyc
```

## How the upload actually worked

Since this was all done in the browser rather than with git commands:

1. Create the folder by adding a file at `foldername/.gitkeep` (GitHub
   won't track an empty folder any other way)
2. Go into that folder, use Upload files, drag everything in, write a
   commit message, commit to `main`

Did this for `notebooks/`, `results/`, and `figures/` one at a time.

## Repo settings worth having

- Topics: `ecg`, `arrhythmia`, `deep-learning`, `cnn`, `bilstm`,
  `attention`, `mit-bih`, `incart`, `classification`, `healthcare-ai`
- Issues and Discussions turned on
- Colab badge in the README pointing at:
  `https://colab.research.google.com/github/rj05-ux/CNN-BiLSTM-Dual-Attention-ECG-Classification/blob/main/notebooks/PartA_Setup.ipynb`
