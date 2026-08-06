# Churn MLOps Pipeline — PySpark Feature Engineering, CI/CD & Drift Monitoring

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=flat&logo=apachespark&logoColor=white)
![AWS EMR](https://img.shields.io/badge/AWS%20EMR-FF9900?style=flat&logo=amazonaws&logoColor=white)
![MLflow](https://img.shields.io/badge/MLflow-0194E2?style=flat&logo=mlflow&logoColor=white)
![Evidently AI](https://img.shields.io/badge/Evidently%20AI-drift-blue?style=flat&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=flat&logo=githubactions&logoColor=white)
![Apache Airflow](https://img.shields.io/badge/Apache%20Airflow-017CEE?style=flat&logo=apacheairflow&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat&logo=grafana&logoColor=white)

## Overview

A production MLOps pipeline for a subscription company's churn model, where the reason this is a Data/ML Engineering problem and not just a notebook exercise is scale: retraining requires recomputing rolling engagement features across tens of millions of historical usage events, which is where PySpark stops being optional and becomes the only viable option. Around that Spark-based feature/scoring core sits the full production lifecycle — CI/CD-triggered retraining, champion/challenger validation, drift monitoring, and automatic rollback.

## The Problem

A subscription company's churn model needs to be retrained regularly as customer behavior shifts — a model trained on last quarter's usage patterns quietly gets worse as the product, pricing, or user base changes. This alone is a common MLOps problem. What makes it a genuine data engineering challenge here: churn features (30/60/90-day engagement trends, feature-usage rates, support-ticket frequency, rolling session counts) have to be recomputed across the company's **entire historical event log** every time the model retrains — tens of millions of rows — which pandas cannot do in a reasonable retraining window. This is the concrete reason feature computation and batch scoring in this project run on **PySpark**, not as a stylistic choice but as the only approach that keeps retraining fast enough to run on a schedule.

The second real problem: a model that quietly degrades is worse than one that fails loudly, because nobody notices until retention numbers are already down. The pipeline needs to detect *why* performance is dropping — is it that customer behavior itself changed (feature drift), or that the relationship between behavior and churn changed (concept/target drift) — and trigger retraining automatically rather than waiting for a human to notice a dashboard trending the wrong way.

## Data Source

A synthetic but realistically-scaled subscription usage event log (tens of millions of events: logins, feature usage, support tickets, billing events) generated with genuine engagement patterns and injected behavioral drift at specific points in time — so the drift-monitoring and auto-retrain trigger can be verified against a known, reproducible drift event, not just assumed to work.

## Architecture

- **Feature Engineering (PySpark, the core of the project):** Rolling-window engagement features computed across the full historical event log via PySpark jobs running on **AWS EMR**, writing versioned feature sets to S3 — the same scale problem the Feature Store project solves for point-in-time correctness, here solved for retraining throughput.
- **CI/CD Pipeline (GitHub Actions),** triggered on changes to training code:
  1. Run unit tests (data validation, feature computation logic, model interface)
  2. Trigger the PySpark feature-computation job on EMR against a test slice of data
  3. Run the training pipeline and validate the new model meets minimum performance thresholds
  4. **Champion/Challenger comparison** — new model vs. current production model, evaluated on the same PySpark-computed holdout set
  5. Auto-promote to the **MLflow Model Registry** (staging → production) only if the challenger wins
- **Batch Scoring:** Nightly PySpark job scoring the full active customer base — again a scale problem, not a pandas-sized one — with predictions written to the serving store.
- **Drift Monitoring:** Evidently AI reports generated daily via Airflow — feature drift (PSI, KS test), prediction drift, and target drift once ground truth is available — with an automated retraining trigger when drift crosses threshold, closing the loop instead of requiring a human to kick off retraining.
- **Rollback:** Automatic rollback to the previous production model if live metrics degrade below SLA for 3 consecutive days.
- **Monitoring Dashboard:** Grafana dashboards for model performance, drift metrics, EMR job cost/duration, and pipeline health.

## Tech Stack

- Python 3.11+, `pytest`
- **PySpark** (feature engineering + batch scoring at scale)
- **AWS EMR** (Spark execution)
- MLflow (model registry, staging → production promotion)
- Evidently AI (drift monitoring)
- GitHub Actions (CI/CD)
- Apache Airflow 2.x (orchestration, drift monitoring schedule)
- Grafana (dashboards)
- Docker & Docker Compose

## What This Demonstrates

- PySpark used because the problem genuinely requires it (tens of millions of rows, retraining-window time constraints) — not bolted on for the sake of using it
- Full production ML lifecycle: CI/CD for ML code, automated champion/challenger promotion, closed-loop drift-triggered retraining, and automatic rollback
- The real gap between notebook ML and production ML — most of this project's complexity is in the surrounding engineering, not the model itself
- Cost/performance awareness of running Spark on managed infrastructure (EMR), tracked alongside model metrics rather than treated as someone else's problem

## Status
🚧 In Development

## Project Structure
```
├── .github/workflows/
│   └── ml_pipeline.yml
├── spark_jobs/
│   ├── feature_engineering/
│   │   └── rolling_engagement_features.py
│   └── batch_scoring/
│       └── score_active_customers.py
├── dags/
│   ├── retraining_dag.py
│   └── drift_monitoring_dag.py
├── src/
│   ├── training/
│   ├── validation/
│   │   └── champion_challenger.py
│   ├── monitoring/
│   └── rollback/
├── tests/
│   ├── test_features.py
│   ├── test_training.py
│   └── test_model_quality.py
├── grafana/
├── docker-compose.yml
└── README.md
```
