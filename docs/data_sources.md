# Data Sources

**Project:** Retention Intelligence Platform
**Document Version:** 1.0
**Last Updated:** 2026-10-06

---

## 1. Purpose

This document records the external datasets used by the Retention Intelligence Platform.

The objective is to maintain reproducible data lineage by recording:

- source organization;
- source location;
- dataset purpose;
- acquisition date;
- local storage location;
- file name;
- file format;
- file checksum;
- known limitations;
- intended analytical use.

Raw datasets are treated as immutable source artifacts.

---

# 2. Dataset Registry

| Dataset                  | Source        | Role                             | Status           |
| ------------------------ | ------------- | -------------------------------- | ---------------- |
| IBM Telco Customer Churn | IBM           | Primary churn modeling           | Acquired         |
| Criteo Uplift            | Criteo AI Lab | Uplift/treatment-effect modeling | Not yet acquired |

---

# 3. IBM Telco Customer Churn

## 3.1 Source

**Organization:** IBM

**Repository:**

https://github.com/IBM/telco-customer-churn-on-icp4d

**Dataset file:**

`data/Telco-Customer-Churn.csv`

**Raw download endpoint:**

https://raw.githubusercontent.com/IBM/telco-customer-churn-on-icp4d/master/data/Telco-Customer-Churn.csv

---

## 3.2 Dataset Role

This is the primary dataset for the initial customer churn modeling pipeline.

It will be used to develop and evaluate:

- customer churn propensity modeling;
- feature engineering;
- probability calibration;
- risk segmentation;
- model explainability;
- model serving;
- monitoring;
- data-drift detection.

---

## 3.3 Dataset Characteristics

The dataset represents customer-level telecommunications information and observed churn outcomes.

It is a **static customer snapshot**, not a complete longitudinal customer-event dataset.

Therefore it cannot, by itself, support claims such as:

- true 30-day future churn probability;
- true 90-day future churn probability;
- true 180-day future churn probability;
- longitudinal customer lifetime value;
- causal effect of a retention intervention.

Those capabilities require appropriate timestamped historical and intervention data.

---

## 3.4 Local Storage

Raw file:

```text
data/raw/telco/Telco-Customer-Churn.csv
```

The raw file must not be manually modified.

---

## 3.5 Acquisition Record

**Acquisition date:** 2026-10-06

**Acquisition method:** Direct download from IBM GitHub repository

**File format:** CSV

**File name:**

```text
Telco-Customer-Churn.csv
```

**SHA-256:**

```text
16320c9c1ec72448db59aa0a26a0b95401046bef5d02fd3aeb906448e3055e91
```

---

## 3.6 Data Lineage

Current lineage:

```text
IBM GitHub Repository
        ↓
Raw CSV Download
        ↓
data/raw/telco/
        ↓
Schema Validation
        ↓
Data Quality Validation
        ↓
Validated Dataset
        ↓
Feature Engineering
        ↓
Model Dataset
```

---

## 3.7 Known Limitations

The dataset is not a production telecom data warehouse.

Important limitations include:

1. It is primarily a customer-level snapshot.
2. It does not provide a complete timestamped customer event history.
3. It does not provide a genuine intervention history.
4. It does not provide randomized retention-treatment outcomes.
5. It therefore cannot independently establish causal retention strategies.
6. Public-data population characteristics may differ from a real company's customers.

These limitations must be reflected in model evaluation and project claims.

---

# 4. Criteo Uplift Dataset

## 4.1 Source

**Organization:** Criteo AI Lab

**Hugging Face dataset:**

https://huggingface.co/datasets/criteo/criteo-uplift

---

## 4.2 Dataset Role

The Criteo dataset will be used later for uplift/treatment-effect methodology.

Its purpose is to investigate:

> Which individuals are more likely to respond positively because they receive an intervention?

This is fundamentally different from churn prediction.

---

## 4.3 Status

**Status:** Not yet acquired

The dataset will not be downloaded until the initial Telco churn pipeline has been validated.

---

## 4.4 Separation Rule

The IBM Telco and Criteo datasets must not be merged at the customer-record level.

They represent different populations and different analytical problems.

Their outputs will eventually be connected conceptually through the decision-policy architecture:

```text
Churn Risk
     +
Treatment Uplift
     +
Customer Value
     +
Intervention Cost
     ↓
Retention Decision
```

---

# 5. Data Governance Rules

## Rule 1 — Raw data is immutable

Never edit:

```text
data/raw/
```

If cleaning is required, produce a new artifact.

---

## Rule 2 — Every external dataset must have provenance

Each dataset must record:

```text
source
acquisition date
file name
checksum
purpose
limitations
```

---

## Rule 3 — No undocumented transformations

Every transformation from raw → validated → processed must be reproducible through code.

---

## Rule 4 — Dataset versions must be distinguishable

If the source changes, the new artifact receives a new acquisition record and checksum.

---

## Rule 5 — Public data limitations must not be hidden

Model capabilities must never exceed what the source data can legitimately support.
