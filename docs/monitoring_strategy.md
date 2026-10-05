# Retention Intelligence Platform

## Monitoring Strategy

**Document status:** Foundational
**Version:** 1.0

---

# 1. Monitoring Objective

Monitoring exists to answer five questions:

```text
1. Is the service alive?
2. Is the data healthy?
3. Is the model behaving correctly?
4. Has the data or customer population changed?
5. Is the business decision quality deteriorating?
```

Prometheus and Grafana will provide the primary operational monitoring layer.

Drift detection will provide statistical monitoring.

---

# 2. Monitoring Architecture

```text
                  ML SERVICE
                      │
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
   API Metrics    Data Metrics   Model Metrics
       │              │              │
       └──────────────┼──────────────┘
                      ↓
                 Prometheus
                      ↓
                  Grafana
                      │
                      ↓
                 Dashboards
```

Drift detection operates alongside this:

```text
Reference Data
      │
      ↓
Drift Engine
      │
      ↓
Drift Metrics / Alerts
      │
      ↓
Prometheus → Grafana
```

---

# 3. Monitoring Layers

The project will monitor four layers.

## Layer 1 — Infrastructure

Monitor:

```text
CPU
memory
container restarts
request count
request latency
HTTP errors
pod availability
```

---

## Layer 2 — Data Quality

Monitor:

```text
missingness
schema changes
invalid values
categorical changes
feature freshness
input volume
```

---

## Layer 3 — Model Health

Monitor:

```text
prediction distribution
prediction confidence
class distribution
calibration where labels become available
performance metrics
```

---

## Layer 4 — Business / Decision Health

Monitor:

```text
targeting rate
intervention rate
expected CLV at risk
intervention cost
retention outcome
incremental value
```

---

# 4. Prometheus Responsibilities

Prometheus will collect time-series metrics from the application.

Example metrics:

```text
prediction_requests_total
prediction_request_latency_seconds
prediction_errors_total

model_predictions_total
model_prediction_probability
model_version_info

input_validation_failures_total
data_drift_events_total
```

Prometheus is primarily an **observability/metrics system**.

It is not the feature store, model registry or primary data warehouse.

---

# 5. Grafana Responsibilities

Grafana will provide dashboards.

Initial dashboards:

```text
01 — Service Health
02 — Data Quality
03 — Model Performance
04 — Drift
05 — Business Decisions
```

---

# 6. Drift Types

The project will distinguish:

## Data drift

Input distribution changes.

Example:

```text
monthly_charges
```

distribution changes substantially.

---

## Prediction drift

Model output distribution changes.

Example:

```text
average churn probability:
0.21 → 0.48
```

This may indicate:

- population change;
- data pipeline issue;
- business change;
- model degradation.

Prediction drift alone does not prove model failure.

---

## Concept drift

Relationship between features and target changes.

Example:

```text
same customer behavior
↓
different churn probability in reality
```

This requires observed outcome labels and cannot always be detected immediately.

---

# 7. Drift Detection Methods

Initial implementation:

```text
Numerical:
PSI
KS test

Categorical:
Jensen-Shannon divergence
frequency comparison
```

The project should not blindly use one drift metric.

Different distributions require appropriate statistical treatment.

---

# 8. Reference Dataset

A reference dataset will represent the expected model input distribution.

Example:

```text
reference/
    training_distribution.parquet
```

Production data will be compared against this baseline.

---

# 9. Drift Window

Initial approach:

```text
reference:
training data

current:
rolling 7-day production window
```

For low-volume data:

```text
rolling 30-day window
```

may be more statistically meaningful.

The window should be configurable.

---

# 10. Alert Levels

## INFO

Small distribution movement.

No action.

## WARNING

Meaningful distribution change.

Investigate.

## CRITICAL

Severe drift or data-quality failure.

Potentially stop or restrict automated decisions.

---

# 11. Example Drift Policy

Conceptual:

```text
PSI < 0.10
→ normal

0.10–0.25
→ investigate

> 0.25
→ significant drift
```

These thresholds are starting conventions, not universal truths.

They must be validated against project behavior.

---

# 12. Model Performance Monitoring

Performance can only be measured after outcomes become available.

Therefore:

```text
Prediction
    ↓
Wait for outcome
    ↓
Join prediction + outcome
    ↓
Calculate performance
```

The monitoring system must distinguish:

```text
REAL-TIME OBSERVABILITY
```

from:

```text
DELAYED MODEL PERFORMANCE MONITORING
```

This distinction is critical.

---

# 13. Prediction Monitoring

Track:

```text
prediction count
mean probability
median probability
probability quantiles
positive-risk percentage
decision distribution
```

Sudden changes can reveal:

- broken preprocessing;
- upstream data change;
- model deployment issue;
- genuine customer population change.

---

# 14. Business Monitoring

Track:

```text
customers scored
customers targeted
targeting rate
intervention cost
expected value
realized retention
realized CLV
```

The final objective is not merely:

```text
low latency
```

but:

```text
better retention economics
```

---

# 15. Kubernetes Monitoring

Kubernetes-level monitoring will include:

```text
pod availability
pod restarts
CPU utilization
memory utilization
deployment status
replica availability
```

Prometheus will collect Kubernetes/application metrics.

---

# 16. Alerting

Alerts should be actionable.

Bad alert:

```text
"Model drift detected."
```

Better:

```text
"monthly_charges PSI = 0.31 over the last 7 days.
Investigate billing-data distribution change."
```

Alerts should contain:

```text
metric
current value
threshold
time window
severity
recommended investigation
```

---

# 17. Monitoring Does Not Automatically Trigger Retraining

Drift detection should NOT automatically mean:

```text
drift
→ retrain
```

Correct workflow:

```text
Drift detected
      ↓
Investigate
      ↓
Determine cause
      ↓
Validate data
      ↓
Evaluate model
      ↓
Retrain if justified
      ↓
Validate candidate
      ↓
Deploy
```

Automatic retraining without diagnosis is dangerous.

---

# 18. Monitoring Principle

> **Monitoring should detect changes and create evidence for decisions. It should not blindly automate responses to statistical signals.**
