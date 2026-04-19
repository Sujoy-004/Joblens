# JobLens — Job Market Insights & Salary Prediction System

JobLens is a static, notebook-driven capstone project that turns the public Data Science Salaries dataset into a salary prediction and job-market dashboard. The repository root is the source of truth for the current layout and deployment setup.

## Repository Layout

```
JobLens/
├── data/
│   ├── ds_salaries.csv
│   └── cleaned_data.csv
├── docs/
│   ├── documentation.md
│   └── JobLens_Documentation.pdf
├── frontend/
│   ├── index.html
│   └── dashboard_data.json
├── graphify-out/
│   └── graph.json
├── notebook/
│   └── job_market_pipeline.ipynb
├── outputs/
│   ├── clusters.csv
│   ├── dashboard_data.json
│   ├── GENERATED_BY_NOTEBOOK.txt
│   └── predictions.csv
├── README.md
└── vercel.json
```

## What’s in the repo

The notebook in [notebook/job_market_pipeline.ipynb](notebook/job_market_pipeline.ipynb) handles the full pipeline: cleaning the raw CSV, engineering features, training the regression and clustering models, and exporting the dashboard files. The frontend in [frontend/index.html](frontend/index.html) is a zero-backend static dashboard that reads [frontend/dashboard_data.json](frontend/dashboard_data.json) through a relative fetch.

The docs folder contains the written submission materials. The generated outputs folder keeps model artifacts and exported dashboard data that can be regenerated from the notebook.

## Dataset

Source dataset: [Data Science Salaries 2023 — Kaggle](https://www.kaggle.com/datasets/arnabchaki/data-science-salaries-2023)

Primary input file: [data/ds_salaries.csv](data/ds_salaries.csv)

The cleaned pipeline output is written to [data/cleaned_data.csv](data/cleaned_data.csv). The current exported dashboard payload describes 2,536 cleaned roles.

## Local Setup

The repo has no heavy framework setup. A minimal Python environment with notebook support is enough.

```bash
pip install pandas numpy scikit-learn jupyter
```

Run the notebook from the repository root:

```bash
jupyter notebook notebook/job_market_pipeline.ipynb
```

Run all cells top to bottom to regenerate the CSV and JSON outputs in [data](data), [outputs](outputs), and [frontend](frontend).

## Deployment

This project is deployed as a static site with Vercel. The active routing in [vercel.json](vercel.json) rewrites `/` to [frontend/index.html](frontend/index.html) and `/dashboard_data.json` to [frontend/dashboard_data.json](frontend/dashboard_data.json).

Live deployment: [joblens-capstone-kiit0001.vercel.app](https://joblens-capstone-kiit0001.vercel.app)

For local testing, open [frontend/index.html](frontend/index.html) directly in a browser. Keep [frontend/index.html](frontend/index.html) and [frontend/dashboard_data.json](frontend/dashboard_data.json) together, because the page fetches the JSON with a relative path.

## Models

| Model | Purpose | Notes |
|---|---|---|
| Gradient Boosting Regressor | Salary prediction | Trained on encoded job and company features |
| K-Means (k=5) | Job clustering | Used to group similar roles and drive the cluster view |

## Features

- Salary predictor with input validation for supported job titles.
- Cluster viewer powered by precomputed PCA coordinates.
- Market charts for salary by experience and top job-title volume.
- Similar-jobs panel for contextual comparisons after a prediction.
- Static frontend with no backend dependency.

## Notes

- [docs/documentation.md](docs/documentation.md) and the notebook are the main human-readable project references.
- [outputs/dashboard_data.json](outputs/dashboard_data.json) and [frontend/dashboard_data.json](frontend/dashboard_data.json) are generated artifacts, not hand-edited source files.
- [graphify-out/graph.json](graphify-out/graph.json) is an auxiliary export used for graph-style inspection.

## Author

Capstone submission by B.Tech CSE
