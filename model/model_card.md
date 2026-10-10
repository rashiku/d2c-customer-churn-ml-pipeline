# Model Card — D2C Customer Churn Prediction

| Field | Value |
|-------|-------|
| **Model Name** | LightGBM Churn Classifier |
| **Version** | 1.0 |
| **Snapshot Date** | 2025-09-30 |
| **Prediction Window** | 60 days post-snapshot (2025-10-01 to 2025-11-29) |
| **Model File** | `model.pkl` |
| **Last Evaluated** | 2025-09-30 (test set held out) |

---

## 1. Intended Use

### Primary Use
Identify customers at risk of churning in the next 60 days so the retention team can deliver targeted interventions before the customer leaves.

### Intended Users
- **CRM / Retention Team** — to prioritise outreach and campaign budgets
- **Marketing Team** — to select appropriate intervention type per risk segment
- **Product / Analytics Team** — to monitor engagement drop-off patterns

### Intended Deployment Context
- Batch scoring: run at the end of each calendar month against the full active customer base
- Output feeds into the internal CRM tool to surface high-risk customers for manual review or automated campaign triggering

---

## 2. Model Approach

| Component | Choice | Justification |
|-----------|--------|---------------|
| Algorithm | LightGBM (Gradient Boosted Trees) | Handles mixed feature types natively, robust to outliers, strong performance on tabular data |
| Baseline | Logistic Regression (L2, C=0.1, balanced class weight) | Interpretable, fast, establishes minimum performance bar |
| Encoding | OrdinalEncoder + LightGBM `categorical_feature` | Allows the tree model to learn optimal categorical splits |
| Class weight | `balanced` | Adjusts for the 47% churn rate to avoid the model biasing toward the majority class |
| Decision threshold | **0.35** | Selected by maximising F2-score on the validation set — weights recall 2× over precision |

### Feature Engineering
Four derived features were added on top of the 25 base features in `rfm_modeling_snapshot.csv`:

| Feature | Formula | Purpose |
|---------|---------|---------|
| `engagement_rate` | cart_adds_30d / (product_views_30d + 1) | Purchase intent signal |
| `email_click_rate` | campaign_clicks_30d / (email_opens_30d + 1) | Campaign engagement quality |
| `spend_per_order` | monetary_180d / (frequency_180d + 1) | Average basket size proxy |
| `recency_x_frequency` | log1p(recency_days) × (frequency_180d + 1) | Combined RFM interaction |

**Total features used: 29**

---

## 3. Data

### Training Data
| Split | Samples | Churn Rate |
|-------|---------|-----------|
| Train | 1,728 | 47.0% |
| Validation | 336 | 43.8% |
| Test | 336 | 50.0% |

### Data Sources (all pre-snapshot)
- `rfm_modeling_snapshot.csv` — RFM + support + web activity aggregations as of 2025-09-30
- Feature window: 180-day order history, 90-day support history, 30-day web activity

### Missing Values Handled
| Column | Treatment |
|--------|-----------|
| `loyalty_tier` (~1,386 nulls) | Filled with `"Not_Enrolled"` — absence of loyalty tier is a meaningful churn signal |

### Leakage Prevention
- Only data on or before `2025-09-30` was used as model input.
- Post-snapshot rows in `orders.csv` (dates after 2025-09-30) were excluded from all feature computation.
- The target column `churn_next_60d` was never used as a feature.
- The pre-assigned `split` column was used as-is to prevent train/test contamination.

---

## 4. Performance

### Validation Set (n=336)

> **Note on threshold:** LightGBM metrics below are at the **tuned threshold of 0.35** (F2-optimised). Baseline LR metrics are at the **default threshold of 0.50**. ROC-AUC and PR-AUC are threshold-independent and are the fairest basis for direct comparison — on both, LR marginally outperforms LightGBM (ROC-AUC: 0.8787 vs 0.8757). LightGBM's advantage is in recall after threshold tuning.

| Metric | LightGBM (threshold=0.35) | Baseline LR (threshold=0.50) |
|--------|--------------------------|------------------------------|
| Accuracy | 0.7589 | 0.7976 |
| Precision | 0.6602 | 0.7616 |
| Recall | **0.9252** | 0.7823 |
| F1-Score | 0.7705 | 0.7718 |
| **F2-Score** | **0.8564** | — |
| ROC-AUC | 0.8757 | **0.8787** |
| PR-AUC | 0.8540 | **0.8654** |

### Test Set (n=336, threshold=0.35) — Evaluated Once

| Metric | Value |
|--------|-------|
| Accuracy | 0.7500 |
| Precision | 0.6795 |
| Recall | **0.9464** |
| F1-Score | 0.7910 |
| **F2-Score** | **0.8775** |
| ROC-AUC | 0.8610 |
| PR-AUC | 0.8340 |

**Confusion Matrix (Test Set):**
```
                 Predicted: Not Churn   Predicted: Churn
Actual: Not Churn       93                  75
Actual: Churn            9                 159
```

### Threshold Rationale
The default threshold of 0.50 maximises accuracy but penalises recall. In a churn-prevention context, a **False Negative** (missing a churner) costs the business the full CLV of that customer with no intervention attempt. A **False Positive** (unnecessary campaign) costs ₹0–₹40 per customer. The asymmetry justifies lowering the threshold to **0.35**, which achieves **94.6% recall on the test set** while keeping precision above 67%.

