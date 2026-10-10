# D2C Customer Churn Prediction — End-to-End ML Pipeline

An end-to-end machine learning project for identifying Direct-to-Consumer (D2C) customers at risk of churning within the next 60 days. The repository covers data auditing, exploratory data analysis (EDA), RFM segmentation, behavioral feature engineering, LightGBM churn prediction, artifact management, and a FastAPI inference service complete with automated test coverage.

---

## 📋 Executive Summary & Objective

The primary goal of this project is to predict whether an active D2C customer is likely to churn during the **60-day period following the 2025-09-30 snapshot**. 

By identifying high-risk churn candidates early, retention teams can prioritize customers for targeted, cost-effective intervention workflows.

---

## 🏗️ Architecture & Data Flow

```text
               ┌────────────────────────┐
               │    D2C Customer Data   │
               └───────────┬────────────┘
                           │
                           ▼
               ┌────────────────────────┐
               │    Data Audit & EDA    │
               └───────────┬────────────┘
                           │
                           ▼
               ┌────────────────────────┐
               │   RFM & Behavioral     │
               │  Feature Engineering   │
               └───────────┬────────────┘
                           │
                           ▼
               ┌────────────────────────┐
               │ Churn Prediction Model │
               │      (LightGBM)        │
               └───────────┬────────────┘
                           │
                           ▼
               ┌────────────────────────┐
               │    model/model.pkl     │
               └───────────┬────────────┘
                           │
                           ▼
               ┌────────────────────────┐
               │  FastAPI REST Service  │
               │ ├── GET  /health       │
               │ ├── POST /predict      │
               │ └── POST /batch_predict│
               └───────────┬────────────┘
                           │
                           ▼
               ┌────────────────────────┐
               │  Automated Testing &   │
               │    Monitoring Plan     │
               └────────────────────────┘
```

---

## 📂 Repository Structure

```text
d2c-customer-churn-ml-pipeline/
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── customers.csv
│   ├── orders.csv
│   ├── support_tickets.csv
│   ├── web_events_snapshot.csv
│   ├── churn_labels.csv
│   ├── intervention_history.csv
│   ├── rfm_modeling_snapshot.csv
│   └── DATA_DICTIONARY.md
│
├── notebooks/
│   ├── 01_eda/
│   │   └── eda_audit.ipynb
│   ├── 02_rfm/
│   │   └── rfm_segmentation.ipynb
│   └── 03_model/
│       └── churn_model.ipynb
│
├── analysis/
│   ├── business_memo.md
│   ├── data_quality_report.md
│   ├── retention_strategy.md
│   ├── manual_review_cases.md
│   └── error_analysis.md
│
├── outputs/
│   ├── eda/
│   │   └── charts/
│   ├── rfm/
│   │   ├── charts/
│   │   ├── summary_metrics/
│   │   ├── tables/
│   │   └── segments.csv
│   └── model/
│       ├── charts/
│       └── error_cases.csv
│
├── model/
│   ├── model.pkl
│   ├── metrics.json
│   └── model_card.md
│
└── api/
    ├── app/
    │   ├── __init__.py
    │   └── main.py
    ├── tests/
    │   ├── __init__.py
    │   └── test_api.py
    ├── monitoring_plan.md
    └── README.md
```

---

## 📊 Pipeline Stages

### 1. Data Audit & EDA
The initial stage conducts quality auditing and exploratory analysis across customer demographics, orders, support tickets, web engagement events, churn labels, and intervention history.
* **Feature Cutoff:** All features are computed using data on or before **2025-09-30**.
* **Feature Windows:**
  * Order History: **180 days**
  * Support History: **90 days**
  * Web Activity: **30 days**

> **Note:** Post-snapshot order records are strictly excluded from feature computation to prevent temporal data leakage.

---

### 2. RFM & Behavioral Segmentation
Transactional and behavioral records are aggregated into RFM scores and customer-level features:
* **Recency:** Time since the customer's last purchase.
* **Frequency:** Total number of purchases within the feature window.
* **Monetary Value:** Total customer spending within the feature window.

In addition, support ticket frequencies and web engagement metrics are blended to derive comprehensive customer profile segments.

---

### 3. Feature Engineering & Leakage Controls
The final model utilizes **29 features** (25 base features + 4 custom derived features):

| Feature | Formula | Purpose |
| :--- | :--- | :--- |
| `engagement_rate` | `cart_adds_30d / (product_views_30d + 1)` | Purchase-intent signal |
| `email_click_rate` | `campaign_clicks_30d / (email_opens_30d + 1)` | Campaign engagement quality |
| `spend_per_order` | `monetary_180d / (frequency_180d + 1)` | Average basket-size proxy |
| `recency_x_frequency` | `log1p(recency_days) × (frequency_180d + 1)` | Combined RFM interaction |

* Missing entries in `loyalty_tier` are imputed with the category `"Not_Enrolled"`.
* Strict split separation is enforced via a pre-assigned `split` column to prevent train/validation/test contamination.

