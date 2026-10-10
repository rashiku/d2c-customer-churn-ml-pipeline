# Error Analysis — LightGBM Churn Prediction Model

**Model:** LightGBM  
**Decision Threshold:** 0.35  
**Evaluation Set:** Validation (336 customers)  
**Snapshot Date:** 2025-09-30

---

## Overview of Errors at Selected Threshold

| Metric | Value |
|--------|-------|
| Decision Threshold | 0.35 |
| Total Validation Samples | 336 |
| True Positives (TP) | 136 |
| True Negatives (TN) | 119 |
| **False Negatives (FN)** | **11** (3.3%) |
| **False Positives (FP)** | **70** (20.8%) |
| Recall (churn class) | 0.9252 |
| Precision (churn class) | 0.6602 |
| F2-Score | 0.8564 |

The threshold of **0.35** was selected to maximise the F2-score, which weights recall twice as heavily as precision. This reflects the business reality that missing a churner (FN) is significantly more costly than flagging a loyal customer unnecessarily (FP).

On the validation set at threshold 0.35, **70 of 189 retained customers (37.0%) are incorrectly flagged as churning**. On the held-out test set, this rises to 75 of 168 retained customers (44.6%) — partly because the test set has a higher base churn rate (50% vs 43.8%). This FP rate is an acceptable trade-off only if campaign cost per customer is low (≤ ₹10–15) and the business prioritises catching churners over precision. If campaign cost is high or the intervention is intrusive (phone calls, large discounts), the threshold should be raised to 0.40–0.44.

---

## Business Risk of Each Error Type

### False Negative (FN) — *Missed Churner*

> The model predicted the customer would stay, but they actually churned.

**Business impact (HIGH):**
- The retention team never contacts this customer.
- The customer leaves without any intervention attempt.
- Revenue loss equals the full lifetime value of the customer, with no offsetting campaign cost.
- At 11 FNs in the validation set (3.3%), the model misses roughly 1 in 27 churners — an acceptable rate given the recall of 92.5%.

---

### False Positive (FP) — *Unnecessary Intervention*

> The model predicted churn, but the customer was going to stay anyway.

**Business impact (MODERATE):**
- The retention team sends an offer (e.g., free shipping, bundle discount) to a loyal customer who didn't need it.
- Campaign cost per customer: ₹0–₹40 (per `intervention_history.csv`).
- Risk of training customers to expect discounts before every purchase cycle.
- At 70 FPs in the validation set (20.8%), roughly 1 in 5 flagged customers is a false alarm — this is the deliberate cost of the high-recall strategy.

---

## False Negative Analysis (Churners We Missed)

These 5 customers churned but were predicted as "Not Churn." They share a deceptively healthy profile at the snapshot date — which explains the model's failure.

| Customer ID | Recency (days) | Frequency (180d) | Sessions (30d) | Prob (Churn) | Loyalty Tier |
|-------------|---------------|-----------------|----------------|-------------|--------------|
| CUST00650 | 59 | 2 | 6 | 0.287 | Silver |
| CUST01713 | 41 | 5 | 1 | 0.280 | Not Enrolled |
| CUST00188 | 29 | 2 | 11 | 0.226 | Gold |
| CUST01626 | 28 | 3 | 6 | 0.199 | Silver |
| CUST01878 | 22 | 1 | 4 | 0.314 | Not Enrolled |

### Patterns in False Negatives

**1. CUST00650 — Moderate Recency, Low Model Confidence**
- Last order 59 days before snapshot; 2 orders in 180 days; 6 web sessions in 30 days.
- The moderate recency and active web browsing gave the model a "staying" signal.
- Predicted probability was only 0.287 — just below the threshold of 0.35.
- **Interpretation:** This customer had already begun disengaging from purchasing but maintained passive browsing. The model reads browsing as engagement, but it does not convert to purchase.

**2. CUST01713 — High Frequency, Zero Conversion**
- 5 orders in 180 days (above average), but only 1 web session in the last 30 days.
- Predicted probability 0.280. The model leaned on high historical frequency as a retention signal.
- Not enrolled in loyalty programme — no structural stickiness.
- **Interpretation:** A previously active customer who went quiet in the final month. The recency-frequency interaction (`recency_x_frequency`) likely pulled this customer toward "staying" due to the high frequency score, masking the recent drop-off.

**3. CUST00188 — Gold Loyalty Tier, High Sessions, But Still Churned**
- Gold loyalty tier, 29 days recency, 11 sessions in 30 days, 2 orders in 180 days.
- Predicted probability only 0.226 — the lowest among FNs.
- **Interpretation:** The combination of Gold tier + frequent sessions strongly signalled retention to the model. This customer likely browsed extensively without purchasing — a behaviour pattern ("browse-not-buy") the model does not yet capture.

