# Retention Intelligence Platform

## Dataset Selection & Evidence Strategy

**Document status:** Foundational
**Version:** 1.0

---

# 1. Purpose

This document defines the datasets used by the project and, more importantly, what each dataset can and cannot legitimately support.

The project will avoid constructing false claims by forcing a strict separation between:

```text
Predictive evidence
```

and:

```text
Causal / intervention evidence
```

---

# 2. Dataset Requirements

The ideal production dataset would contain:

```text
customer identity
customer history
timestamps
product usage
transactions
revenue
support interactions
subscription state
churn events
intervention history
intervention assignment
intervention outcome
```

A single public dataset rarely contains all of these components at sufficient quality.

Therefore a multi-source strategy is required.

---

# 3. Dataset A — IBM Telco Customer Churn

## Role

Primary customer-retention modeling dataset.

It will support:

```text
customer-state modeling
churn propensity
feature engineering
risk segmentation
baseline CLV experimentation
survival methodology where the available event structure permits it
```

The dataset contains customer demographic, service, contract and billing variables together with churn information.

---

# 4. Strengths

Useful for:

- demonstrating customer-level risk prediction;
- categorical/numerical feature engineering;
- customer segmentation;
- probability calibration;
- business-oriented churn analysis;
- initial deployment testing.

It is compact enough to make the first complete production architecture manageable.

---

# 5. Limitations

This dataset does NOT provide a complete production customer event history.

It therefore cannot honestly support:

```text
high-frequency behavioral trajectories
```

in the same way a real CRM event stream would.

It also does not provide randomized retention treatment assignment.

Therefore it cannot by itself establish:

```text
"This intervention causes this customer to stay."
```

---

# 6. Dataset B — Criteo Uplift Prediction Dataset

## Role

Causal / uplift modeling dataset.

Criteo provides randomized treatment/control information and outcome variables specifically for uplift/incrementality research.

The unbiased release contains approximately 14 million observations.

---

# 7. Strengths

Supports:

```text
treatment/control modeling
uplift prediction
heterogeneous treatment effects
policy evaluation
incrementality analysis
```

This makes it appropriate for learning the methodology required to answer:

> Which individuals are more responsive to an intervention?

---

# 8. Limitations

Criteo is not a telecom/customer-subscription CRM dataset.

Its features are anonymized and its experimental context differs from the retention domain.

Therefore:

```text
DO NOT JOIN WITH TELCO DATA
```

and:

```text
DO NOT CLAIM
the uplift model directly predicts telecom retention intervention outcomes.
```

Instead, it demonstrates the causal modeling methodology that the retention platform would require when real intervention data becomes available.

---

# 9. Dataset C — Real Production-Like Data

A future production implementation should replace or augment the public datasets with internal business data.

Required sources would include:

```text
CRM
Subscription system
Billing
Transactions
Product analytics
Support
Campaign platform
Intervention logs
Experiment assignment
Retention outcomes
```

The architecture is intentionally designed so these sources can replace the public datasets without changing the conceptual decision pipeline.

---

# 10. Dataset Strategy

The project therefore follows:

```text
                PUBLIC EVIDENCE

      ┌─────────────────────────────┐
      │                             │
      ↓                             ↓
Customer Churn Data          Randomized Treatment
      │                             │
      ↓                             ↓
Risk / Survival / CLV       Uplift / Causal Modeling
      │                             │
      └──────────────┬──────────────┘
                     ↓
             Decision Architecture
                     ↓
              Production Design
```

This is a methodological architecture, not a customer-level data join.

---

# 11. Dataset Selection Decision

## Primary dataset

**IBM Telco Customer Churn**

Use for:

```text
Phase 1:
customer risk and value modeling
```

---

## Causal dataset

**Criteo Uplift Prediction Dataset**

Use for:

```text
Phase 2:
treatment-effect and uplift modeling
```

---

## Production target

A future business deployment should use:

```text
longitudinal customer event data
+
randomized intervention data
```

from the actual business.

---

# 12. Why Not Use a Generic Kaggle Churn Dataset?

Generic churn datasets are often optimized for classroom classification rather than production decision-making.

Typical weaknesses include:

- one-row-per-customer structure;
- limited timestamps;
- weak behavioral history;
- no treatment assignment;
- no intervention outcomes;
- limited economic information;
- no reliable causal evidence.

Such datasets can still be useful for benchmarking, but they should not be presented as complete production retention data.

---

# 13. Dataset Acceptance Criteria

Before adopting any additional dataset, evaluate:

```text
Temporal coverage
Customer granularity
Event history
Target quality
Missingness
Data leakage risk
Economic variables
Intervention information
Treatment assignment
Outcome quality
Population relevance
Licensing
Reproducibility
```

---

# 14. Dataset Versioning

Every dataset used in the project must record:

```text
dataset name
source
URL
download/acquisition date
version
file checksum where applicable
row count
column count
schema
license
known limitations
```

---

# 15. Data Evidence Classification

Each variable should eventually be classified as:

```text
OBSERVED
DERIVED
ASSUMED
SIMULATED
```

Example:

```text
monthly_charges → OBSERVED

30d_usage_change → DERIVED

intervention_cost → ASSUMED / BUSINESS INPUT

future retention uplift → MODELED
```

The project must never hide assumptions as observed facts.

---

# 16. Synthetic / Simulated Business Variables

Where the public dataset does not provide required business inputs, the project may introduce clearly labeled assumptions for decision-engine demonstrations.

Examples:

```text
intervention cost
campaign budget
customer contact capacity
hypothetical retention offers
```

These must be stored in configuration rather than embedded invisibly in model code.

Example:

```text
configs/
└── business_policy.yaml
```

---

# 17. Transferability Warning

Performance on public datasets does not prove production performance.

The project must explicitly distinguish:

```text
MODEL PERFORMANCE ON PUBLIC DATA
```

from:

```text
EXPECTED PRODUCTION PERFORMANCE
```

Production deployment requires validation on representative internal data.

---

# 18. Final Dataset Decision

The initial project will proceed with:

### Predictive layer

```text
IBM Telco Customer Churn
```

### Causal/uplift layer

```text
Criteo Uplift Prediction Dataset
```

### Future production architecture

```text
Real longitudinal customer data
+
Randomized retention experimentation
```

This is the most defensible approach given the availability of public data.

---

# 19. Final Principle

> **Use each dataset only for the question it can actually answer. Never manufacture a causal or business claim simply because the dataset makes the project easier.**
