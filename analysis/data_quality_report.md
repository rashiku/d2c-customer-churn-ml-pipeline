# Data Quality Report
## D2C Customer Churn Capstone — Part 1

**Snapshot date:** `2025-09-30`  
**Report prepared from:** executed `eda_audit.ipynb`  
**Universe:** 2,400 customers | 6 source files

---

## Executive Summary

The dataset is broadly fit for modelling. All six files join cleanly on `customer_id` with no orphaned keys. The main issues are a large block of missing loyalty enrolment data, a designed-in set of duplicate-like order rows, a meaningful tail of high-value order outliers, and 1,872 post-snapshot order rows that must be excluded to prevent label leakage. None of these issues are blockers, but each requires an explicit treatment decision before feature engineering begins.

---

## 1. Missing Values

| File | Column | Missing Count | Missing % | Issue Class |
|---|---|---|---|---|
| `customers.csv` | `loyalty_tier` | 1,386 | **57.8%** | Structural — not enrolled, not MCAR |
| `customers.csv` | `skin_type` | 401 | 16.7% | Self-reported; may be MNAR |
| `orders.csv` | `rating` | 80 | 0.8% | Customer did not rate |

### 1.1 `loyalty_tier` — Critical

With 57.8% of customers having no loyalty tier, this is not a random missingness pattern. These customers were never enrolled in the loyalty programme. Treating the nulls as a category called `"not_enrolled"` is semantically correct and should be preferred over imputation. The churn rate for unenrolled customers (48.3%) is modestly higher than for enrolled customers (45.1%), confirming the null carries predictive signal.

**Recommended treatment:** Encode as a 4-level categorical: `Silver`, `Gold`, `Platinum`, `not_enrolled`.

### 1.2 `skin_type` — Low Impact

16.7% missing. This column may be MNAR (customers with sensitive or dry skin may be less likely to disclose). It has no direct role in the RFM or behavioural feature set. Impute with a `"not_provided"` sentinel or exclude from the initial feature set.

### 1.3 `orders.rating` — Low Impact

Only 80 rows (0.8%) across 10,009 order records. When computing `avg_rating` per customer, use `skipna=True` (pandas default). Flag any customer whose rating is computed from a single observation, as their score will be noisy.

---

## 2. Duplicate Records

### 2.1 `_DUP` Order Records

Twelve order IDs carry a `_DUP` suffix (e.g. `ORD008249_DUP`). All 12 match an original row exactly on `order_id` (stripped), `customer_id`, `order_date`, and `gross_amount`. They represent 0.1% of total order rows and are distributed across 12 distinct customers.

**Recommended treatment:** Drop all rows where `order_id.str.endswith('_DUP')` before any feature computation. This is a deterministic filter — no ambiguity about which row to keep.

### 2.2 Customer-Level Duplicates

Zero duplicate `customer_id` values in `customers.csv`. All other files that carry `customer_id` have clean keys with no orphans.

### 2.3 Fully Identical Rows

No dataset contains any fully duplicate rows (all columns identical).

---

## 3. Outliers — `orders.gross_amount`

| Statistic | Value |
|---|---|
| Q1 | ≈ ₹435 |
| Q3 | ≈ ₹942 |
| IQR × 3 upper fence | **₹2,396** |
| Outlier row count | **73 rows (0.9%)** |
| Outlier mean | ₹3,702 |
| Outlier max | **₹24,789** |

73 pre-snapshot order rows exceed the 3×IQR upper fence of ₹2,396. These are not data errors — high-value orders (e.g. bundled Wellness or Fragrance sets) are plausible for a D2C beauty brand. However, the tail extends to ₹24,789, which warrants investigation.

**Impact:** If `total_spend` or `avg_order_value` are used as raw features without any treatment, the extreme values in this 0.9% tail will distort both the feature distribution and any model that lacks tree-based splitting (e.g. logistic regression, KNN). Tree models (XGBoost, Random Forest) are largely robust to this.

**Recommended treatment options (choose one):**
- Winsorise `gross_amount` at the 99th percentile (~₹4,200) before aggregating.
- Log-transform `total_spend` and `avg_order_value` as model inputs.
- Leave raw if using gradient-boosted trees; document the decision.

**Other numeric columns checked:**
- `discount_pct`: 0 values outside [0, 1]. ✓
- `rating`: 0 values outside [1, 5]. ✓
- `delivery_days`: 0 values outside [1, 11]. ✓

---

## 4. Date Consistency & Post-Snapshot Leakage

