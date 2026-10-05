# Retention Intelligence Platform

## Model Card

**Document status:** Initial specification
**Version:** 1.0
**Status:** Pre-training

---

# 1. Model Overview

The Retention Intelligence Platform will contain multiple models rather than one universal model.

Initial components:

```text
Churn Propensity Model
Survival Model
CLV Model
Uplift Model
Decision Policy
```

Each component will have its own evaluation and limitations.

---

# 2. Intended Use

The models are intended to support:

- customer retention prioritization;
- risk assessment;
- customer value estimation;
- intervention planning;
- retention budget allocation;
- decision-support dashboards.

The models are decision-support systems.

They are not autonomous replacements for business judgment.

---

# 3. Out-of-Scope Uses

The models must not be used to:

- make unrelated credit decisions;
- determine legal eligibility;
- infer sensitive personal characteristics;
- make unsupported causal claims;
- directly transfer public-dataset predictions into production without validation.

---

# 4. Training Data

Primary predictive experiments:

```text
IBM Telco Customer Churn
```

Uplift methodology:

```text
Criteo Uplift Prediction Dataset
```

These datasets represent different analytical tasks and are not merged.

---

# 5. Model Inputs

Potential predictive inputs include:

```text
customer tenure
service configuration
contract type
billing characteristics
payment method
customer demographics where appropriate
service adoption
derived customer-state features
```

Final feature list will be determined after data validation and feature engineering.

---

# 6. Targets

Primary targets:

```text
30-day churn
90-day churn
180-day churn
time-to-churn
future customer value
treatment uplift
```

Not every target can be reliably constructed from every public dataset.

The model card must state the actual target used by each trained model.

---

# 7. Evaluation

Churn:

```text
PR-AUC
ROC-AUC
Brier score
calibration
precision/recall at operating point
```

Survival:

```text
Concordance index
Integrated Brier Score
calibration
```

CLV:

```text
MAE
RMSE
segment-level calibration
```

Uplift:

```text
Qini
AUUC
policy value
incremental gain
```

---

# 8. Model Selection

Model selection must consider:

```text
predictive performance
calibration
stability
interpretability
latency
resource requirements
maintenance complexity
business value
```

The highest benchmark score does not automatically win.

---

# 9. Baselines

Every advanced model must be compared against a baseline.

Potential baselines:

```text
majority-class model
logistic regression
simple survival model
simple CLV baseline
simple treatment-effect baseline
```

If a complex model cannot materially outperform a simpler baseline under realistic operating conditions, the complex model should not be deployed.

---

# 10. Explainability

The system should provide:

```text
global feature importance
local explanations
feature contribution
risk drivers
```

Explainability is intended to support:

- debugging;
- analyst review;
- business interpretation;
- model governance.

Explanations must not be treated as causal explanations.

---

# 11. Calibration

Probability outputs must be evaluated for calibration.

Example:

A group of customers receiving predicted probability:

```text
0.70
```

should experience churn at approximately 70% over the corresponding target definition if the model is well calibrated.

---

# 12. Known Limitations

The public datasets have important limitations.

## Static customer snapshot

The IBM Telco dataset does not provide the complete longitudinal event history required for a genuine production-grade temporal churn system.

## Causal limitation

Observational churn data cannot establish that an intervention will cause retention.

## Population limitation

Public datasets may not represent the target company's customers.

## Economic limitation

Real intervention costs and contribution margins may be unavailable.

---

# 13. Fairness & Responsible Use

Sensitive or protected attributes must not be used casually.

The project will evaluate whether model performance differs materially across relevant customer groups where such analysis is appropriate and legally permissible.

Potential metrics:

```text
calibration by group
recall by group
false-positive rate
false-negative rate
```

Fairness analysis must consider the business context and data limitations.

---

# 14. Production Requirements

Before production deployment:

```text
data validation
model validation
bias assessment
security review
performance testing
drift baseline
rollback mechanism
model versioning
monitoring
```

must be complete.

---

# 15. Model Versioning

Every model artifact must contain:

```text
model_version
training_dataset_version
feature_version
training_timestamp
code_version
hyperparameters
evaluation_results
```

---

# 16. Deployment Policy

A candidate model must pass validation before promotion.

Conceptually:

```text
TRAIN
 ↓
VALIDATE
 ↓
TEST
 ↓
MODEL CARD
 ↓
CONTAINERIZE
 ↓
DEPLOY TO STAGING
 ↓
MONITOR
 ↓
PROMOTE
```

---

# 17. Rollback

Every production deployment must have a known previous stable version.

If the candidate model causes:

```text
prediction failure
latency degradation
unexpected output distribution
severe drift
business degradation
```

the system must support rollback.

---

# 18. Model Card Principle

> **A model is not production-ready because it scores well on a test set. It is production-ready only when its data assumptions, limitations, operational behavior and business consequences are understood and controlled.**
