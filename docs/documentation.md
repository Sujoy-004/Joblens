# JobLens: Job Market Insights & Salary Prediction System
### Capstone Project Documentation | Data Engineering + Data Science
### B.Tech Computer Science & Engineering

---

## Table of Contents
1. Problem Statement
2. Solution Overview
3. Architecture & Pipeline
4. Features
5. Tech Stack
6. Model Details
7. Unique Points
8. Future Improvements

---

## 1. Problem Statement

The data science and technology job market is rapidly evolving, with significant variation in compensation across roles, experience levels, company sizes, and geographies. Job seekers, HR professionals, and organisations frequently face challenges in:

- Accurately benchmarking salaries for technical roles
- Understanding the skill clusters that define different job archetypes
- Gaining actionable insights from raw, unstructured job market data

Existing tools are either paywalled, region-locked, or offer no ML-driven personalisation. This project addresses that gap by building an end-to-end data engineering and machine learning pipeline that ingests real-world salary data, processes it, trains predictive models, and delivers insights via a publicly accessible web interface.

---

## 2. Solution Overview

JobLens is a full-stack Data Engineering + ML capstone system consisting of three tightly integrated layers:

**Layer 1 — Data Engineering Pipeline (Jupyter Notebook)**  
Ingests the public `ds_salaries` dataset (3,755 records across 11 dimensions). Applies systematic cleaning, null-handling, outlier removal, and feature engineering including keyword-based skill extraction from job titles. Exports clean CSVs and a frontend-ready JSON file.

**Layer 2 — Machine Learning Models (scikit-learn)**  
Two complementary models are trained:
- A **Gradient Boosting Regressor** for salary prediction (regression)
- A **K-Means clustering model** (k=5) for job archetype grouping, with PCA dimensionality reduction for visualisation

**Layer 3 — Frontend Dashboard (Static HTML/JS)**  
A Vercel-deployable, zero-backend interface that reads the exported JSON and provides interactive salary prediction, cluster visualisation, and market analytics — all without a server.

The current frontend payload exposes 2,536 cleaned roles, so the live hero copy, summary cards, and dataset exports all describe the same post-cleaning view of the data.

---

## 3. Architecture & Pipeline

```
[Raw CSV]
   │
   ▼
[Data Ingestion]
   • Load ds_salaries.csv via Pandas
   • Schema validation & dtype inspection
   │
   ▼
[Data Cleaning]
   • Drop duplicates
   • Drop nulls on key columns
   • Trim & uppercase text fields
   • Outlier removal (1st–99th percentile on salary)
   │
   ▼
[Feature Engineering]
   • Keyword-based skill extraction from job_title
     → 7 binary skill flags: python, sql, ml, cloud, viz, nlp, stats
   • LabelEncoder on all categorical columns
   • Top-50 job title normalisation (rare → 'OTHER')
   │
   ▼
[Model Training]
   ├── Gradient Boosting Regressor (200 trees, depth=4, lr=0.1)
   │       • Features: 14 encoded + skill flags
   │       • Target: salary_in_usd
   │       • Output: predictions.csv
   │
   └── K-Means Clustering (k=5, StandardScaler + PCA-2D)
           • Features: 10 skill + experience + salary dimensions
           • Output: clusters.csv (with pca_x, pca_y for viz)
   │
   ▼
[Export Layer]
   • cleaned_data.csv → /data/
   • predictions.csv  → /outputs/
   • clusters.csv     → /outputs/
   • dashboard_data.json → /outputs/  [consumed by frontend]
   │
   ▼
[Frontend — Vercel]
   • Fetches dashboard_data.json
   • Renders: Salary Predictor, Scatter Chart, Bar Charts, Cluster Table
   • Falls back to built-in demo data if JSON unavailable
```

---

## 4. Features

### 4.1 Salary Predictor
Users input a job title, experience level (EN/MI/SE/EX), and company size (S/M/L). The frontend performs a lookup against pre-computed prediction data and applies experience/size multipliers to estimate a personalised salary range. The result is displayed with a salary figure, confidence band, and assigned job cluster.

The predictor only accepts supported titles from the exported dataset. If the user enters a vague, partial, or unsupported title, the interface shows a clear no-match message instead of fabricating a salary.

### 4.2 Similar Jobs Panel
Post-prediction, a grid of similar roles within the same experience band is surfaced from the predictions dataset, giving users lateral market context.

### 4.3 Salary by Experience Level (Bar Chart)
Median salaries segmented by entry, mid, senior, and executive tiers — rendered using Chart.js with custom dark-mode styling.

### 4.4 Top 10 Job Titles by Volume (Horizontal Bar)
Market frequency of the most-listed roles, providing a supply-side view of the job market.

