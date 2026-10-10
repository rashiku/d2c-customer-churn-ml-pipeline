# FastAPI Inference Service

This directory contains the FastAPI service used to serve the trained customer churn prediction model.

---

## 🏗️ Architecture

```
model/model.pkl
       │
       ▼
FastAPI Application
       │
       ├── GET  /health
       ├── POST /predict
       └── POST /batch_predict
```

---

## 🚀 Running the API

Run the following command from the project root directory:

```bash
uvicorn api.app.main:app --reload
```

### 📖 API Documentation

Once the server is running, interactive API documentation is available at:

- **Swagger UI:** [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

---

## 🧪 Testing

Execute the test suite using `pytest`:

```bash
pytest api/tests/test_api.py -v
```

---

## 📦 Model Artifact

- **Path:** `model/model.pkl`
- The API dynamically loads the trained model from the specified file path and does not maintain a separate copy.