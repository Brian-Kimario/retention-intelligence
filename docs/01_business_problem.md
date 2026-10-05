# Retention Intelligence Platform

## Business Problem & Decision Framework

**Document status:** Foundational
**Version:** 1.0
**Purpose:** Define the business problem before any data science or engineering implementation begins.

---

## 1. Executive Problem Statement

The business operates a recurring-revenue customer relationship in which customer attrition reduces future revenue, increases acquisition pressure, and destroys potential Customer Lifetime Value (CLV).

The business currently faces a fundamental decision problem:

> **Which customers should receive retention intervention, what intervention should they receive, and when should the intervention occur in order to maximize expected incremental customer value while controlling intervention cost and operational capacity?**

A simple churn prediction system is insufficient.

Predicting that a customer is likely to churn does not establish:

- whether the customer is economically important;
- when churn is likely to occur;
- whether the customer can be influenced;
- whether an intervention will change the customer's behavior;
- which intervention is appropriate;
- whether intervention cost is justified;
- whether the retention strategy is actually generating incremental value.

Therefore, this project will build a **Retention Intelligence Platform** rather than a standalone churn classifier.

---

# 2. Business Objective

The primary business objective is:

> **Maximize expected retained customer value through risk-aware, value-aware and treatment-aware retention decisions.**

The platform will support five sequential business questions:

1. **Who is likely to churn?**
2. **When is churn likely to occur?**
3. **How much customer value is at risk?**
4. **Which customers are likely to respond to intervention?**
5. **Which intervention and timing produce the highest expected incremental economic value?**

---

# 3. Business Decisions Supported

The system is intended to support decisions at three levels.

## 3.1 Customer-level decisions

For an individual customer:

- Should the customer be contacted?
- How urgent is the intervention?
- What intervention is appropriate?
- How much intervention cost is justified?
- What is the expected value of intervening?

---

## 3.2 Campaign-level decisions

For a retention campaign:

- Which customers should be included?
- Which customers should be excluded?
- How should the available retention budget be allocated?
- What proportion of customers should receive each intervention?
- What expected incremental value should the campaign generate?

---

## 3.3 Board-level decisions

At strategic level:

- How much customer value is currently at risk?
- Which customer segments represent the largest exposure?
- What is the expected return from retention investment?
- How does retention budget affect expected retained value?
- Is retention performance improving or deteriorating?
- Is the deployed decision system still reliable?

---

# 4. Core Business Principle

The system must distinguish four concepts:

```text
CHURN RISK
    ≠
CUSTOMER VALUE
    ≠
TREATMENT RESPONSE
    ≠
BUSINESS ACTION
```

A high-risk customer is not automatically a good retention target.

For example:

```text
Customer A
High churn risk
Low CLV
Low treatment response
→ Do not spend heavily.

Customer B
Moderate churn risk
High CLV
High treatment response
→ Potentially high-value target.

Customer C
High churn risk
High CLV
Negative treatment effect
→ Avoid unnecessary intervention.
```

The platform therefore optimizes decisions rather than simply ranking customers by churn probability.

---

# 5. Conceptual Decision Chain

The final decision architecture is:

```text
Customer State
      ↓
Churn Propensity
      ↓
Time-to-Churn / Survival
      ↓
Customer Lifetime Value
      ↓
Treatment Uplift
      ↓
Intervention Economics
      ↓
Decision Policy
      ↓
Recommended Action
      ↓
Observed Outcome
      ↓
Monitoring & Evaluation
```

---

# 6. ML Components

The platform will contain four analytical components.

## 6.1 Churn Propensity Model

Estimates:

```text
P(churn within horizon | customer state)
```

Initial horizons:

- 30 days
- 90 days
- 180 days

Purpose:

> Identify customers whose probability of churn is materially elevated.

---

## 6.2 Survival Model

Estimates the probability that a customer remains active over time.

Outputs may include:

- survival probability;
- hazard;
- expected time-to-event;
- probability of churn within future intervals.

Purpose:

> Determine the temporal urgency of retention intervention.

---

## 6.3 Customer Lifetime Value Model

Estimates future economic value associated with a customer.

The project will distinguish:

```text
Revenue
Contribution / Profit
Expected Future Value
CLV at Risk
```

Purpose:

> Determine the economic importance of the customer rather than treating every customer equally.

---

## 6.4 Uplift / Treatment-Effect Model

Estimates the incremental effect of an intervention.

Conceptually:

```text
Uplift(x)
=
E[Y | treatment, X=x]
-
E[Y | control, X=x]
```

Purpose:

> Identify customers whose behavior is expected to change because of intervention.

---

# 7. Business Optimization Objective

The decision engine will estimate:

```text
Expected Incremental Value
=
Expected Incremental Benefit
-
Intervention Cost
```

A simplified conceptual formulation is:

```text
Expected Incremental Value

≈

CLV at Risk
×
Estimated Incremental Retention Effect
-
Intervention Cost
```

The production implementation must use carefully defined business assumptions and must not present this simplified expression as a universally valid causal equation.

---

# 8. Constraints

The decision system must account for:

- retention budget;
- campaign capacity;
- customer eligibility;
- intervention cost;
- intervention frequency;
- minimum expected economic return;
- customer experience constraints;
- regulatory/privacy requirements;
- model confidence;
- operational availability.

A mathematically high-scoring customer is not automatically eligible for action.

---

# 9. Success Criteria

The project will evaluate success at four levels.

## Model-level

- discrimination;
- calibration;
- temporal generalization;
- survival performance;
- CLV estimation quality;
- uplift performance.

## Decision-level

- expected incremental value;
- policy value;
- targeting efficiency;
- intervention cost efficiency.

## System-level

- API latency;
- availability;
- error rate;
- deployment reliability;
- monitoring coverage.

## Business-level

Ultimately:

```text
Incremental retained value
/
Retention investment
```

is more important than model accuracy alone.

---

# 10. What This Project Will NOT Claim

The project will not claim that:

- correlation proves causation;
- churn probability proves intervention effectiveness;
- drift automatically means model failure;
- a public dataset represents a real company's complete CRM;
- simulated economics are equivalent to production financial results;
- an uplift model trained on one experimental population automatically transfers to another business.

All such assumptions must be explicitly documented.

---

# 11. Final Product Definition

The finished platform should be capable of producing a decision record conceptually similar to:

```text
Customer ID: C12345

Churn Risk:
78%

Expected Churn Horizon:
18–32 days

Estimated CLV:
$1,240

CLV at Risk:
$967

Estimated Treatment Uplift:
+14%

Recommended Intervention:
Proactive service outreach

Recommended Timing:
Within 7 days

Estimated Intervention Cost:
$18

Expected Incremental Value:
Positive

Decision:
TARGET
```

The system must also explain which model outputs and business constraints produced the recommendation.

---

# 12. Guiding Principle

> **The objective is not to predict churn as accurately as possible. The objective is to make better retention decisions using reliable predictions, temporal reasoning, causal evidence, economic value and continuous monitoring.**

That principle governs every subsequent design decision in this project.
