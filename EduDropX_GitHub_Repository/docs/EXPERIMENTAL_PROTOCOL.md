# Experimental protocol and implementation record

This document describes the configuration visible in the supplied journal-review notebook. It is intended to help reviewers trace the implemented workflow. It does not silently replace or reinterpret the paper's methodology.

## Data and target

- Source dataset described in the paper: 1,200 observations and 32 original columns.
- Structured questionnaire items: 21.
- Open-ended questionnaire responses: 4.
- Target: `target`, where 0 denotes non-dropout and 1 denotes dropout.
- Reported class counts: 739 non-dropouts (61.58%) and 461 dropouts (38.42%).
- Explicitly excluded columns include the timestamp, fully missing columns, `ever_dropped_out` (identical to the target), `dropout_reason` (post-outcome information), and the target itself.

## Sentiment extraction

The four open-ended responses are processed separately with `ahs95/banglabert-sentiment-analysis`. The model emits probabilities for five sentiment classes. The notebook computes the probability-weighted continuous score using the paper's mapping and consolidates the four response-level scores into `sentiment_combined`. The sentiment model is used as a pretrained feature extractor and is not fine-tuned on the dropout target. Blank responses receive a neutral score of 0.0 in the notebook.

## Active feature switches

The notebook retains these active switches:

```python
ORDERED_QUESTIONS_AS_SINGLE_COLUMN = True
MERGE_ALL_SENTIMENT_INTO_ONE = True
NOMINAL_QUESTIONS_AS_SINGLE_COLUMN = True
```

Under this configuration, ordered questionnaire fields and selected nominal fields are converted to fixed integer-coded columns. Profession codes are nominal category identifiers; their numeric values do not represent a natural ranking, although the downstream estimators receive them as numeric values. The four sentiment scores are averaged into one feature. The notebook's active run contains 26 numeric predictors, so the one-hot-encoding branch is inactive.

## Split, preprocessing, feature selection, and balancing

- Train/test split: stratified 80:20, `random_state=42` (960 training rows and 240 test rows for the reported dataset).
- Numeric preprocessing: median imputation followed by standardization.
- If any categorical columns remain after feature switches, the notebook's alternative branch uses most-frequent imputation and one-hot encoding.
- The preprocessing object is fitted once on the full 80% training partition, then applied to the held-out test partition.
- Mutual-information ranking is computed once on the processed full training partition; the 20 highest-ranked features define the reduced scenario.
- Five-fold stratified cross-validation uses `random_state=42` and shuffling.
- SMOTE is applied to the training subset of each CV fold, not to the corresponding validation fold. Final models are also fitted using SMOTE on the complete training partition before held-out test evaluation.

**Validation boundary:** preprocessing and mutual-information selection are not refitted separately inside each CV fold in this notebook. Thus the CV validation folds have contributed to the fit of the preprocessing/feature-selection objects through their inclusion in the full training partition. The held-out test partition remains separate from preprocessing, feature ranking, model selection, and ensemble selection. Do not describe this implementation as fully nested cross-validation or a fully fold-isolated preprocessing pipeline.

## Classifiers and settings

The notebook compares the following model families in each feature scenario:

- Random Forest: 500 trees, balanced class weights.
- Gradient Boosting: scikit-learn default estimator settings, with `random_state=42`.
- SVM: RBF kernel, `C=1.0`, probability estimates enabled, balanced class weights.
- MLP: hidden layers `(128, 64)`, early stopping, `max_iter=500`.
- Logistic Regression: `max_iter=2000`, balanced class weights.
- AdaBoost: 300 estimators and learning rate 0.05.
- XGBoost: 400 estimators, depth 5, learning rate 0.04, row and column subsampling 0.85, binary logistic objective.
- LightGBM: 400 estimators, learning rate 0.04, 31 leaves, row and column subsampling 0.85, balanced class weights.
- ANN: dense layers 128 and 64, dropout 0.30 and 0.20, Adam optimizer, 150-epoch ceiling, batch size 32, early-stopping patience 15.
- 1-D CNN: Conv1D layers with 32 and 16 filters, global max pooling, dense layer 32, dropout 0.25, Adam optimizer, 150-epoch ceiling, batch size 32, early-stopping patience 15.

XGBoost, LightGBM, and TensorFlow-based ANN/CNN execution depends on the corresponding packages being available. Model definitions and parameters are preserved in the notebook; they should not be changed when reproducing the reported outputs.

## Evaluation and interpretation

Reported metrics are accuracy, balanced accuracy, precision, recall, F1-score, ROC-AUC, and average precision (PR-AUC). Threshold-dependent metrics use a probability threshold of 0.5. The top-three soft-voting ensemble is selected by mean five-fold CV F1-score and averages the member models' predicted dropout probabilities. SHAP TreeExplainer is used for global and local interpretation of selected tree models.

## Main reported results

The paper reports the following Random Forest held-out results:

| Scenario | Accuracy | Balanced accuracy | F1 | ROC-AUC | PR-AUC |
|---|---:|---:|---:|---:|---:|
| All-features | 94.17% | 94.04% | 92.47% | 98.58% | 97.89% |
| Top-20 | 93.75% | 93.90% | 92.06% | 98.58% | 97.90% |

These are manuscript-reported values and retained notebook output values; this package-preparation step did not rerun the training workflow.