| Check | Result |
|---|---|
| Post-snapshot order rows | **1,872 rows (18.7%)** |
| Pre-snapshot order rows (safe) | 8,137 rows |
| `signup_date` range | `2024-01-01` → `2025-09-15` — all before snapshot ✓ |
| Customers signed up after snapshot | 0 ✓ |
| Ticket dates after snapshot | 0 ✓ |

### 4.1 Post-Snapshot Orders — High Severity Leakage Risk

1,872 order rows (18.7% of all orders) have `order_date > 2025-09-30`. These rows are part of the target-label construction logic — they define whether `churn_next_60d` is 0 or 1. **Using any of these rows as model features would constitute direct label leakage** and invalidate all evaluation metrics.

**Required action:** Filter orders with `order_date <= 2025-09-30` before any feature computation. This filter must be applied before aggregation, not after. A post-aggregation filter would still contaminate features if order-level data had been processed together.

The pre-snapshot set of 8,137 rows spans `2024-01-09` to `2025-09-30` and covers all 2,400 customers with at least one order.

---

## 5. Join / Key Integrity

| Dataset | IDs in File | Orphans (not in customers) | Customers with No Record |
|---|---|---|---|
| `customers` | 2,400 | 0 | — |
| `orders` | 2,400 | 0 | 0 |
| `tickets` | 1,247 | 0 | **1,153** |
| `web_events` | 2,400 | 0 | 0 |
| `churn_labels` | 2,400 | 0 | 0 |
| `interventions` | 2,400 | 0 | 0 |

### 5.1 Support Tickets — Expected Sparsity

1,153 customers (48.0% of the universe) have no support tickets at all. This is expected behaviour for a D2C brand — most customers never contact support. It is **not** a data error.

**Modelling implication:** Ticket-derived features (`ticket_count`, `avg_sentiment`, `avg_resolution_hours`) will be `NaN` for these 1,153 customers after a left join. Treat NaN `ticket_count` as 0. For sentiment and resolution hours, impute with the population median or a neutral sentinel (e.g. 0.0 for sentiment) and add a binary indicator `has_ticket`.

### 5.2 No Orphaned Keys

Zero `customer_id` values appear in any file that are absent from `customers.csv`. All joins are safe.

### 5.3 Zero-Order Customers

All 2,400 customers have at least one pre-snapshot order. There are no customers with an entirely empty purchase history in the modelling window.

---

## 6. Columns with Leakage Risk

| Column | File | Risk | Recommended Action |
|---|---|---|---|
| `order_date > 2025-09-30` | `orders.csv` | **CRITICAL** — direct leakage | Exclude all rows unconditionally |
| `churn_next_60d` | `churn_labels.csv` / `rfm_modeling_snapshot.csv` | **CRITICAL** — target variable | Never use as a feature input |
| `split` | `churn_labels.csv` | Administrative — not leakage, but irrelevant as a feature | Exclude from feature matrix |
| `snapshot_date` | Multiple files | Constant (always `2025-09-30`); zero variance | Drop from feature matrix |
| `manual_priority_bucket` | `intervention_history.csv` | Potentially uses forward-looking information depending on how the CRM team assigned it | Use only as a baseline comparison, not a primary model feature; validate that assignment used only pre-snapshot data |

---

## 7. Summary Table

| Issue | Severity | Action Required |
|---|---|---|
| Post-snapshot orders (1,872 rows) | 🔴 Critical | Exclude before any feature engineering |
| `churn_next_60d` as feature | 🔴 Critical | Never use as input |
| `_DUP` order rows (12 rows) | 🟡 Medium | Drop `order_id.str.endswith('_DUP')` |
| `gross_amount` outliers (73 rows, max ₹24,789) | 🟡 Medium | Winsorise or log-transform before linear models |
| `loyalty_tier` nulls (1,386 rows, 57.8%) | 🟡 Medium | Encode as `not_enrolled`; do not impute |
| `skin_type` nulls (401 rows, 16.7%) | 🟢 Low | Sentinel encoding or exclude |
| `rating` nulls (80 rows, 0.8%) | 🟢 Low | `skipna=True` in mean computation |
| 1,153 customers with no support ticket | 🟢 Low | Fill `ticket_count` with 0; add `has_ticket` flag |
| `manual_priority_bucket` leakage risk | 🟡 Medium | Validate assignment timing; use as baseline only |

---

*Report generated from `eda_audit.ipynb` executed on the full 2,400-customer universe as of snapshot date `2025-09-30`.*
