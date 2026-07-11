# SOC Lite AI IDS — Technical Validation Report

**Repository:** https://github.com/Ananyanagaraj11/soc-lite-ai-ids  
**Validation date:** July 11, 2026  
**Local clone:** `c:\Users\anany\Downloads\Resume\_analysis\soc-lite-ai-ids`

---

## Deployment & URLs

| Item | Value |
|------|-------|
| **GitHub Repository** | https://github.com/Ananyanagaraj11/soc-lite-ai-ids |
| **Live Demo** | **No public deployment found.** GitHub `homepage` is empty. Documented Render URL (`https://ai-cyber-threat-dashboard-1.onrender.com`) did not respond to health checks during validation. |
| **Dashboard (local)** | http://127.0.0.1:8000/dashboard/index.html |
| **Backend API (local)** | http://127.0.0.1:8000 |
| **Swagger Docs (local)** | http://127.0.0.1:8000/docs |
| **Health Endpoint (local)** | http://127.0.0.1:8000/health |
| **Login** | None — no authentication implemented |

### Local run verification (passed)

| Check | Result |
|-------|--------|
| `GET /health` | `{"status":"healthy","model_loaded":true}` |
| `GET /docs` | HTTP 200 — 5 routes documented in OpenAPI |
| `POST /predict` | Returns `predicted_class` + `confidence` |
| `POST /analyze/csv` | Processed 3,000 rows; returned 1,000 prediction rows (API cap) |
| Dashboard landing | HTTP 200 |
| Model artifacts loaded at startup | Yes |

---

## 1. Project Overview

**SOC Lite AI IDS** is an end-to-end network intrusion detection demo: train a PyTorch MLP on tabular network-traffic features, serve predictions through a **FastAPI** backend, and visualize results in a **static SOC-style dashboard** (HTML/CSS/JS + Plotly.js).

It is designed for CICIDS2017 / UNSW-NB15-style CSV data but runs locally with a bundled sample CSV generator when full datasets are not present.

---

## 2. Problem Solved

Security operations teams need to **classify network traffic events** (benign vs. attack types) at scale. This project provides:

- **Batch CSV inference** for SOC analysts uploading traffic exports
- **Single-vector inference** via REST API
- **Visual KPIs and charts** (attack rate, class distribution, confidence buckets, alerts table)
- **Reproducible training pipeline** for CICIDS2017 directory or single CSV inputs

---

