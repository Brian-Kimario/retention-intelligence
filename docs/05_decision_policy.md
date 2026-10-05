# Retention Intelligence Platform

## Decision Policy & Intervention Strategy

**Document status:** Foundational
**Version:** 1.0

---

# 1. Purpose

The decision policy converts model outputs into economically constrained retention recommendations.

The policy exists because:

> **Prediction is not a decision.**

The model estimates what may happen.

The policy determines what the business should do.

---

# 2. Inputs to the Decision Engine

For each eligible customer:

```text
customer_id

churn_probability_30d
churn_probability_90d
churn_probability_180d

expected_time_to_churn

expected_CLV

CLV_at_risk

estimated_uplift

intervention_cost

customer_eligibility

campaign_capacity

business_constraints
```

---

# 3. Decision Pipeline

```text
Customer
   ↓
Risk Assessment
   ↓
Timing Assessment
   ↓
Value Assessment
   ↓
Treatment Response Assessment
   ↓
Economic Assessment
   ↓
Eligibility Constraints
   ↓
Action Selection
```

---

# 4. Step 1 — Risk Gate

A customer must have sufficiently elevated risk to enter the retention decision process.

Risk is not necessarily defined by a universal probability threshold.

The threshold will be determined using:

- precision/recall tradeoffs;
- operational capacity;
- expected value;
- intervention costs.

---

# 5. Step 2 — Timing Gate

The system evaluates whether intervention is:

```text
too early
appropriate
urgent
too late
```

The decision must consider the estimated survival trajectory.

Example:

```text
>90 days expected survival:
Monitor

30–90 days:
Early intervention

7–30 days:
Priority intervention

<7 days:
Urgent intervention
```

These are initial policy categories, not validated business thresholds.

They must be optimized experimentally.

---

# 6. Step 3 — Value Gate

Calculate:

```text
CLV at Risk
```

Customers with negligible economic value may not justify expensive interventions.

This prevents:

```text
highest churn risk
=
highest priority
```

from becoming the default strategy.

---

# 7. Step 4 — Treatment Response Gate

The system estimates:

```text
incremental treatment effect
```

Potential categories:

```text
HIGH POSITIVE UPLIFT
→ treatment likely beneficial

LOW/NEUTRAL UPLIFT
→ treatment likely inefficient

NEGATIVE UPLIFT
→ avoid intervention
```

---

# 8. Step 5 — Economic Gate

Estimate:

```text
Expected Incremental Value
=
Expected Incremental Retained Value
-
Intervention Cost
```

If:

```text
Expected Incremental Value <= 0
```

the default recommendation is:

```text
NO PAID INTERVENTION
```

unless another strategic constraint overrides the policy.

---

# 9. Intervention Classes

The initial policy framework supports:

```text
I0 — No intervention

I1 — Educational / informational message

I2 — Personalized product recommendation

I3 — Customer success / support outreach

I4 — Retention incentive

I5 — High-value service recovery
```

The public datasets do not necessarily contain all of these interventions.

Therefore these categories represent the target production architecture, not fabricated historical observations.

---

# 10. Intervention Hierarchy

The system should prefer the **least expensive effective intervention**.

Conceptually:

```text
No action
   ↓
Low-cost communication
   ↓
Personalized engagement
   ↓
Support intervention
   ↓
Financial incentive
```

A discount should not be the default solution to every churn risk.

---

# 11. Customer Segmentation

The decision engine should create segments similar to:

## Segment A — High Value / High Risk / High Uplift

```text
ACTION:
Priority intervention
```

---

## Segment B — High Value / High Risk / Low Uplift

```text
ACTION:
Low-cost engagement or monitor
```

Do not waste expensive incentives.

---

## Segment C — Low Value / High Risk / High Uplift

```text
ACTION:
Low-cost automated intervention
```

---

## Segment D — Low Value / High Risk / Negative Uplift

```text
ACTION:
No expensive intervention
```

---

## Segment E — Low Risk

```text
ACTION:
Normal lifecycle engagement
```

---

# 12. Budget Allocation

If retention budget is limited:

```text
Rank eligible customers by:

Expected Incremental Value
/
Intervention Cost
```

subject to:

```text
budget
capacity
eligibility
frequency constraints
```

The project will simulate constrained campaign allocation.

---

# 13. Example

Customer:

```text
CLV = $1,500

Churn probability = 70%

Estimated uplift = 20%

Intervention cost = $50
```

Simplified expected incremental value:

```text
$1,500 × 20%
-
$50

= $250
```

Recommendation:

```text
TARGET
```

---

# 14. Negative Example

Customer:

```text
CLV = $150

Estimated uplift = 5%

Intervention cost = $30
```

Simplified:

```text
$150 × 5%
-
$30

= -$22.50
```

Recommendation:

```text
DO NOT USE EXPENSIVE RETENTION INCENTIVE
```

The customer can still receive normal lifecycle communication.

---

# 15. Timing Policy

The system must eventually optimize intervention timing rather than use a fixed rule.

The decision should consider:

```text
expected churn time
current risk
risk trajectory
treatment response
intervention cost
customer contact history
```

Potential policy:

```text
Early warning
     ↓
Monitor trajectory

Risk acceleration
     ↓
Low-cost intervention

High-value intervention window
     ↓
Priority action

Near-churn
     ↓
Escalation

Post-churn
     ↓
Recovery / win-back strategy
```

---

# 16. Frequency Control

A customer should not receive repeated interventions simply because the model repeatedly predicts high risk.

The decision engine must eventually enforce:

```text
maximum interventions / period
cooldown period
recent intervention status
recent response
```

This prevents customer fatigue and unnecessary spending.

---

# 17. Policy Safety

The system must support:

```text
HUMAN REVIEW
```

for decisions involving:

- unusually high customer value;
- unusually high intervention cost;
- uncertain model predictions;
- low-confidence treatment effects;
- policy exceptions.

Automation must not remove accountability.

---

# 18. Policy Evaluation

The policy should be evaluated against:

```text
No intervention baseline
Random targeting
Risk-only targeting
Value-only targeting
Risk × value targeting
Uplift targeting
Economic optimization
```

The objective is to demonstrate that adding decision intelligence produces better targeting than simpler policies.

---

# 19. Business KPIs

Track:

```text
Retention rate
Incremental retention
CLV retained
CLV at risk
Intervention cost
Cost per retained customer
Incremental revenue
Incremental profit
Retention ROI
Budget utilization
Customer contact rate
```

---

# 20. Core Policy Principle

> **Do not spend money merely because a model predicts churn. Spend money when the expected incremental economic value of intervention justifies the cost and the customer is eligible for action.**