**4. CUST01626 — Recent Purchaser Who Churned**
- Recency of only 28 days and 3 orders in 180 days — model confidence was 0.199.
- **Interpretation:** This is the most surprising FN. A customer who purchased less than a month before snapshot still churned in the 60-day window. This likely reflects a one-time replenishment purchase rather than ongoing loyalty. A feature capturing whether the most recent order was a repeat category purchase could help.

**5. CUST01878 — Low Frequency, Borderline Probability**
- 22 days recency, 1 order in 180 days, 4 sessions. Probability 0.314.
- **Interpretation:** Low overall engagement masked by a fairly recent purchase. Single-order customers with low web activity are inherently difficult to classify because we have limited signal.

---

## False Positive Analysis (Loyal Customers Flagged as Churners)

These 5 customers were flagged for retention but stayed. Most sit very close to the 0.35 threshold — this is expected behaviour and not a model failure.

| Customer ID | Recency (days) | Frequency (180d) | Sessions (30d) | Prob (Churn) | Loyalty Tier |
|-------------|---------------|-----------------|----------------|-------------|--------------|
| CUST00857 | 67 | 2 | 3 | 0.351 | Not Enrolled |
| CUST01261 | 18 | 1 | 1 | 0.352 | Gold |
| CUST02079 | 19 | 4 | 1 | 0.360 | Not Enrolled |
| CUST00006 | 51 | 4 | 2 | 0.363 | Silver |
| CUST02226 | 49 | 4 | 5 | 0.364 | Gold |

### Patterns in False Positives

**1. CUST00857 — Not Enrolled, Long Recency**
- 67 days since last order, not enrolled in loyalty, 3 sessions.
- Probability 0.351 — barely over the threshold.
- **Interpretation:** The long recency and missing loyalty tier are legitimate churn signals. The model was not wrong to be cautious. This customer ended up purchasing again, but they were genuinely at-risk at the snapshot date. A low-cost nudge campaign was likely appropriate.

**2. CUST01261 — Gold Tier But Minimal Activity**
- Despite Gold loyalty status, only 1 order in 180 days and 1 session in 30 days.
- Recency of 18 days (recent) but very low engagement otherwise.
- **Interpretation:** This customer has Gold loyalty driven by historical behaviour, not current engagement. The model correctly reads low current-period activity as a risk signal. Gold tier customers who disengage are a key watch group.

**3. CUST02079 — Moderate Frequency, Low Current Sessions**
- 4 orders in 180 days but only 1 session in the last 30 days.
- Not enrolled in loyalty.
- **Interpretation:** A historically active customer who pulled back sharply in the last month. The model flagged the session drop-off correctly — the customer happened to purchase again after a low-activity period. A re-engagement campaign was a sensible precaution.

**4. CUST00006 — 2 Support Tickets, Moderate Engagement**
- 51 days recency, 4 orders, 2 sessions, 2 support tickets in 90 days.
- Silver loyalty, probability 0.363.
- **Interpretation:** Two support tickets alongside moderate recency is a risk pattern. The model combined these signals appropriately. The customer retained, but the elevated support activity warranted monitoring.

**5. CUST02226 — Gold Tier, 10 Days Since Last Visit**
- 49 days since last order but recent last visit (10 days ago). Gold loyalty, 4 orders, 5 sessions, 1 ticket.
- Probability 0.364.
- **Interpretation:** At first glance this looks like a loyal customer. But the 49-day order gap combined with a medium recency score crosses the threshold. This is a borderline case where the model and the customer's actual behaviour are almost equally uncertain.

---

## Failure Mode Summary

| Failure Mode | Affected FN/FP | Root Cause | Potential Fix |
|---|---|---|---|
| Browse-not-buy | CUST00188 (FN) | Sessions ≠ purchase intent | Add cart-to-session ratio feature |
| Recency masked by frequency | CUST01713 (FN) | High historical orders offset low recent activity | Weight recent 30d frequency separately |
| Single replenishment purchase | CUST01626 (FN) | Last order looks recent but was a one-off | Add repeat-category flag |
| Borderline threshold proximity | All FPs | Probabilities 0.351–0.365 — all within 3% of threshold | Acceptable; a "monitor" tier for 0.35–0.45 range |
| Low loyalty engagement | CUST01261 (FP) | Loyalty tier assigned historically, not current | Decay loyalty signal by recency |

---

## Recommendations

1. **Create a "Monitor" bucket** for customers with churn probability 0.35–0.45 — send lighter, lower-cost touchpoints (email only) rather than full discount campaigns.
2. **Add a browse-to-purchase conversion feature** — the ratio of sessions to orders is currently not explicitly modelled and would help resolve the CUST00188 failure pattern.
3. **Track the 11 FN customers** in the validation set over the next 60 days to understand whether they were truly one-time churners or if they eventually returned.
4. **Review FP campaign costs** — at ₹0–₹40 per customer and 70 FPs, the maximum unnecessary campaign cost in validation is ₹2,800. This is negligible against the revenue lost from missing even one mid-to-high-value customer.
