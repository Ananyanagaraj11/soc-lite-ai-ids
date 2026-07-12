# 🛡️ AI-Powered Cyber Threat Intelligence Dashboard

> Real-time network intrusion detection system with ML-powered attack classification, threat severity scoring, and interactive SOC-style dashboard.

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-orange)](https://pytorch.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-green)](https://fastapi.tiangolo.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

**Topics:** cybersecurity, machine-learning, pytorch, fastapi, threat-detection, network-security, intrusion-detection, dashboard

<p align="center">
  <a href="https://ai-cyber-threat-dashboard-1.onrender.com/"><strong>Live demo</strong></a> ·
  <a href="https://ai-cyber-threat-dashboard-1.onrender.com/docs"><strong>API docs</strong></a> ·
  <a href="#-quick-start"><strong>Run locally</strong></a> ·
  <a href="#-sample-data"><strong>Sample data</strong></a>
</p>

## Live demo

**[https://ai-cyber-threat-dashboard-1.onrender.com/](https://ai-cyber-threat-dashboard-1.onrender.com/)** — SOC dashboard + FastAPI backend (upload CSV, view threat KPIs and charts)

| Environment | URL |
|-------------|-----|
| **Public app (Render)** | [ai-cyber-threat-dashboard-1.onrender.com](https://ai-cyber-threat-dashboard-1.onrender.com/) |
| **Swagger** | [/docs](https://ai-cyber-threat-dashboard-1.onrender.com/docs) |
| **Health** | [/health](https://ai-cyber-threat-dashboard-1.onrender.com/health) |
| **Local** | http://localhost:8000/ |

## 📊 Sample data

Bundled CSV for training and dashboard demos (no external dataset required):

| File | Rows | Features | Labels |
|------|------|----------|--------|
| [`data/sample_data.csv`](data/sample_data.csv) | **3,000** | `f1`–`f20` (20) | **1,819 Normal**, **1,181 Attack** |

```bash
# Regenerate if missing
python scripts/generate_sample_data.py

# Train on sample data (quick local demo)
python training/train.py --data data/sample_data.csv

# Analyze via API
curl -X POST "https://ai-cyber-threat-dashboard-1.onrender.com/analyze/csv" \
  -F "file=@data/sample_data.csv"
```

## 🎯 Overview

An end-to-end machine learning pipeline for detecting and classifying cyber attacks from network traffic data. Train deep learning models on CICIDS2017/UNSW-NB15 datasets, deploy via FastAPI, and visualize threats in real-time through an interactive dashboard.

**Key Features:**
- 🔍 **Multi-class attack detection** (BENIGN, Bot, DDoS, PortScan, etc.)
- 📊 **Interactive dashboard** with KPIs, charts, and real-time alerts
- 📈 **CSV batch analysis** with detailed results and accuracy metrics
- 🎯 **Feature importance** explanations for model predictions
- 🚀 **One-click deployment** to Vercel + Render for live demo

## 🚀 Quick Start

### Prerequisites

- Python 3.10+ ([Download](https://www.python.org/downloads/))
- Git ([Download](https://git-scm.com/download/win))

### Installation

```bash
# Clone the repository
git clone https://github.com/Ananyanagaraj11/soc-lite-ai-ids.git
cd soc-lite-ai-ids

# Install dependencies
pip install -r requirements.txt
```

### Train Model

```bash
# Train on CICIDS2017 dataset
python training/train.py --data data/CICIDS2017 --use-cicids2017-dir --max-rows-total 50000

# Copy artifacts to backend
scripts\copy_artifacts.bat  # Windows
# or
cp training/outputs/*.pt training/outputs/*.joblib training/outputs/*.txt backend/artifacts/  # Linux/Mac
```

### Run Locally

```bash
# Start backend (API + Dashboard)
python -m uvicorn backend.app:app --host 0.0.0.0 --port 8000

# Open in browser: http://localhost:8000
# Try sample CSV: data/sample_data.csv
```

**Flow:** Index → CSV Analysis → Upload CSV → Analyze File → Dashboard → View Results

## 📊 Features

### Dashboard Components

- **KPIs:** Total Events, Attack Events, Attack Percentage, Threat Level
- **Attack Distribution:** Donut chart showing class breakdown
- **Top Attack Types:** Horizontal bar chart of most frequent attacks
- **Severity Over Time:** Line chart tracking attack severity scores
- **Events Timeline:** Cumulative counts (Total, Attacks, Benign)
- **Feature Importance:** Bar chart of top contributing features
- **Recent Alerts:** Real-time alert feed with confidence scores
- **CSV Results Table:** Detailed predictions with actual vs predicted labels

### API Endpoints

- `GET /health` - Backend health check
- `GET /config` - Model configuration (input_dim, class_names)
- `POST /predict` - Single prediction from feature vector
- `POST /predict/explain` - Prediction with feature importance
- `POST /analyze/csv` - Batch CSV analysis (full dataset)
- `GET /api/last-analysis` - Retrieve last CSV analysis results

## 🗂️ Project Structure

```
soc-lite-ai-ids/
├── backend/
│   ├── app.py              # FastAPI server (API + static dashboard)
│   ├── explain.py          # Feature importance explanations
│   └── artifacts/          # Model files (model.pt, scaler.joblib, classes.pt)
├── dashboard/
│   ├── index.html          # Landing page
│   ├── analysis.html       # CSV upload & analysis
│   ├── dashboard.html      # Main dashboard
│   ├── js/
│   │   ├── dashboard.js    # Dashboard logic
│   │   ├── upload.js       # CSV upload handler
│   │   └── config.js       # API configuration
│   └── css/
│       └── dashboard.css   # Styles
├── training/
│   ├── dataset_loader.py   # CSV loading, preprocessing, split
│   ├── train.py            # PyTorch MLP training
│   └── evaluate.py         # Model evaluation & metrics
├── scripts/
│   ├── generate_sample_data.py  # Create data/sample_data.csv (3k rows)
│   ├── run_backend.bat     # Start backend
│   ├── copy_artifacts.bat  # Copy model to backend
│   └── git_setup.bat       # Initialize git repo
├── requirements.txt        # Python dependencies
├── vercel.json             # Vercel deployment config
├── render.yaml             # Render deployment config
└── README.md
```

## 🧠 Machine Learning

- **Model:** Multi-layer Perceptron (MLP) with BatchNorm and Dropout
- **Architecture:** Input → [128, 64] hidden layers → Output (num_classes)
- **Training:** CrossEntropyLoss with class weights, Adam optimizer, 50 epochs
- **Datasets:** CICIDS2017, UNSW-NB15
- **Preprocessing:** StandardScaler, label encoding, class weight balancing

## 🌐 Deployment

### Option 1: Vercel (Frontend) + Render (Backend)

1. **Push to GitHub:**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/Ananyanagaraj11/soc-lite-ai-ids.git
   git push -u origin main
   ```

2. **Deploy Backend (Render):**
   - Go to [render.com](https://render.com) → New Web Service
   - Connect GitHub repo → Uses `render.yaml` automatically
   - Copy backend URL: `https://your-backend.onrender.com`

3. **Deploy Frontend (Vercel):**
   - Go to [vercel.com](https://vercel.com) → Import GitHub repo
   - Uses `vercel.json` automatically
   - Update `dashboard/js/config.js` with your Render backend URL
   - Push → Auto-redeploys

**Live Demo:** [https://ai-cyber-threat-dashboard-1.onrender.com/](https://ai-cyber-threat-dashboard-1.onrender.com/)

See **[DEPLOY.md](DEPLOY.md)** for detailed steps.

### Option 2: Streamlit Cloud

Deploy `dashboard/streamlit_app.py` to [share.streamlit.io](https://share.streamlit.io) for a simpler one-service deployment.

## 📸 Screenshots

- **Landing Page:** Animated cyber-themed intro with navigation
- **CSV Analysis:** Upload, analyze, view summary and detailed results
- **Dashboard:** Real-time KPIs, charts, alerts, and threat visualization

## 🛠️ Technology Stack

| Component | Technology |
|-----------|-----------|
| **ML Framework** | PyTorch |
| **API** | FastAPI, Uvicorn |
| **Frontend** | HTML5, CSS3, JavaScript, Plotly.js |
| **Data Processing** | pandas, numpy, scikit-learn |
| **Deployment** | Vercel, Render, Streamlit Cloud |

## 📚 Datasets

- **[CICIDS2017](https://www.unb.ca/cic/datasets/ids-2017.html)** - Canadian Institute for Cybersecurity
- **[UNSW-NB15](https://research.unsw.edu.au/projects/unsw-nb15-dataset)** - UNSW Canberra

## 🤝 Contributing

Contributions welcome! Please open an issue or submit a pull request.

## 📄 License

MIT License - feel free to use for research, learning, or internal SOC tooling.

**Note:** Dataset licenses (CICIDS2017, UNSW-NB15) apply to the data files themselves.

## 🔗 Links

- **Live Demo:** [https://ai-cyber-threat-dashboard-1.onrender.com/](https://ai-cyber-threat-dashboard-1.onrender.com/)
- **Sample CSV:** [`data/sample_data.csv`](data/sample_data.csv) (3,000 rows)
- **Documentation:** See `DEPLOY.md`, `GITHUB_SETUP.md`, `TRAINING_AND_CSV_CHECK.md`
- **Issues:** [GitHub Issues](https://github.com/Ananyanagaraj11/soc-lite-ai-ids/issues)

---

⭐ **Star this repo if you find it useful!**