---

## 📈 Model Training & Evaluation

### Model Configuration
* **Algorithm:** LightGBM Churn Classifier (v1.0)
* **Baseline Comparison:** Logistic Regression (default threshold: `0.50`)
* **Optimal Decision Threshold:** `0.35` (Tuned on validation set to optimize $F_2$-score, prioritizing recall over precision).

### Dataset Splits
Total dataset size: **2,400 samples**

| Split | Samples | Churn Rate |
| :--- | :---: | :---: |
| **Train** | 1,728 | 46.99% |
| **Validation** | 336 | 43.75% |
| **Test** | 336 | 50.00% |

---

### Performance Metrics

#### Validation Set ($n = 336$)

| Metric | LightGBM (Threshold = 0.35) | Baseline Logistic Regression (Threshold = 0.50) |
| :--- | :---: | :---: |
| **Accuracy** | 0.7589 | **0.7976** |
| **Precision** | 0.6602 | **0.7616** |
| **Recall** | **0.9252** | 0.7823 |
| **F1-Score** | 0.7705 | **0.7718** |
| **F2-Score** | **0.8564** | — |
| **ROC-AUC** | 0.8757 | **0.8787** |
| **PR-AUC** | 0.8540 | **0.8654** |

#### Test Set ($n = 336$, LightGBM @ Threshold = 0.35)

| Metric | Value |
| :--- | :---: |
| **Accuracy** | 0.7500 |
| **Precision** | 0.6795 |
| **Recall** | **0.9464** |
| **F1-Score** | 0.7910 |
| **F2-Score** | **0.8775** |
| **ROC-AUC** | 0.8610 |
| **PR-AUC** | 0.8340 |

#### Test Confusion Matrix

```text
                      Predicted
                   Not Churn   Churn
Actual Not Churn      93        75
Actual Churn           9       159
```

---

## 🚀 Deployment & Service Interface

### Artifact Management
The project strictly maintains a single, centralized model binary at `model/model.pkl` to prevent model version mismatch between training and deployment pipelines. Associated outputs (`metrics.json`, `model_card.md`) reside alongside the main binary.

### FastAPI REST Service
Exposes RESTful endpoints for real-time and batch predictions.

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/health` | Verify model status and service health |
| `POST` | `/predict` | Single customer churn risk prediction |
| `POST` | `/batch_predict` | Batch predictions for multiple customer profiles |

---

## 🛠️ Quickstart Guide

### 1. Environment Setup

```bash
# Create and activate virtual environment
python -m venv .venv

# Windows PowerShell:
.venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Executing the Pipeline Notebooks

Run Jupyter from the project root directory:

```bash
jupyter notebook
```

Execute the analysis and modeling pipeline sequentially:
1. `notebooks/01_eda/eda_audit.ipynb`
2. `notebooks/02_rfm/rfm_segmentation.ipynb`
3. `notebooks/03_model/churn_model.ipynb`

### 3. Launching the API Service

Start the Uvicorn server from the project root:

```bash
uvicorn api.app.main:app --reload
```

Access the interactive API documentation (Swagger UI) at:
[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

### 4. Running Automated Tests

Run the test suite using `pytest`:

```bash
pytest api/tests/test_api.py -v
```

> **Verified Status:** `55 passed in 7.53s`

---

## 🛡️ Governance & Model Limitations

### Monitoring Plan
Detailed operational policies are documented in `api/monitoring_plan.md`, outlining guidelines for:
* Input data drift and statistical shift monitoring
* Delay-adjusted actual performance evaluation
* Automated retraining triggers and incident response
* Algorithmic fairness and responsible AI standards

### Key Limitations
* **Snapshot Dependency:** Trained on a single snapshot (`2025-09-30`); requires evaluation across multiple temporal snapshots.
* **Correlational, Not Causal:** Predictions capture risk indicators but do not establish causality.
* **Distribution Divergence:** Validation and test split churn distributions vary slightly; continuous monitoring is required.

---

## 🛠️ Tech Stack

* **Core:** Python, Pandas, NumPy
* **Machine Learning:** LightGBM, Scikit-learn, SHAP, Joblib
* **API & Testing:** FastAPI, Pydantic, Uvicorn, Pytest
* **Environment & Tools:** Jupyter Notebook, Git

---

## 📌 Project Component Status

| Component | Status |
| :--- | :---: |
| EDA Notebook | ✅ Complete |
| RFM Segmentation Notebook | ✅ Complete |
| Churn Modeling Notebook | ✅ Complete |
| Model Artifact (`model.pkl`) | ✅ Available |
| Metrics Artifact (`metrics.json`) | ✅ Available |
| Model Card (`model_card.md`) | ✅ Available |
| FastAPI Endpoint Service | ✅ Active |
| API Test Suite | ✅ **55 Passed** |
| Monitoring Specification | ✅ Documented |

---

## 👤 Author

**Mohammed Rashiku B C**