At threshold 0.35, 75 of 168 retained test customers (44.6%) are incorrectly flagged. This is an acceptable trade-off only if campaign cost per customer is low (≤ ₹10–15) and the business prioritises catching churners over precision. If campaign cost is high or the intervention is intrusive (phone calls, large discounts), the threshold should be raised to 0.40–0.44.

### Top Predictive Features (by mean |SHAP|)

| Rank | Feature | Mean |SHAP| | Direction |
|------|---------|------------|-----------|
| 1 | `recency_days` | 0.363 | High recency → high churn risk |
| 2 | `last_visit_days_ago` | 0.255 | Long absence → high churn risk |
| 3 | `monetary_180d` | 0.171 | Low spend → higher churn risk |
| 4 | `recency_x_frequency` | 0.092 | Combined RFM interaction |
| 5 | `frequency_180d` | 0.075 | Low frequency → higher churn risk |
| 6 | `product_views_30d` | 0.068 | Low views → higher churn risk |
| 7 | `spend_per_order` | 0.036 | Low basket size → slightly higher risk |
| 8 | `negative_ticket_rate_90d` | 0.032 | Negative support → higher risk |
| 9 | `return_rate_180d` | 0.028 | High returns → higher risk |
| 10 | `days_since_signup` | 0.028 | Newer customers slightly more at-risk |

---

## 5. Limitations

1. **Single snapshot — no temporal validation.** The model is trained on one snapshot date (2025-09-30). It has not been validated against historical snapshot dates. Its performance may degrade if trained customer behaviour patterns shift significantly (e.g. sale season, new product launch).

2. **Behavioural signals are 30-day web metrics only.** The web activity features (`sessions_30d`, `product_views_30d`, etc.) cover only the 30 days before the snapshot. Customers who become inactive exactly after the snapshot period are not captured.

3. **No causal inference.** The model identifies correlation between features and churn — it does not determine *why* a customer is churning. A customer flagged as high-risk due to long recency may have churned because of a service failure, a competitor offer, or natural lifecycle completion. The model cannot distinguish these.

4. **Loyalty tier imputation may hide nuance.** ~57.8% of customers have `loyalty_tier = Not_Enrolled`. Treating all non-enrolled customers as a homogeneous group may obscure differences between customers who were never offered enrolment vs. those who opted out.

5. **Baseline (LR) has higher ROC-AUC.** The Logistic Regression baseline achieves ROC-AUC 0.879 vs. LightGBM's 0.876 on validation. The LightGBM model is selected for its superior recall and F2-score under the tuned threshold — but this illustrates that the LightGBM advantage is primarily in threshold behaviour, not raw discrimination.

6. **Class balance differs across splits.** The test set has a 50% churn rate vs. 43.8% on validation — this means test-set metrics are slightly easier to achieve on recall but harder on precision. Performance on a future live cohort may sit between the two.

---

## 6. Ethical Risks

### Discount Dependency Risk
Customers repeatedly flagged as "at-risk" and offered discounts may learn to withhold purchases until a discount arrives. The model output should not drive automatic discount triggers without a hold-out control group.

### Disparate Treatment Risk
If a protected characteristic (e.g. age group, city tier) correlates with churn — which it may, given `age_group` and `city_tier` are in the feature set — the model may systematically deprioritise retention efforts for certain demographic groups. For example, if `Tier 3` city customers have higher base churn rates, they may receive more frequent campaigns or, conversely, may be under-served if campaign budgets are capped. The retention team should audit campaign reach by demographic segment before large-scale deployment.

### Automated Refusal Risk
The model output must not be used to automatically exclude customers from promotions or downgrade their service tier. A high churn-probability score is a retention trigger, not a disenrollment signal.

### Privacy
The model uses age group, city tier, product category preference, and loyalty tier. These should be treated as sensitive attributes and not shared outside the retention workflow.

---

## 7. Monitoring Needs

### What to Track After Deployment

| Signal | Frequency | Alert Threshold |
|--------|-----------|----------------|
| Mean predicted churn probability | Monthly | ±5% change from baseline |
| Recall on labelled outcomes (60-day lag) | Monthly | Drop below 0.85 |
| Feature distribution shift (`recency_days`, `sessions_30d`) | Monthly | KS-statistic > 0.10 |
| False negative rate | Monthly | Exceeds 8% |
| Campaign conversion rate (intervened vs. predicted churners who stayed) | Monthly | Drops below 20% |
| Prediction volume by segment | Monthly | Unexpected spike in any single segment |

### Retraining Triggers
- Recall drops below 0.85 on a labelled cohort.
- A major product or pricing change that alters purchase behaviour patterns.
- Feature distribution shift detected for two consecutive months.
- Quarterly retraining is recommended as a baseline regardless of drift signals.

---

## 8. When the Model Should Not Be Used

- **Do not use to decide whether to terminate customer accounts or remove benefits.**
- **Do not use as the sole input for high-value customer intervention decisions** — always combine with human review for customers where predicted probability is between 0.35–0.50 (borderline zone).
- **Do not deploy on a customer cohort that differs significantly from the training population** (e.g. B2B customers, wholesale accounts, customers acquired in a new channel with no historical orders).
- **Do not use predictions older than 30 days** — the model is based on a 30-day web activity window; a prediction more than one month old may no longer reflect the customer's current state.
- **Do not use the model output in communications to customers.** A churn score is an internal operational tool, not a customer-facing metric.
