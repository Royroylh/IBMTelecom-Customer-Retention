# Model Outputs for Part D

Reproduced on 4 November 2026 (Hong Kong) by running the original `scripts/model_decision_tree.py` from repository commit `5a6d5b8619b4fcf92d66e8dc3157b894b00cc255`. The modelling code, data cleaning and parameter-selection procedure were unchanged. These files were generated in this reproduction run; they were not retrieved from member C's original local output directory.

Source: [IBMTelecom-Customer-Retention](https://github.com/onoizu/IBMTelecom-Customer-Retention/tree/5a6d5b8619b4fcf92d66e8dc3157b894b00cc255).

## Files

| File | Rows | Purpose |
|---|---:|---|
| test_predictions.csv | 1,409 | Test customer IDs, observed labels, risk scores, predictions, leaf IDs and segment IDs. Join to the repository's telco_clean.csv one-to-one on customerID. |
| DT_profiles.csv | 15 | Test-set profiles for each leaf. n, churn_count and churn_rate describe test customers; train_n and train_churn_rate describe training customers. |
| priority_segments.csv | 3 | Training, validation and test counts for S1–S3, test churn rates, median monthly charges and tenure, and stability statistics. |
| DT_rules.csv | 15 | All leaf rules and training statistics. conditions_json preserves the exact conditions. |
| data_split.csv | 7,043 | Each customer's assignment to train, validation or test. |

Raw data, cleaned data and existing EDA tables are not duplicated here. This directory supplies model outputs that were absent from the repository and the GP folder when checked. The repository ignores `reports/model/`, so running the script locally does not automatically add its outputs to GitHub.

## Definitions and use

- Use the 1,409 test customers as the main population for Part D's segment evidence, consistent with member C's PDF. Join business attributes using `customerID`, not row position. Do not describe test-set statistics as covering all 7,043 customers.
- `actual_churn`: 1 means observed churn; 0 means no observed churn. `DT_prediction` is the predicted class using the threshold selected on validation data. `DT_risk` is the model risk score, which differs from the segment's observed test churn rate.
- S1, S2 and S3 are the three priority leaves selected by training churn proportions. `Other` combines the remaining 12 leaves and is not one homogeneous customer group.
- Probabilities, churn rates, category shares and confidence intervals are stored as fractions from 0 to 1. Multiply by 100 to display percentages. Columns named `field=category` in `DT_profiles.csv` contain the category's share of test customers within that leaf.
- `tenure` is measured in months. Charges retain the source units; no currency has been assumed. Median monthly charges are not total revenue or customer lifetime value.
- S1: monthly contract, fiber optic, `tenure <= 1.5`. S2: monthly contract, fiber optic, `1.5 < tenure <= 15.5`. S3: monthly contract, non-fiber service, `tenure <= 3.5`, and `TechSupport=No`. Within this dataset's category combinations, S3 corresponds to DSL. Refer to `DT_rules.csv` for the exact model conditions.
- All five CSV files were copied directly from the original script's outputs, preserving column names and numerical precision. No additional merged customer table or duplicate EDA output was created.

## Verification

Training, validation and test sizes are 4,225 / 1,409 / 1,409. Test customer IDs are unique and agree with `data_split.csv`. Observed labels, leaf rules, segment assignments and threshold-based predictions were checked for every test customer. Priority-group counts, churn counts and medians were independently recalculated after joining the cleaned data by ID.

S1: 40/44 churners (90.9%). S2: 102/166 (61.4%). S3: 37/60 (61.7%). Together, these groups contain 270 test customers and 179 churners, matching the existing PDF. The confusion matrix is TN=813, FP=222, FN=115 and TP=259. ROC-AUC=0.826, AP=0.600 and F1=0.606, rounded to the report's precision.

File hashes, recorded software versions, the threshold and verification results are in `reproduction_manifest.json`. Reproduction does not create a new independent test set. The original report's limitations concerning prior test-set inspection, sensitivity of exact rules and the distinction between association and causation still apply.

## Reproduction

At the repository commit above, create an isolated Python 3.12 environment, install `requirements-model.txt`, and run the following from the repository root:

```bash
python -m pip install -r requirements-model.txt
python scripts/model_decision_tree.py
```

All five CSV files in this directory come from the resulting `reports/model/` outputs. No alternative modelling implementation was used.
