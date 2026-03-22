# Fraud Case Intelligence System

This project is an AI-powered fraud case intelligence system that translates model predictions into interpretable root-cause categories, analyst-facing summaries, and operational insights.

The project combined data science work — feature engineering, fraud modeling, and risk scoring — with product thinking to design a dashboard and decision-support workflow that helps teams investigate flagged cases, identify friction, and act on patterns more efficiently.

The core question this project tries to answer is:

**What patterns are driving fraud flags, where might the system be creating customer friction, and what should an analyst or ops team look at next?**

## Project Goals

- Build a baseline fraud-risk model for imbalanced customer fraud detection
- Preserve `consumer_id` for case traceability
- Convert model predictions into structured case intelligence
- Surface likely root causes and recurring behavioral patterns
- Identify possible false-positive or friction-heavy segments
- Present the results in a simple Streamlit analyst console

## Dataset

The data is merged on `consumer_id` from three domains:

- `datasets/user/`: user profile, email, delivery address data
- `datasets/item/`: latest item and order attributes
- `datasets/activity/`: per-day, per-week, and per-month behavioral activity

Key dataset facts:

- `16,485` customers
- About `6.5%` fraudulent customers
- Imbalanced binary classification problem
- Final merged modeling table contains behavioral, item, address, email, and activity signals

## Project Workflow

### 1. EDA and Cleaning

Notebook: [`01_eda_cleaning.ipynb`](01_eda_cleaning.ipynb)

Main work:

- merged source tables on `consumer_id`
- analyzed fraud vs non-fraud distributions
- handled missing values
- preserved `consumer_id` for traceability
- created a stratified `80/20` train-test split

Why stratification mattered:

- the fraud rate is only about `6.5%`
- stratified splitting preserved the fraud ratio across train and test
- this made evaluation more reliable for an imbalanced classification problem

### 2. Feature Engineering and Baseline Modeling

Notebook: [`02_feature_eng.ipynb`](02_feature_eng.ipynb)

Main work:

- loaded saved train and test sets
- used `StratifiedKFold` cross-validation on the training set
- encoded `latest_changed_password` with `OneHotEncoder`
- encoded high-cardinality fields like `latest_item_category` and `latest_item_product_title` with frequency encoding
- trained a baseline `LogisticRegression` model inside a preprocessing pipeline
- generated test-set risk scores and prediction outputs

Why cross-validation was used:

- to evaluate the model more reliably than a single validation split
- to preserve the fraud ratio in each fold
- to keep all preprocessing inside the model pipeline and avoid leakage

### 3. Case Intelligence Layer

Notebook: [`03_case_intelligence.ipynb`](03_case_intelligence.ipynb)

Main work:

- built `case_intel_df` from baseline predictions
- attached case-level context and behavioral features
- added `risk_band`
- created error flags such as true positives and false positives
- started building a rule-based intelligence layer for:
  - root-cause categories
  - reason codes
  - friction analysis
  - summary tables

This notebook shifts the project from:

`fraud prediction` -> `fraud case intelligence`

## Current Baseline Results

Current baseline model: `LogisticRegression`

Test-set metrics from the baseline workflow:

- `Average Precision (AP)`: `0.680`
- `ROC AUC`: `0.960`
- `Precision`: `0.42`
- `Recall`: `0.89`
- `F1-score`: `0.57`

Confusion matrix at the current threshold:

```text
[[2812  268]
 [  24  193]]
```

Interpretation:

- the baseline model separates fraud and non-fraud well overall
- recall is high, so most fraud cases are caught
- precision is lower, which means there is still meaningful customer friction from false positives
- this makes the intelligence and friction-analysis layer especially important

## Analyst-Facing Outputs

Saved case table:

- [`datasets/case_intelligence_baseline.csv`](datasets/case_intelligence_baseline.csv)

Current case table includes fields such as:

- `consumer_id`
- `y_true`
- `risk_score`
- `predicted_label`
- `risk_band`
- raw behavioral and item features
- error flags like `is_tp`, `is_fp`, `is_fn`, `is_tn`

This table is the foundation for:

- root-cause exploration
- case-level review
- friction analysis
- AI-generated summaries
- dashboard views

## Streamlit Dashboard

App file: [`app.py`](app.py)

The dashboard is designed as a **case intelligence console**, not a generic ML evaluation page.

It is meant to surface:

1. What kinds of cases are being flagged
2. Where customer friction may be happening
3. What individual flagged cases look like
4. What an analyst or ops team should review next

Current dashboard sections:

- `Overview`
- `Case Explorer`
- `Friction Analysis`

To run locally:

```bash
python -m pip install streamlit
python -m streamlit run app.py
```

## Project Structure

```text
Digital-Marketplace-Fraud-Prediction-Model/
├── 01_eda_cleaning.ipynb
├── 02_feature_eng.ipynb
├── 03_case_intelligence.ipynb
├── app.py
├── README.md
└── datasets/
    ├── activity/
    ├── item/
    ├── user/
    ├── train/
    ├── test/
    └── case_intelligence_baseline.csv
```

## Next Steps

- finalize `root_cause_category` and `reason_code_combo`
- add stronger category-level and friction-level summary tables
- generate AI postmortem summaries from the structured outputs
- improve threshold tuning based on business tradeoffs
- extend the dashboard with richer filtering and recommendations

## Why This Project Is Different

This is not just a fraud model repository.

The project is designed to simulate how a real fraud operations or escalation team works:

- score risk
- preserve traceability
- explain why cases were flagged
- identify repeat patterns
- surface likely friction
- turn model outputs into action
