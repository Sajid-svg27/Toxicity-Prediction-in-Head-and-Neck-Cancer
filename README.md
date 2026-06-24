# Toxicity Prediction – Xerostomia (Dry Mouth)

Predicting radiation-induced xerostomia in head-and-neck cancer patients using **CT radiomics**, **clinical variables**, and **dose-volume metrics**.

---

## Overview

Radiation-induced xerostomia is a common toxicity following head-and-neck radiotherapy. This project investigates whether combining radiomic descriptors with established dosimetric predictors can improve prediction of post-treatment dry mouth.

The proposed workflow employs a **late-fusion strategy**, where radiomic and clinical/dosimetric information are processed independently before being merged for classification.

---

## Dataset

### Radiomic Features
- Extracted from planning CT images on a per-patient basis
- Initial feature set: ~11,800 radiomic features
- Reduced to ~1,495 features after variance and correlation filtering

### Clinical and Dosimetric Features (22 variables)

Clinical variables include:

- Age
- Sex
- BMI
- Smoking status
- Alcohol consumption
- T-stage
- N-stage
- Chemotherapy status

Dose-volume metrics include:

- Left/Right Parotid Dmean
- Left/Right Submandibular Gland Dmean
- Oral Cavity Dmean
- Mandible Dmean
- Esophagus Dmean
- PCM Dmean
- Dose per fraction

### Target Variable

```text
Drymouth_binary
```

Binary endpoint:

- 0 → No xerostomia
- 1 → Xerostomia present

Patients with missing outcome labels were excluded.

Patient identifiers were used as the merge key between feature sets.

---

# Pipeline

Radiomic and clinical features are processed separately and merged only after feature reduction. This prevents the large radiomic feature space from overwhelming clinically established dose predictors.

## 1. Train/Test Split

- Single patient-level split
- 80/20 ratio
- Stratified on xerostomia status

The same patients define train and test partitions for both feature streams.

---

## 2. Radiomic Stream (Training Set Only)

### Missing Values

- Median imputation
- Features with more than **5% missing values** removed

### Feature Selection

**Variance Filtering**

Remove near-constant features.

↓

**Correlation Filtering**

Reduce redundancy among highly correlated radiomic descriptors.

↓

**mRMR**

Select top **150 features** using Maximum Relevance Minimum Redundancy.

↓

**Mann–Whitney U Test**

Non-parametric comparison between toxicity groups.

Benjamini–Hochberg False Discovery Rate correction applied.

↓

**Random Forest Feature Ranking**

Final reduction to approximately **10 radiomic features**.

---

## 3. Clinical Stream (Training Set Only)

- Median imputation for numerical variables
- One-hot encoding for categorical variables
- No aggressive feature selection performed

This preserves clinically relevant dose-volume metrics known to influence xerostomia risk.

---

## 4. Late Fusion

Selected radiomic features are concatenated with processed clinical and dosimetric variables to form the final feature matrix.

---

# Leakage Prevention

The test set is isolated before any preprocessing step.

All transformations are fitted exclusively on training data, including:

- Imputation
- Encoding
- Feature selection

## Permutation Validation

Labels were randomly shuffled multiple times.

Observed performance under permutation:

```text
AUC = 0.51 ± 0.06
```

This collapse to chance-level performance suggests the model is learning meaningful signal rather than exploiting information leakage.

---

# Modelling

Primary classifier:

### XGBoost

Configuration includes:

- Regularization
- Class imbalance correction using `scale_pos_weight`
- Early stopping

A neural network baseline was also evaluated, but tree-based models demonstrated superior performance on this relatively small tabular dataset.

---

# Evaluation Metrics

Performance was assessed on the held-out test cohort using:

- ROC–AUC
- Average Precision (AP)
- Confusion Matrix

---

# Results

| Metric | Value |
|-------|-------|
| Held-out ROC–AUC | **0.68** |
| Permutation AUC | 0.51 ± 0.06 |

The achieved performance is modest but consistently above chance.

Dose-volume metrics appear to provide the strongest predictive signal.

---




**Sajid Saleem**

Medical Physics | Radiomics | Machine Learning | Radiation Toxicity Prediction