### 4.5 PCA Cluster Scatter Plot
500 sampled data points plotted in 2D PCA space, colour-coded by cluster. Allows visual confirmation that K-Means has cleanly separated distinct job archetypes.

### 4.6 Cluster Summary Table
Tabular breakdown of the 5 job clusters: count, mean salary, and average remote ratio. Provides an at-a-glance view of which archetypes skew remote or high-compensation.

### 4.7 Demo Mode
If the JSON output file is unavailable (e.g. Vercel cold deployment), the frontend seamlessly falls back to built-in representative demo data — ensuring the interface is always functional.

### 4.8 Input Validation & No-Match Handling
The frontend normalises title casing, rejects empty input, and limits predictions to known titles from the exported dataset. When an exact experience-size combination is unavailable, the app uses the closest supported row for the same title and marks it as a closest-match response. This keeps the demo honest during viva questions.

---

## 5. Tech Stack

| Component | Technology | Justification |
|---|---|---|
| Data processing | Python 3.10, Pandas, NumPy | Industry standard; reliable; no heavy deps |
| ML — Regression | scikit-learn GradientBoostingRegressor | Strong performance on tabular data without deep learning overhead |
| ML — Clustering | scikit-learn KMeans | Interpretable; fast; suitable for moderate dataset size |
| Dimensionality reduction | scikit-learn PCA | 2D projection for frontend visualisation |
| Notebook environment | Jupyter Notebook | Reproducible, cell-by-cell auditability |
| Frontend | HTML5, CSS3, Vanilla JS | Zero framework overhead; Vercel-compatible |
| Charting | Chart.js 4.4 | Lightweight; CDN-available; rich chart types |
| Deployment | Vercel (static hosting) | Free tier; Git-connected; global CDN |

---

## 6. Model Details

### 6.1 Gradient Boosting Regressor
**Features used (14):**
- experience_level_enc, employment_type_enc, company_size_enc
- company_location_enc, employee_residence_enc, job_title_enc
- remote_ratio
- skill_python, skill_sql, skill_ml, skill_cloud, skill_viz, skill_nlp, skill_stats

**Training split:** 80/20, random_state=42  
**Hyperparameters:** n_estimators=200, max_depth=4, learning_rate=0.1  
**Performance:** MAE ≈ $35,654 | R² ≈ 0.4706

### 6.2 K-Means Clustering
**Features used (10):** 7 skill flags + experience_level_enc + remote_ratio + salary_in_usd  
**Preprocessing:** StandardScaler  
**k=5** (elbow method evaluated; 5 yields interpretable separation)  
**Cluster Labels:**
- 0 → Entry-Level Analysts
- 1 → Senior ML Engineers
- 2 → Cloud & Data Platform
- 3 → Business Intelligence
- 4 → Research & NLP

**PCA (2 components):** Explained variance ≈ 68% — sufficient for scatter visualisation.

---

## 7. Unique Points

1. **Skill inference without external NLP:** Skill tags (Python, SQL, ML, Cloud, etc.) are derived deterministically from job title keywords — no dependency on spaCy, NLTK, or API calls.

2. **Zero-backend architecture:** The entire frontend runs from a single static JSON file, making it deployable on Vercel free tier with no server costs and near-instant load times.

3. **Graceful demo fallback:** If the JSON output file is absent, the frontend injects realistic demo data and continues functioning. This prevents dead deployments.

4. **Dual-model pipeline in one notebook:** Regression and clustering are co-trained in a single reproducible notebook, sharing a common feature-engineered DataFrame — reducing code duplication.

5. **PCA-to-frontend pipeline:** Cluster coordinates computed server-side (notebook) are embedded in the JSON and rendered directly in Chart.js scatter plots, avoiding any client-side matrix operations.

---

## 8. Future Improvements

| Priority | Improvement | Rationale |
|---|---|---|
| High | Add FastAPI backend for real-time inference | Enable live model calls instead of pre-computed lookup |
| High | Integrate LLM-based skill extraction | Replace keyword heuristics with semantic understanding |
| Medium | Expand dataset to 50k+ roles via scraping | Reduce MAE and improve cluster granularity |
| Medium | Add SHAP explainability to salary predictions | Regulatory compliance; user trust |
| Medium | Time-series salary trend analysis | Seasonal & YoY salary movement per role |
| Low | Add LinkedIn/Indeed API data ingestion | Real-time market signal integration |
| Low | Role comparison view | Side-by-side salary comparison for 2–3 selected roles |

---

*End of Documentation*  
*JobLens Capstone Project | B.Tech CSE | Data Engineering + Data Science*