## 3. Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Browser (SOC Analyst)                    │
│  index.html → analysis.html → dashboard.html                 │
│  Plotly.js charts · fetch() to FastAPI                       │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              FastAPI (backend/app.py) :8000                  │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐ │
│  │ StaticFiles │  │ REST API     │  │ In-memory store     │ │
│  │ /dashboard  │  │ predict      │  │ last_analysis_store │ │
│  └─────────────┘  │ analyze/csv  │  └─────────────────────┘ │
│                   │ health       │                           │
│                   └──────┬───────┘                           │
│                          ▼                                   │
│              PyTorch MLP + StandardScaler                      │
│              backend/artifacts/*.pt, *.joblib                │
└─────────────────────────────────────────────────────────────┘

Training (offline):
  dataset_loader.py → train.py → training/outputs/
  → copy_artifacts.bat → backend/artifacts/
```

**Services required locally:** **1** — Uvicorn serving FastAPI (API + static dashboard).

**Database:** None. Last CSV analysis stored in process memory (`last_analysis_store` dict).

---

## 4. Request Flow

### Flow A — CSV batch analysis (primary UX)

1. User opens `/dashboard/analysis.html`
2. User selects/drops a CSV file
3. `upload.js` POSTs `multipart/form-data` to `POST /analyze/csv`
4. Backend: parse CSV → drop label column if present → numeric features only → `StandardScaler.transform` → MLP forward pass → softmax
5. Backend builds summary (class counts, attack totals, 5 confidence buckets) and stores full result in `last_analysis_store`
6. Response includes up to **1,000** prediction rows (`predictions[:1000]` in code)
7. User navigates to `/dashboard/dashboard.html`
8. `dashboard.js` GETs `/api/last-analysis` and renders KPIs + Plotly charts

### Flow B — Single prediction

1. Client POSTs JSON `{ "features": [float, ...] }` to `POST /predict`
2. Backend scales features, runs inference, returns `{ predicted_class, confidence }`

### Flow C — Health / ops

- `GET /health` → `{ status, model_loaded }`
- `GET /` → 302 redirect to `/dashboard/index.html`

---

## 5. ML Pipeline

| Stage | Module | Description |
|-------|--------|-------------|
| Data ingest | `training/dataset_loader.py` | Load single CSV or CICIDS2017 directory; infer label column |
| Cleaning | `dataset_loader.py` | Drop ID columns, handle inf/NaN, numeric-only features |
| Encoding | `dataset_loader.py` | `LabelEncoder` for labels |
| Split | `dataset_loader.py` | 80/20 train/val, stratified when possible |
| Scaling | `dataset_loader.py` | `StandardScaler` fit on train only |
| Class imbalance | `train.py` | `compute_class_weight("balanced")` → weighted CrossEntropyLoss |
| Training | `training/train.py` | Adam optimizer, configurable epochs (default **50**) |
| Artifacts | `train.py` | Saves `model.pt`, `classes.pt`, `scaler.joblib`, `feature_names.txt`, `history.json` |
| Evaluation | `training/evaluate.py` | Accuracy, macro F1, confusion matrix PNG, metrics JSON |
| Inference | `backend/app.py` | Load artifacts at startup; batch + single predict |

### Current local model (synthetic sample — for demo only)

Generated via `scripts/generate_sample_data.py`:

| Metric | Value (from `training/outputs/history.json`, 8 epochs) |
|--------|------------------------------------------------------|
| Training rows | 3,000 |
| Features | 20 (`f1`–`f20`) |
| Classes | 2 (`Normal`, `Attack`) |
| Best val accuracy (epoch 2) | 0.5317 |
| Best val macro F1 (epoch 2) | 0.5144 |
| Hidden layers (checkpoint) | (128, 64, 32) |

> **Note:** Model artifacts are **not committed** to the GitHub repo (only `feature_names.txt` was present upstream). Artifacts must be trained locally or copied before inference works.

---

## 6. Data Preprocessing

### Training (`prepare_for_training`)

- Removes identifier columns (`id`, `srcip`, `dstip`, etc.)
- Replaces ±inf with NaN; drops all-NaN columns
- Mean-imputes NaNs using **training column means only**
- Clips values to [-1e30, 1e30]
- `StandardScaler` on train; transform on validation

### Inference (`analyze_csv` / `predict`)

- Auto-detects label column `Label` or ` Label` (CICIDS2017 convention)
- Numeric columns only; inf → NaN → 0 fill
- Uses pre-trained `scaler.transform` (must match training feature count)

---

## 7. Model Architecture

### Training model — `IntrusionDetectionMLP` (`training/train.py`)

```
Input (n_features)
  → Linear → BatchNorm → ReLU → Dropout(0.3)   [128]
  → Linear → BatchNorm → ReLU → Dropout(0.3)   [64]
  → Linear → BatchNorm → ReLU → Dropout(0.3)   [32]
  → Linear (num_classes)
```

### Serving model — `CyberThreatModel` (`backend/app.py`)

Same pattern; hidden dims loaded from checkpoint (`hidden_dims` key, default `[128, 64]` if missing).

Checkpoint stores: `model_state_dict`, `input_dim`, `num_classes`, `hidden_dims`, `dropout`.

---

## 8. Dashboard Components

| Page | Path | Purpose |
|------|------|---------|
| Landing | `dashboard/index.html` | Navigation to CSV analysis and dashboard |
| CSV Analysis | `dashboard/analysis.html` | Upload, progress bar, analyze via `/analyze/csv` |
| Main Dashboard | `dashboard/dashboard.html` | KPIs, Plotly charts, alerts, results table |
| Debug | `dashboard/debug.html` | Developer debugging |
| Test | `dashboard/test.html` | Manual testing |

### Dashboard widgets (`dashboard.js`)

| Component | Element ID | Data source |
|-----------|------------|-------------|
| KPI — Total Events | `totalEvents` | `total_rows` or prediction count |
| KPI — Attack Events | `attackEvents` | `summary.total_attacks` |
| KPI — Attack % | `attackPercent` | computed |
| Attack distribution pie | `attackDistPie` | `class_distribution` |
| Top attack types bar | `topAttacksBar` | class counts |
| Severity over time | `severityLine` | confidence over index |
| Events timeline | `eventsTimeline` | cumulative counts |
| Confidence histogram | `confidenceDist` | 5 buckets from API |
| Recent alerts | alerts section | top predictions |
| CSV results table | table section | up to 1000 rows |

**Frontend stack:** HTML5, CSS3 (`dashboard.css`), vanilla JavaScript, Plotly.js, Moment.js (clock).

---

## 9. API Inventory

| Method | Route | Purpose | Request | Response |
|--------|-------|---------|---------|----------|
| GET | `/` | Redirect to dashboard | — | 302 → `/dashboard/index.html` |
| GET | `/health` | Liveness + model status | — | `{ status: "healthy", model_loaded: bool }` |
| GET | `/api/last-analysis` | Last CSV analysis | — | Analysis JSON or `null` |
| POST | `/predict` | Single inference | `{ features: float[] }` | `{ predicted_class, confidence }` or `{ error }` |
| POST | `/analyze/csv` | Batch CSV inference | `multipart/form-data` file field | `{ success, total_rows, predictions[], summary{} }` |

### Documented in README but **not implemented** in `backend/app.py`

| Route | Status |
|-------|--------|
| `GET /config` | **Missing** — `dashboard.js` calls this for live prediction button; returns 404 |
| `POST /predict/explain` | **Missing** — `explain.py` exists but is not wired to any route |

---

## 10. Folder Structure (project source, excluding `.venv`)

```
soc-lite-ai-ids/                    (85 files)
├── backend/
│   ├── app.py                      # FastAPI app (253 lines)
│   ├── explain.py                    # SHAP/gradient attribution (unused by API)
│   └── artifacts/                  # model.pt, scaler.joblib, classes.pt, feature_names.txt
├── dashboard/                      # 5 HTML pages, 3 JS, 1 CSS
├── docs/                           # GitHub Pages mirror of dashboard
├── training/
│   ├── train.py
│   ├── dataset_loader.py
│   ├── evaluate.py
│   └── outputs/
├── scripts/                        # 10 .bat helpers + generate_sample_data.py
├── data/sample_data.csv            # generated locally
├── requirements.txt
├── requirements-render.txt         # slim CPU torch for Render
├── Dockerfile
├── render.yaml
├── vercel.json
└── README.md + 19 other .md docs
```

**Python source files:** 6 (`app.py`, `explain.py`, `train.py`, `dataset_loader.py`, `evaluate.py`, `generate_sample_data.py`)

**Tests:** **None** in project source (no `tests/` directory, no pytest files).

---

## 11. Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| Single FastAPI process serves API + static dashboard | Simplifies local demo and Render deployment (one `uvicorn` command) |
| Artifacts loaded at import time | Fast inference; fails gracefully (`model_loaded: false`) if missing |
| In-memory `last_analysis_store` | No DB dependency; state lost on restart |
| CORS `allow_origins=["*"]` | Supports Vercel frontend → Render backend split |
| PyTorch 2.6+ `add_safe_globals` | Fixes `torch.load` security defaults for numpy reconstruct |
| CSV response capped at 1,000 rows | Limits payload size for browser dashboard |
| Attack count excludes class `"BENIGN"` only | CICIDS2017 convention; other label names (e.g. `Normal`) count as attacks |
| `explain.py` separated but unwired | Attribution logic ready; API endpoint not connected |
| Class-weighted loss | Handles imbalanced IDS datasets |

---

## 12. Docker & Deployment

### Dockerfile

- Base: `python:3.11-slim`
- Installs `requirements-render.txt` (CPU-only PyTorch)
- Copies `backend/` + `dashboard/`
- **Build fails** if `backend/artifacts/model.pt` missing
- CMD: `uvicorn backend.app:app --host 0.0.0.0 --port 8000`

### render.yaml

- Web service, health check `/health`, start command uvicorn

### vercel.json

- Expects `dashboard/package.json` + `npm run build` — **no `package.json` scripts for build** in dashboard (only metadata file exists); Vercel config may be incomplete

---

## 13. Resume-Worthy Accomplishments (code-backed)

1. **Built a FastAPI inference service** with 5 HTTP routes including multipart CSV upload and OpenAPI docs.
2. **Implemented a full PyTorch training pipeline** with stratified split, StandardScaler, balanced class weights, and artifact export (4 files per model).
3. **Designed a multi-page SOC dashboard** with 5 Plotly chart types and KPI cards fed by REST API.
4. **Supported CICIDS2017 multi-file ingestion** via directory loader with row caps (`load_cicids2017`, `max_rows_total`).
5. **Containerized the application** with Dockerfile + Render blueprint including health-check path.
6. **Implemented batch inference** processing full CSV in memory with summary aggregation and confidence histogram buckets (5 bins).
7. **Added gradient-based feature attribution module** (`explain.py`) with SHAP fallback path (module present; API not exposed).
8. **Created offline evaluation tooling** producing `metrics.json`, confusion matrix PNG, and training curve plots.

---

## 14. Verifiable Metrics (from implementation)

| Metric | Count / Value |
|--------|---------------|
| FastAPI routes (implemented) | **5** |
| Python modules (project) | **6** |
| Dashboard HTML pages | **5** |
| Dashboard JS controllers | **3** (`config.js`, `upload.js`, `dashboard.js`) |
| MLP hidden layers (training default) | **3** (128 → 64 → 32) |
| Confidence buckets in API response | **5** |
| Max predictions returned per CSV request | **1,000** |
| Model artifact files per deployment | **4** |
| Deployment configs | **3** (Dockerfile, render.yaml, vercel.json) |
| Supported dataset loaders | **2** (single CSV, CICIDS2017 directory) |
| Project files (excl. `.venv`) | **85** |
| Default training epochs (CLI default) | **50** |
| Sample CSV generator rows | **3,000** |
| Sample CSV features | **20** |
| Unit / integration tests | **0** |

---

## 15. Technical Skills Proven by Code

- Python 3.10+
- FastAPI, Uvicorn, Pydantic, CORS middleware
- PyTorch (nn.Module, BatchNorm, Dropout, softmax inference)
- scikit-learn (StandardScaler, LabelEncoder, class weights, metrics)
- pandas / NumPy data wrangling
- REST API design (JSON + multipart upload)
- Static file serving (FastAPI StaticFiles)
- JavaScript (async fetch, DOM, Plotly.js)
- HTML/CSS SOC UI theming
- Docker containerization
- Render / Vercel deployment configuration
- joblib persistence
- Optional SHAP explainability

---

## 16. GitHub / Resume Readiness Gaps

| Gap | Severity | Recommendation |
|-----|----------|----------------|
| Model artifacts not in repo | **High** | Commit trained model OR document one-command train + GitHub Actions artifact |
| No working public demo | **High** | Deploy to Render with artifacts; set GitHub homepage URL |
| `GET /config` missing | **Medium** | Add endpoint returning `input_dim`, `class_names` |
| `explain.py` not exposed | **Medium** | Wire `POST /predict/explain` or remove from README |
| No automated tests | **Medium** | Add pytest for `/health`, `/predict`, `/analyze/csv` |
| README lists unimplemented endpoints | **Low** | Align README with `app.py` |
| Vercel build config references npm build | **Low** | Fix or remove vercel.json |

---

## 17. Interview Questions & Answers

### Architecture & Design

**Q1: Why serve the dashboard from FastAPI instead of a separate frontend server?**  
**A:** `backend/app.py` mounts `dashboard/` via `StaticFiles` at `/dashboard`, so one Uvicorn process handles both API and UI. This matches the Render deployment model (single web service) and eliminates CORS complexity for local development on port 8000.

**Q2: How does state persist between the CSV upload page and the dashboard?**  
**A:** After `POST /analyze/csv`, results are stored in an in-memory dict `last_analysis_store["data"]`. The dashboard page calls `GET /api/last-analysis` on load. State is lost on server restart.

**Q3: Why is there no database?**  
**A:** The project targets a demo/SOC prototype. Persistence requirements are limited to “last analysis,” handled in memory. Production would need Redis or PostgreSQL for multi-user history.

### ML Pipeline

**Q4: Walk through training data preprocessing.**  
**A:** `prepare_for_training()` in `dataset_loader.py` loads CSV or CICIDS2017 directory, drops ID columns, keeps numeric features, encodes labels with `LabelEncoder`, splits 80/20 (stratified), imputes NaN from training means, clips extremes, and fits `StandardScaler` on train only.

**Q5: How do you handle class imbalance?**  
**A:** `get_class_weights()` uses sklearn `compute_class_weight("balanced")`, passed to `CrossEntropyLoss(weight=...)` in `train.py`.

**Q6: What is the model architecture?**  
**A:** MLP: alternating Linear → BatchNorm1d → ReLU → Dropout(0.3) for hidden sizes (128, 64, 32), then a linear classification head. Serving uses the same structure via `CyberThreatModel` with dims from checkpoint.

**Q7: How are artifacts versioned for deployment?**  
**A:** `train.py` writes `model.pt` (state dict + metadata), `classes.pt`, `scaler.joblib`, `feature_names.txt`. `scripts/copy_artifacts.bat` copies them to `backend/artifacts/` where `app.py` loads at startup.

### API & Inference

**Q8: What happens when model files are missing?**  
**A:** The try/except block in `app.py` sets `model = None`. `/health` returns `model_loaded: false`. `/predict` and `/analyze/csv` return error responses instead of crashing the server.

**Q9: How does batch CSV inference work?**  
**A:** File is read into pandas, optional label column stripped, numeric matrix scaled, full tensor forward pass, softmax per row. Summary aggregates class counts and 5 confidence buckets. Response truncates to 1,000 prediction objects.

**Q10: Why does the README mention `/config` but Swagger doesn't show it?**  
**A:** `dashboard.js` `runLivePrediction()` fetches `/config` for `input_dim`, but the route was never added to `app.py`. This is a known gap — live prediction button fails; CSV flow works.

### Frontend

**Q11: How does the dashboard know which API URL to call?**  
**A:** `config.js` sets `window.ENV_API_BASE` to `http://localhost:8000` on localhost, else defaults to a Render URL. `dashboard.js` uses same-origin when port is 8000.

**Q12: What chart library is used and why?**  
**A:** Plotly.js — supports interactive pie, bar, line, and scatter charts with dark-theme layouts defined in `dashboard.js`.

### Security & Production

**Q13: Is there authentication?**  
**A:** No. All endpoints are public. CORS is fully open (`allow_origins=["*"]`).

**Q14: What would you change for production?**  
**A:** Add auth (API keys/JWT), restrict CORS, persist analysis history, add rate limiting on `/analyze/csv`, pin dependency versions, add health checks for scaler/feature dimension mismatch, and deploy models via object storage rather than git.

**Q15: How does Docker enforce model presence?**  
**A:** Dockerfile `RUN test -f backend/artifacts/model.pt || exit 1` fails the image build if artifacts are missing.

### Data & Evaluation

**Q16: How does the loader support CICIDS2017?**  
**A:** `load_cicids2017()` globs `*.csv` in a directory, concatenates with optional per-file and total row caps, infers `Label` column.

**Q17: What metrics does `evaluate.py` produce?**  
**A:** Accuracy, macro F1, per-class precision/recall/F1, full sklearn classification report JSON, confusion matrix PNG, optional training curve PNG from `history.json`.

### Debugging & Edge Cases

**Q18: How are CICIDS2017 infinite values handled?**  
**A:** Training replaces inf with NaN, drops empty columns, mean-imputes. Inference replaces inf with NaN then fills 0.

**Q19: Why might attack count be wrong for non-CICIDS labels?**  
**A:** `attack_count` sums classes where name != `"BENIGN"`. Datasets using `Normal` instead of `BENIGN` will count Normal traffic as attacks.

**Q20: What is `explain.py` for?**  
**A:** Computes top-k feature contributions via gradient attribution (or SHAP if installed). Intended for `/predict/explain` but not connected to FastAPI yet.

---

## 18. Environment Configuration

**`.env` (created for local run):**

```env
# No secrets required.
# Backend reads model artifacts from backend/artifacts/
```

**Secrets required:** None for local inference. Training on private datasets requires no cloud credentials.

---

*Report generated from source inspection and live local validation. No metrics invented beyond what appears in code, config, or measured local runs.*
