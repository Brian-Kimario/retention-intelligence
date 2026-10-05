# Retention Intelligence Platform

## Data Strategy & Data Architecture

**Document status:** Foundational
**Version:** 1.0

---

# 1. Purpose

This document defines how data will be acquired, structured, validated, transformed and consumed by the Retention Intelligence Platform.

The central requirement is:

> **Every model must operate on information that would genuinely have been available at the time the prediction was made.**

Temporal correctness is a first-class data requirement.

---

# 2. Data Architecture Principle

The project will distinguish:

```text
RAW DATA
    ↓
VALIDATED DATA
    ↓
CUSTOMER STATE
    ↓
FEATURE DATASET
    ↓
MODEL DATASET
    ↓
PREDICTION
    ↓
DECISION
    ↓
OUTCOME
```

The raw source must never be silently overwritten by preprocessing.

---

# 3. Data Layers

## 3.1 Raw Layer

Contains source data exactly as acquired.

Examples:

```text
data/raw/
├── telco/
└── uplift/
```

Requirements:

- preserve original files;
- record source URL;
- record acquisition date;
- record source version where available;
- calculate file checksum where practical;
- never modify raw data in place.

---

## 3.2 Validated Layer

Contains data that has passed structural and quality checks.

Checks include:

- schema;
- data types;
- missingness;
- duplicate records;
- valid categorical values;
- numerical ranges;
- date validity;
- impossible values;
- target consistency.

---

## 3.3 Customer-State Layer

The central analytical abstraction is a **customer snapshot**.

Conceptually:

```text
customer_id
snapshot_date
feature_1
feature_2
...
feature_n
```

Each row answers:

> "What did we know about this customer at this point in time?"

This is more important than simply having one row per customer.

---

## 3.4 Feature Layer

Features are derived from customer history without using future information.

Examples:

```text
tenure
recent_activity
activity_frequency
activity_change
revenue_30d
revenue_90d
service_adoption
support_contacts
payment_failures
contract_status
```

Where historical event data is available, features should be generated relative to `snapshot_date`.

---

# 4. Data Categories

The target production architecture should conceptually support:

## Customer identity

```text
customer_id
account_age
customer_segment
```

## Product/service

```text
services_used
plan
feature_adoption
usage
```

## Financial

```text
revenue
charges
payment_behavior
discounts
```

## Engagement

```text
activity_frequency
recency
session/activity trend
```

## Support

```text
tickets
complaints
resolution time
```

## Contract

```text
contract type
renewal status
billing cycle
```

## Temporal

```text
snapshot_date
tenure
rolling behavior
trend
```

---

# 5. Data Acquisition Strategy

The project will use two analytically distinct public data sources.

## Primary predictive dataset

A public customer churn dataset will be used to demonstrate:

- customer-state construction;
- churn propensity;
- survival analysis;
- value modeling;
- production scoring.

The preferred initial dataset is IBM Telco Customer Churn because it provides a compact customer-level subscription dataset with demographic, service, billing and churn attributes.

---

## Causal / uplift dataset

The Criteo Uplift Prediction Dataset will be used separately to demonstrate:

- randomized treatment/control data;
- uplift modeling;
- heterogeneous treatment effects;
- policy evaluation.

Criteo's dataset contains treatment assignment and response outcomes and was designed for uplift/incrementality research.

---

# 6. Critical Dataset Boundary

The datasets MUST NOT be joined.

They represent different populations and experiments.

The project will instead demonstrate:

```text
CUSTOMER RETENTION DATA
        ↓
Risk / Survival / CLV
        ↓
Retention decision architecture


CRITEO RANDOMIZED DATA
        ↓
Uplift / causal methodology
        ↓
Treatment-response methodology
```

The project will explicitly state this limitation.

---

# 7. Production Data Model

A future real business implementation should contain entities conceptually similar to:

```text
customers
subscriptions
transactions
customer_events
product_usage
support_events
interventions
intervention_outcomes
customer_snapshots
predictions
model_versions
decision_records
```

The public-data implementation will approximate these concepts only where the source data supports them.

Unsupported fields must not be fabricated and presented as observed data.

---

# 8. Data Contracts

Every production input should eventually have a contract defining:

```text
field
type
allowed range
nullable
source
freshness
business meaning
```

Example:

```text
monthly_charges
type: numeric
nullable: false
range: >= 0
meaning: recurring customer charge
```

Data contract violations should be observable and testable.

---

# 9. Data Leakage Controls

The following are mandatory.

### No future target information

A prediction made at:

```text
2026-10-01
```

must not use events after:

```text
2026-10-01
```

unless those events were genuinely available in real time.

### No post-churn information

Features must not indirectly encode the churn outcome.

### No future aggregates

Rolling features must use historical windows only.

Incorrect:

```text
rolling_mean(current + future)
```

Correct:

```text
historical_window_before_prediction_time
```

---

# 10. Data Splitting Strategy

Time-dependent data must use chronological evaluation.

Conceptually:

```text
TRAIN
────────────────────────

VALIDATION
────────────────

TEST
────────────────

FUTURE HOLDOUT
────────────────
```

Random row-level splitting is prohibited for the primary temporal evaluation if it allows future customer states to influence training.

---

# 11. Data Quality Monitoring

Production monitoring should track:

```text
schema changes
missingness
duplicates
invalid values
feature freshness
category changes
distribution shifts
unexpected volume
```

A model should not receive silently corrupted data.

---

# 12. Data Lineage

Every model version should be traceable to:

```text
source dataset
source version/date
processing version
feature version
training period
model version
configuration
evaluation results
```

The objective is reproducibility.

---

# 13. Data Privacy

A real implementation must apply:

- data minimization;
- access control;
- encryption;
- appropriate retention;
- pseudonymization;
- auditability.

The public project must not expose personally identifiable information.

---

# 14. Data Strategy Principle

> **The dataset is not merely model input. It is the evidence supporting every business decision produced by the system.**

Therefore data quality, temporal validity and lineage are part of model quality.
