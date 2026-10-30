# Journal-review notes and pre-release checks

The notebook is organized into documented, sequential sections and retains the supplied estimator definitions, parameters, split, feature-selection logic, SMOTE procedure, and saved output snapshot. The following items should be checked against the final manuscript before the repository is made public or supplied to a journal.

## 1. Encoding description

The paper's preprocessing subsection states that categorical variables are imputed and one-hot encoded. The active notebook switches instead convert the ordered questionnaire fields and selected nominal fields into fixed integer-coded columns. For the retained run, `categorical_cols` is empty and the one-hot branch is inactive. The manuscript and notebook should describe the same feature representation. Either revise the manuscript to accurately document the active fixed mappings and their treatment by the estimators, or change the code and rerun the complete experiment; do not mix the two descriptions.

## 2. Cross-validation boundary

The notebook fits imputation/scaling on the full 80% training partition before creating the five CV folds. Mutual-information ranking is also computed once on this full training partition. SMOTE is fold-specific and only applied to each CV training subset. Consequently, the validation folds are not fully isolated from preprocessing and feature-selection fitting. The notebook documents this and preserves the experiment. A fully nested pipeline would require a new experiment and updated results, not a documentation-only edit.

## 3. Data availability and privacy

The CSV was not included in the supplied files for repository packaging. Before journal submission, add a clear data availability statement consistent with the actual consent/ethics position. Do not upload raw open-ended learner feedback, identifiable fields, sentiment audit exports, predictions, or fitted artifacts without checking permissions and privacy risk.

## 4. Re-run check

After placing the permitted dataset in `data/raw/`, install `requirements.txt`, launch Jupyter from the repository root, and run the notebook from top to bottom in a clean environment. Confirm that all ten model families are available, that the expected feature counts and class counts appear, and that the generated final metrics match the manuscript. The notebook's embedded outputs are retained reference snapshots and were not freshly regenerated during packaging.

## 5. Release metadata

Before public release, update `CITATION.cff` and the README with the final author list, journal citation, DOI, repository URL, and selected code license. No author details, DOI, public data URL, or license was inferred in this package.
