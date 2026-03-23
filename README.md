# MLOps Pipeline — CI/CD, Champion/Challenger & Drift Monitoring

## Overview
A production MLOps pipeline implementing the full model lifecycle: automated testing, CI/CD-triggered retraining, champion/challenger model validation, data drift monitoring, and automatic rollback. The infrastructure that turns ML experiments into reliable production systems.

## Architecture
- **CI/CD Pipeline**: GitHub Actions workflow triggered on changes to training code
  1. Run unit tests (data validation, feature computation, model interface)
  2. Execute training pipeline on test data
  3. Validate new model meets minimum performance thresholds
  4. Champion/Challenger comparison: new model vs current production model on holdout set
  5. Auto-promote to MLflow Model Registry (staging → production) only if challenger wins
- **Drift Monitoring**: Evidently AI reports generated daily via Airflow
  - Feature drift (PSI, KS test per feature)
  - Prediction drift (distribution shift in model outputs)
  - Target drift (when ground truth becomes available)
  - Automated retraining trigger when drift exceeds thresholds
- **Rollback**: Automatic model rollback if production metrics degrade below SLA for 3 consecutive days
- **Monitoring Dashboard**: Grafana dashboards for model performance, drift metrics, and pipeline health

## Tech Stack
- Python 3.11+, pytest
- MLflow (model registry + model staging)
- Evidently AI (drift monitoring)
- GitHub Actions (CI/CD)
- Apache Airflow 2.x (orchestration + monitoring)
- Grafana (dashboards)
- Docker & Docker Compose

## What This Demonstrates
- Production ML lifecycle management
- Champion/Challenger model validation pattern
- Automated data drift detection and response
- CI/CD for ML code (not just application code)
- Model rollback and recovery strategies
- MLflow Model Registry staging workflow
- The full gap between notebook ML and production ML

## Status
🚧 In Development

## Project Structure
├── .github/workflows/
│   └── ml_pipeline.yml
├── dags/
│   ├── retraining_dag.py
│   └── drift_monitoring_dag.py
├── src/
│   ├── training/
│   ├── validation/
│   ├── monitoring/
│   └── rollback/
├── tests/
│   ├── test_features.py
│   ├── test_training.py
│   └── test_model_quality.py
├── grafana/
├── docker-compose.yml
└── README.md
