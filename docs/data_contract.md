# Retention Intelligence Platform

## Data Contract

**Document status:** Foundational
**Version:** 1.0
**Owner:** Data Science / ML Engineering
**Purpose:** Define the structure, meaning, quality requirements, temporal rules and lineage requirements for every dataset entering the ML pipeline.

---

# 1. Contract Objective

The data contract establishes a formal agreement between:

```text
DATA SOURCE
    ↓
INGESTION
    ↓
VALIDATION
    ↓
FEATURE ENGINEERING
    ↓
MODEL
```

The objective is to ensure that downstream ML components receive data that is:

- structurally valid;
- semantically meaningful;
- temporally valid;
- reproducible;
- traceable;
- safe for model consumption.

A model must never silently adapt to unexpected data.

---

# 2. Dataset Layers

The project uses four primary data layers.

```text
RAW
↓
VALIDATED
↓
PROCESSED
↓
MODEL-READY
```

## Raw

Exactly as downloaded from the source.

No modification.

## Validated

Passed schema and quality checks.

## Processed

Cleaned and transformed analytical data.

## Model-ready

Final feature matrix and target definitions used for training/inference.

---

# 3. Primary Customer Dataset Contract

Initial primary source:

```text
IBM Telco Customer Churn
```

The dataset will be treated as an initial **customer-state dataset**, not as a complete event-stream architecture.

---

# 4. Required Customer-Level Fields

| Field             | Type               |         Required | Meaning                        |
| ----------------- | ------------------ | ---------------: | ------------------------------ |
| customer_id       | string             |              Yes | Unique customer identifier     |
| gender            | categorical        |               No | Customer demographic attribute |
| senior_citizen    | integer/binary     |               No | Senior-citizen indicator       |
| partner           | categorical        |               No | Partner status                 |
| dependents        | categorical        |               No | Dependent status               |
| tenure            | integer            |              Yes | Customer tenure                |
| phone_service     | categorical        |               No | Phone-service status           |
| multiple_lines    | categorical        |               No | Multiple-line status           |
| internet_service  | categorical        |               No | Internet-service category      |
| online_security   | categorical        |               No | Security-service status        |
| online_backup     | categorical        |               No | Backup-service status          |
| device_protection | categorical        |               No | Device-protection status       |
| tech_support      | categorical        |               No | Technical-support status       |
| streaming_tv      | categorical        |               No | Streaming-TV status            |
| streaming_movies  | categorical        |               No | Streaming-movie status         |
| contract          | categorical        |              Yes | Contract type                  |
| paperless_billing | categorical        |               No | Billing preference             |
| payment_method    | categorical        |               No | Payment method                 |
| monthly_charges   | numeric            |              Yes | Monthly charge                 |
| total_charges     | numeric            |            Yes\* | Accumulated charge             |
| churn             | binary/categorical | Yes for training | Observed churn outcome         |

`*` Subject to source-specific missingness and validation.

---

# 5. Data-Type Rules

The ingestion pipeline must explicitly convert source types.

For example:

```text
TotalCharges
```

must not remain an arbitrary string merely because the source CSV stores it as text.

The validated schema must contain:

```text
monthly_charges → numeric
total_charges   → numeric
tenure          → integer
```

Categorical variables must be explicitly represented as categorical/string values before feature encoding.

---

# 6. Identifier Rules

`customer_id` must:

- be unique at the customer level;
- never be used as a predictive feature;
- remain stable throughout processing;
- be preserved for traceability;
- not contain personally identifiable information.

The identifier exists for:

```text
tracking
joins
auditability
prediction records
```

not prediction.

---

# 7. Target Rules

The raw `churn` field may be used to construct training targets.

It must not be used as a feature.

Derived targets must be stored separately from predictors.

Example:

```text
features/
    X_train

targets/
    y_30d
    y_90d
```

The exact applicability of multi-horizon targets depends on whether the source provides sufficient temporal information.

**Critical limitation:** the static IBM Telco dataset does not provide the longitudinal event timestamps required to honestly construct true 30/90/180-day forward-looking churn labels.

Therefore those horizons must NOT be fabricated from the static `Churn` column.

---

# 8. Missing Data Rules

Missing values must be classified as:

```text
TRUE MISSING
NOT APPLICABLE
UNKNOWN
INVALID
```

These categories must not automatically be treated as equivalent.

Example:

```text
No phone service
```

is not necessarily missing data.

It may mean:

```text
feature not applicable
```

The preprocessing layer must preserve this semantic distinction where possible.

---

# 9. Numerical Validation

The following constraints apply.

```text
tenure >= 0
monthly_charges >= 0
total_charges >= 0
```

Impossible values must fail validation rather than being silently clipped.

---

# 10. Categorical Validation

Unexpected categories must be detected.

Example:

```text
contract ∈ {
    Month-to-month,
    One year,
    Two year
}
```

An unexpected category must generate a validation warning/error depending on pipeline stage.

---

# 11. Duplicate Rules

Customer-level duplicates must be detected.

The ingestion pipeline must report:

```text
total rows
unique customers
duplicate customer IDs
```

No duplicates may be silently removed without recording the operation.

---

# 12. Schema Validation

Every ingestion run must validate:

```text
column existence
column names
data types
nullability
allowed categories
numeric ranges
identifier uniqueness
```

Schema failure should prevent downstream model execution.

---

# 13. Temporal Contract

Where timestamps exist, each record must define:

```text
event_time
ingestion_time
available_time
```

The model may only use information available at the prediction timestamp.

For datasets without timestamps, the project must explicitly classify them as:

```text
STATIC SNAPSHOT DATA
```

rather than pretending they represent longitudinal production history.

---

# 14. Dataset Versioning

Each acquisition must record:

```text
source
source URL
acquisition timestamp
file name
file hash
row count
column count
schema version
```

Example metadata:

```yaml
dataset:
  name: ibm_telco_customer_churn
  source: public
  acquisition_date: YYYY-MM-DD
  version: 1.0
  checksum: SHA256
```

---

# 15. Data Quality Gates

The pipeline must fail if critical conditions are violated.

Examples:

```text
customer_id missing
required column missing
invalid numerical value
unexpected target value
duplicate primary identifiers
schema mismatch
```

Warnings may be allowed for:

```text
non-critical missingness
new categorical values
distribution changes
```

but must be logged.

---

# 16. Feature Contract

Every engineered feature must document:

```text
feature_name
definition
source_columns
transformation
observation window
allowed data timestamp
missing-value treatment
business interpretation
```

Example:

```text
feature:
monthly_charge_to_tenure_ratio

definition:
monthly_charges / max(tenure, 1)

source:
monthly_charges
tenure
```

---

# 17. Data Lineage

Every model artifact must be traceable to:

```text
raw dataset
↓
validation version
↓
processing version
↓
feature version
↓
training dataset
↓
model version
```

---

# 18. Contract Enforcement

The contract will eventually be implemented through automated validation tests.

Potential tooling:

```text
Python
Pandas
Pandera
Pytest
Great Expectations
```

The initial implementation should avoid unnecessary tooling complexity.

Start with:

```text
Pandas + Pandera + Pytest
```

and introduce Great Expectations only if the project actually benefits from it.

---

# 19. Contract Principle

> **Unexpected data must produce an observable failure or warning, never silent corruption.**
