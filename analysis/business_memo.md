# Business Memo

**To:** Retention & CRM Leadership  
**From:** Analytics — Customer Intelligence  
**Date:** 2025-10-01  
**Re:** What We Must Understand Before Launching a Retention Campaign  
**Classification:** Internal — Pre-Campaign Review

---

## Situation

Our snapshot as of 30 September 2025 shows a **47% churn rate** across 2,400 customers — meaning nearly one in two customers made no purchase in the following 60 days. Before the retention team commits budget and creative to a campaign, we need to share five findings from the data that should directly shape what we build.

This memo is not a modelling report. It is a flag-raising exercise. Each point below identifies a pattern in the data that, if ignored, will reduce campaign effectiveness or misallocate spend.

---

## 1. Recency Is the Dominant Signal — and We're Already Late for Half Our Customers

Purchase recency (days since last order) is the single strongest predictor of churn in this dataset, with a Spearman correlation of **|r| = 0.61** — far above any other variable we measured.

The churn rates by recency bucket are stark:

| Days since last order | Churn Rate | Customers |
|---|---|---|
| 0 – 30 days | **12%** | 671 |
| 31 – 60 days | **30%** | 441 |
| 61 – 90 days | **46%** | 361 |
| 91 – 180 days | **79%** | 597 |
| 180+ days | **91%** | 302 |

**597 customers have not ordered in 91–180 days, and 302 have not ordered in over six months.** For these two groups, standard re-engagement offers may not be sufficient — they need winback-level incentives or should be triaged separately. Campaigns designed for the "at-risk but still active" segment will not resonate with customers who have already gone quiet.

**Action required:** Segment the campaign by recency tier before anything else. Customers in the 0–60d band need nudges; customers in the 91d+ band need a fundamentally different offer.

---

## 2. Web Disengagement Precedes Purchase Dropout — We Have an Early-Warning System We're Not Using

In the 30 days before the snapshot, churned customers showed dramatically lower digital activity than retained ones:

| Metric (30-day window) | Retained | Churned | Ratio |
|---|---|---|---|
| Sessions | 6.7 | 4.0 | 0.60× |
| Product views | 28.4 | 17.0 | 0.60× |
| Cart adds | 1.9 | 1.1 | 0.58× |
| Last visit (days ago) | 9.8 days | 26.6 days | 2.7× |

Churned customers visited the site an average of **26.6 days before the snapshot** — nearly four weeks before it. Retained customers visited **9.8 days** before. With last-visit-days-ago carrying a Spearman |r| of **0.53** with churn, this metric alone is a meaningful real-time signal.

**What this means operationally:** We do not need to wait for a missed purchase to know a customer is at risk. If a customer's session count drops to near zero or their last visit exceeds 14 days, that is a trigger for automated re-engagement — a push notification, a personalised email, or a browse-abandon recovery flow. We currently have no evidence that such triggers are firing. This is a gap we can close without any model.

---

## 3. The CRM Priority List Is a Decent Start but Misses Over a Quarter of At-Risk Customers

The CRM team's manually assigned `manual_priority_bucket` performs surprisingly well as a rough churn predictor:

- **Precision:** 74.7% — of customers flagged as "high priority," 3 in 4 actually churned.
- **Recall:** 77.1% — of customers who actually churned, the list catches about 3 in 4.

However, **258 customers who churned were NOT flagged as high priority.** These are customers the campaign will miss entirely if it targets only the "high" bucket. They are not edge cases — they represent 23% of all churned customers.

There is also a cost dimension: the campaign's precision means that roughly 1 in 4 customers flagged "high" is actually retained. Sending aggressive retention offers to customers who would have bought anyway erodes margin unnecessarily.

**Action required:** The manual list should be treated as a starting point, not the final targeting list. Before launch, run a model-based scoring pass to recover the 258 missed churners and de-prioritise likely false positives in the "high" bucket.

---

## 4. More Than Half Our Customers Have Never Been Enrolled in the Loyalty Programme

**1,386 customers — 57.8% of our base — have no loyalty tier.** This is not a data gap. These customers were simply never enrolled. Unenrolled customers show a churn rate of 48.3%, compared to 45.1% for enrolled customers.

More striking is the spending gap: Platinum-tier customers spend significantly more on average than unenrolled customers, and enrolled customers at every tier show higher total spend.

This matters for campaign design in two ways. First, a retention campaign that offers a discount or free shipping to an unenrolled customer is a short-term fix. The same budget spent enrolling that customer in the loyalty programme may generate more durable behaviour change. Second, the campaign creative and messaging must account for the fact that more than half the audience has no loyalty relationship with the brand — they cannot be addressed as "loyalty members."

**Action required:** Include loyalty enrolment as a campaign outcome, not just re-purchase. For unenrolled customers, the campaign CTA should be "join + buy" rather than "buy."

---

## 5. There Are Two Discount Stories in This Data, and We Need to Know Which One We're Telling

Customers who buy at consistently high average discounts (>50% average discount) show higher churn than customers who buy at moderate or low discounts, even after controlling for recency. The difference in average discount between churned (28.5%) and retained (27.7%) customers is small but statistically significant (p = 0.049).

This raises a critical campaign design question: **if we offer discounts to retain churned customers, are we retaining them or renting them?** A customer acquired on a 50% discount who is retained by a 40% discount is a customer whose full-price purchase probability may never materialise.

The data cannot answer this question on its own — we need to look at cohort LTV post-intervention. But it tells us we should not assume discount campaigns work equally well across all segments.

**Action required:** Before setting discount depth for retention offers, filter the target list into "high-discount habitual buyers" and "infrequent-discount buyers." Test different offer types for each. For high-discount habitual buyers, consider non-monetary offers (loyalty tier upgrade, early access, free personalisation) instead of further price reduction.

---

## One Additional Data Risk the Team Should Know About

Our order dataset contains **1,872 post-snapshot transactions** (18.7% of all order rows) that were used to build the churn labels. Any analyst or data scientist pulling order data must filter to `order_date ≤ 2025-09-30` before computing any customer-level aggregates. Using post-snapshot order data as a model input would artificially inflate model performance and make the system useless in production. This filter has been applied in all analyses underpinning this memo.

---

## Summary of Recommended Pre-Campaign Actions

| # | Action | Owner | Priority |
|---|---|---|---|
| 1 | Segment target list by recency tier (0-30d / 31-60d / 61-90d / 91d+) | CRM | 🔴 Before launch |
| 2 | Activate web-signal triggers (last visit >14d, 0 sessions in 7d) | Product / CRM | 🔴 Before launch |
| 3 | Run model-based scoring to recover 258 missed churners from manual list | Analytics | 🟡 Before launch |
| 4 | Add loyalty enrolment as a campaign CTA for 1,386 unenrolled customers | Marketing | 🟡 Before launch |
| 5 | Separate high-discount habitual buyers and test non-monetary retention offers | Marketing | 🟡 A/B test |
| 6 | Confirm `order_date` filter is applied in all downstream data pulls | Data Engineering | 🔴 Immediate |

---

*This memo is based on exploratory analysis of the full 2,400-customer snapshot. Numbers cited are from executed notebook outputs and are reproducible via `eda_audit.ipynb`.*
