# EduDropX — Online Learning Dropout Prediction

**Reproducibility repository for the EduDropX research paper**

EduDropX is a sentiment-enhanced, multimodal, explainable machine-learning framework for binary online-learning dropout prediction. This repository contains the journal-review notebook, its execution environment, and documentation of the experimental workflow.

> **Data availability:** The original learner-level CSV is not included in this repository package. The notebook requires `online_learning_dropout_dataset_1200.csv`. Do not publish raw learner records or open-ended feedback unless the study's participant consent, ethics approval, and institutional data-sharing rules explicitly allow it.

## Repository contents

```text
EduDropX_GitHub_Repository/
├── README.md
├── requirements.txt
├── CITATION.cff
├── .gitignore
├── notebooks/
│   └── EduDropX_Journal_Review_Notebook.ipynb
├── data/
│   ├── README.md
│   └── raw/
│       └── README.md
├── docs/
│   ├── EXPERIMENTAL_PROTOCOL.md
│   └── JOURNAL_REVIEW_NOTES.md
└── results/
    └── README.md
```

## Study snapshot

The accompanying paper reports **1,200 learner observations** and **32 original columns**. The target contains **739 non-dropouts (61.58%)** and **461 dropouts (38.42%)**. Four open-ended questionnaire responses are processed separately using the pretrained `ahs95/banglabert-sentiment-analysis` model. The resulting probability-weighted sentiment scores are consolidated into one learner-level feature and combined with structured variables.

| Experimental setting | Reported Random Forest test accuracy | Reported test F1 | ROC-AUC | PR-AUC |
|---|---:|---:|---:|---:|
| All-features (26 predictors) | 94.17% | 92.47% | 98.58% | 97.89% |
| Top-20 (20 predictors) | 93.75% | 92.06% | 98.58% | 97.90% |

These values are transcribed from the supplied paper and retained notebook outputs. The notebook's saved outputs are a reference snapshot; they were not freshly regenerated while preparing this repository package.

## Quick start

### 1. Prepare the data

Place the authorized dataset file at:

```text
data/raw/online_learning_dropout_dataset_1200.csv
```

The notebook expects the CSV to contain the schema declared in Section 2 of the notebook, including the binary `target` column. Target coding is `0 = non-dropout` and `1 = dropout`. The notebook validates the column schema and target values before continuing.

The data file is intentionally excluded from Git by `.gitignore`. If the data may legally and ethically be shared, decide on an approved release mechanism first and update the data-access documentation. Do not remove the ignore rule merely to make the notebook run on GitHub.

### 2. Create an environment

Python **3.10 or 3.11** is recommended for compatibility across the notebook's scientific, NLP, and neural-network dependencies.

```bash
python -m venv .venv
```

Activate the environment:

- **Windows PowerShell:** `.\.venv\Scripts\Activate.ps1`
- **macOS/Linux:** `source .venv/bin/activate`

Install dependencies:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

For GPU-enabled PyTorch, install the PyTorch build appropriate for the machine from the official PyTorch installation selector, then install the remaining requirements. The first sentiment-extraction run downloads the pretrained BanglaBERT model from Hugging Face; network access is needed unless the model is already cached locally.

### 3. Launch and run

Run Jupyter from the repository root so the notebook can locate `data/raw/` and write outputs to the root-level `thesis_results/` directory:

```bash
jupyter lab
```

Open `notebooks/EduDropX_Journal_Review_Notebook.ipynb` and execute the cells **from top to bottom**. The first run can take longer because the sentiment model and the classifier experiments must be loaded and evaluated. The sentiment score cache is stored under `thesis_results/02_sentiment/` and reused on later runs when its schema and row count match.

## Experimental workflow

1. Environment setup and source-data loading.
2. Target validation, questionnaire schema audit, and exclusion of post-outcome/leakage fields.
3. Deterministic cleaning of structured answers.
4. Four separate BanglaBERT sentiment extractions and learner-level sentiment aggregation.
5. Construction of the all-features representation and training/test partition.
6. Exploratory summaries and training-partition mutual-information ranking.
7. Ten classifier families across the all-features and top-20 scenarios, subject to the required packages being available.
8. Five-fold stratified development evaluation with fold-specific SMOTE on each training fold.
9. Held-out metrics, confusion matrices, ROC and precision–recall curves.
10. Top-three soft voting and SHAP-based interpretation.
11. Export of fitted estimators, summaries, plots, and run metadata.

The detailed procedure and implementation-specific qualifications are documented in [`docs/EXPERIMENTAL_PROTOCOL.md`](docs/EXPERIMENTAL_PROTOCOL.md) and [`docs/JOURNAL_REVIEW_NOTES.md`](docs/JOURNAL_REVIEW_NOTES.md).

## Generated artefacts

The notebook writes generated files to `thesis_results/`, including:

- processed feature tables and feature schema;
- sentiment scores and text-level audit exports;
- EDA summaries and figures;
- mutual-information rankings;
- fold-level and consolidated model metrics;
- held-out predictions and diagnostic figures;
- ensemble outputs;
- SHAP explanations and importance tables;
- fitted models and pipeline metadata.

This folder is ignored by Git by default because it may contain learner-derived outputs, text-level audit files, predictions, and large model binaries. Review every generated file before deciding whether any artefact is appropriate to publish. Aggregate tables and non-identifying figures can be added selectively when permitted.

## Reproducibility and interpretation cautions

- The active notebook switches and preprocessing behaviour are described in [`docs/EXPERIMENTAL_PROTOCOL.md`](docs/EXPERIMENTAL_PROTOCOL.md). These are important because the manuscript's one-hot-encoding description does not match the currently active fixed integer mappings.
- Preprocessing and mutual-information feature selection are fitted once on the full 80% training partition before the five CV folds. SMOTE is applied within each fold's training subset. The notebook documents this boundary; the CV values should not be described as a fully nested estimate of the complete preprocessing-and-selection procedure.
- SHAP values describe model associations, not causal effects.
- The study uses a specific Bangladeshi learner sample. The results should not be interpreted as proof of generalization to every institution, course, or learner population.
- Model performance depends on the exact dataset, package versions, hardware, and pretrained model revision. Retain the exported metadata when conducting a new run.

## Data and ethics

The dataset contains questionnaire responses, including open-ended learner feedback. Before making any input data, sentiment cache, prediction file, or model artifact public, verify that release is consistent with participant consent, ethics approval, institutional policy, and applicable privacy requirements. The repository does not include the original dataset or an anonymized sample.

See [`data/README.md`](data/README.md) for the data placement and sharing policy notes.

## Citation

Please cite the associated EduDropX manuscript when using this code. Add the final journal citation and DOI to this section once they are assigned. A machine-readable citation stub is provided in [`CITATION.cff`](CITATION.cff); update author details and the final repository URL before release if required by the journal.

## License

No open-source license has been assigned in this package. Until the authors select and add a license, the repository is visible for review but reuse and redistribution are not automatically granted. Choose a license only after all authors and the relevant institution agree.
