# SOC Lite AI IDS — Resume Package

## Project Summary

End-to-end **network intrusion detection system** combining a **PyTorch MLP training pipeline**, **FastAPI inference API** (single-vector and CSV batch upload), and a **multi-page SOC dashboard** with Plotly visualizations. Supports CICIDS2017/UNSW-NB15-style tabular traffic data with class-weighted training, artifact-based deployment, and Docker/Render configuration.

---

## GitHub & Demo Links

| | URL |
|---|-----|
| **GitHub** | https://github.com/Ananyanagaraj11/soc-lite-ai-ids |
| **Live Demo** | No public deployment found |

---

## Tech Stack

| Layer | Technologies |
|-------|--------------|
| **ML** | PyTorch, scikit-learn, pandas, NumPy, joblib |
| **Backend** | FastAPI, Uvicorn, Pydantic, python-multipart |
| **Frontend** | HTML5, CSS3, JavaScript, Plotly.js |
| **DevOps** | Docker, Render (`render.yaml`), Vercel config |
| **Optional** | SHAP (explainability module present) |

---

## Amazon-Style Resume Bullets

1. Built a **FastAPI intrusion-detection service** with **5 REST endpoints** including multipart CSV batch inference, health monitoring, and auto-generated **OpenAPI/Swagger** documentation served at `/docs`.

2. Developed a **PyTorch MLP training pipeline** with **3 hidden layers** (128→64→32), BatchNorm, Dropout, and **balanced class-weighted CrossEntropyLoss**, exporting **4 deployable artifacts** (`model.pt`, `scaler.joblib`, `classes.pt`, `feature_names.txt`).

3. Implemented **CICIDS2017 multi-file dataset ingestion** with configurable row caps, numeric feature extraction, inf/NaN cleaning, and **StandardScaler** normalization fit on training data only.

4. Delivered a **SOC-style security dashboard** (**5 HTML pages**, **3 JavaScript controllers**) rendering KPIs, attack-distribution charts, confidence histograms (**5 buckets**), and a results table fed by backend analysis APIs.

5. Designed **batch CSV analysis** processing full uploaded datasets in memory, aggregating class distributions and returning up to **1,000 prediction rows** per response with summary statistics for dashboard visualization.

6. Containerized the application with **Docker** (Python 3.11-slim, CPU PyTorch) and **Render blueprint** including `/health` health-check integration for production deployment.

7. Created an **offline model evaluation toolchain** generating accuracy, macro F1, per-class metrics, confusion matrix plots, and training-curve visualizations from saved checkpoints.

8. Integrated **PyTorch 2.6+ safe model loading** and graceful degradation when artifacts are missing, exposing `model_loaded` status via the health endpoint for ops monitoring.

---

## Measurable Implementation Facts

1. **5** implemented FastAPI HTTP routes (`/`, `/health`, `/api/last-analysis`, `/predict`, `/analyze/csv`).
2. **6** Python source modules spanning training, inference, evaluation, and explainability.
3. **1,000** — maximum prediction rows returned per CSV analysis API response (code-enforced cap).
4. **4** model artifact files required per deployment (`model.pt`, `scaler.joblib`, `classes.pt`, `feature_names.txt`).
5. **85** project files excluding virtual environment (including **5** dashboard pages and **3** deployment configs).

---

## One-Line Resume Summary

> **SOC Lite AI IDS** — PyTorch + FastAPI network intrusion detection system with CSV batch inference, SOC dashboard (Plotly), CICIDS2017 training pipeline, and Docker/Render deployment config.

---

## Suggested LinkedIn / GitHub Description

```
SOC Lite AI IDS | PyTorch · FastAPI · SOC Dashboard
Network intrusion detection: train MLP on CICIDS2017-style traffic data,
serve batch CSV + single-vector predictions via REST API, visualize threats
in an interactive SOC dashboard.
github.com/Ananyanagaraj11/soc-lite-ai-ids
```

---

## Before Publishing on GitHub — Checklist

- [ ] Train model on CICIDS2017 and commit artifacts (or add CI step to generate them)
- [ ] Deploy to Render and set GitHub **homepage** to live URL
- [ ] Implement `GET /config` (dashboard live-predict button depends on it)
- [ ] Align README API list with actual routes
- [ ] Add pytest smoke tests for `/health` and `/predict`

---

*All bullets and metrics verified against source code in `Ananyanagaraj11/soc-lite-ai-ids`. No accuracy percentages from CICIDS2017 are claimed — upstream repo does not ship a trained production model.*
