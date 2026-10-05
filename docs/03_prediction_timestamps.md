# Retention Intelligence Platform

## Prediction Timestamp & Temporal Design

**Document status:** Foundational
**Version:** 1.0

---

# 1. Purpose

This document defines exactly **when predictions are made**, what information is allowed to enter the model, and when the outcome becomes observable.

This is the project's primary defense against temporal leakage.

---

# 2. Core Concept

Every prediction is associated with a:

```text
T0 = prediction timestamp
```

At `T0`, the system knows:

```text
X(T0)
```

The model predicts a future event:

```text
Y(T0 → T+h)
```

where `h` is the prediction horizon.

---

# 3. Customer Snapshot

The fundamental unit of prediction is:

```text
(customer_id, snapshot_date)
```

Example:

```text
Customer: C12345
Snapshot: 2026-10-01 00:00
```

All features must represent information available by that timestamp.

---

# 4. Prediction Horizons

The project will initially support:

```text
30-day churn
90-day churn
180-day churn
```

Therefore:

### 30-day target

```text
T0
│
├──────────── 30 days ────────────>
│                                  │
Prediction                       Outcome
```

### 90-day target

```text
T0
│
├──────────────────── 90 days ───────────────────>
│                                                     │
Prediction                                          Outcome
```

### 180-day target

```text
T0
│
├──────────────────────────── 180 days ────────────────────────>
│                                                               │
Prediction                                                    Outcome
```

---

# 5. Feature Observation Window

Features should use explicitly defined historical windows.

Example:

```text
T0 = 2026-10-01

7-day window:
2026-09-24 → 2026-09-30

30-day window:
2026-09-01 → 2026-09-30

90-day window:
2026-07-03 → 2026-09-30
```

The current prediction timestamp must not be allowed to leak future information.

---

# 6. Feature Cutoff

The conceptual rule is:

```text
FEATURE_CUTOFF <= T0
```

For operational systems with delayed data, the safer rule is:

```text
FEATURE_CUTOFF <= T0 - DATA_LATENCY
```

Example:

If billing information becomes available with a two-day delay:

```text
T0 = October 10

Latest trustworthy billing information
= October 8
```

The model cannot assume October 9–10 information exists merely because the event technically occurred.

---

# 7. Churn Target

For a 30-day horizon:

```text
Y30 = 1
if churn occurs after T0
and before/equal to T0 + 30 days

Y30 = 0
otherwise
```

Equivalent conceptual representation:

```text
TARGET_WINDOW:

T0 -------------------------- T0+30d
     future observation
```

---

# 8. Multiple Horizons

The same customer snapshot can produce:

```text
Y30
Y90
Y180
```

Example:

```text
Customer C123

Snapshot:
2026-10-01

30-day churn:
0

90-day churn:
1

180-day churn:
1
```

This means the customer was not an immediate churn threat but became a medium-term risk.

---

# 9. Survival Analysis Timestamp

Survival analysis uses:

```text
entry time
event time
censoring status
```

Example:

```text
Customer enters observation:
January 1

Churn:
September 14

Observed survival duration:
256 days

Event:
1
```

If the observation period ends without churn:

```text
Observed survival duration:
300 days

Event:
0

Status:
Right censored
```

Censoring must not be incorrectly treated as non-churn.

---

# 10. CLV Timestamp

CLV estimation is also anchored to `T0`.

The question is:

> Given everything known at T0, what future economic value is expected?

Therefore:

```text
Historical information
        ↓
T0
        ↓
Future expected cash flows
        ↓
CLV
```

Future realized revenue must not be used as a feature.

It may only be used later for evaluation.

---

# 11. Intervention Timestamp

An intervention is a separate event:

```text
T0 = scoring time

T1 = intervention time
```

The system should support:

```text
prediction
    ↓
decision
    ↓
intervention
    ↓
outcome
```

Example:

```text
T0: October 1
Risk score generated

T1: October 3
Retention intervention sent

T2: November 2
30-day outcome evaluated
```

---

# 12. Uplift Timestamp

For causal analysis:

```text
T0 = treatment assignment
```

Treatment must occur after assignment.

The outcome must occur after treatment.

Correct:

```text
Features
   ↓
T0
   ↓
Treatment / Control
   ↓
Outcome
```

Incorrect:

```text
Features
   ↓
Outcome-derived feature
   ↓
Treatment
```

That would create post-treatment leakage.

---

# 13. Production Scoring Frequency

The initial system will support batch scoring conceptually:

```text
daily scoring
```

Example:

```text
02:00
↓
refresh eligible customer states
↓
generate predictions
↓
calculate decisions
↓
publish retention recommendations
```

Real-time scoring can be introduced later if the business use case requires it.

---

# 14. Why Daily Batch Is the Initial Choice

A real-time architecture is not automatically better.

For retention, daily scoring is often sufficient when:

* customer behavior does not change every second;
* interventions are campaign-based;
* operational teams act in batches;
* data sources update periodically.

Real-time inference should be introduced only if latency materially affects the business decision.

---

# 15. Temporal Validation

The model must be tested on future-like data.

Example:

```text
TRAIN
Jan–Jun

VALIDATION
Jul–Aug

TEST
Sep

FUTURE HOLDOUT
Oct
```

The exact periods depend on the final dataset.

---

# 16. Prediction Record

Every production prediction should conceptually contain:

```text
prediction_id
customer_id
prediction_timestamp
model_version
feature_version
churn_probability_30d
churn_probability_90d
churn_probability_180d
survival_estimate
clv_estimate
uplift_estimate
decision
decision_timestamp
```

This makes predictions auditable.

---

# 17. Golden Rule

> **At T0, the model may know only what a real production system could have known at T0.**

Any experiment violating this rule is invalid regardless of its model score.
