# MarketPulse ML

> Production-grade real-time ML platform for market forecasting, drift detection, and automated retraining.

[![Python](https://img.shields.io/badge/Python-3.12-blue)](https://python.org)
[![Kafka](https://img.shields.io/badge/Streaming-Confluent_Kafka-orange)](https://confluent.io)
[![MLflow](https://img.shields.io/badge/Tracking-MLflow-blue)](https://mlflow.org)
[![Evidently](https://img.shields.io/badge/Drift-Evidently_AI-purple)](https://evidentlyai.com)
[![Prefect](https://img.shields.io/badge/Orchestration-Prefect-teal)](https://prefect.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## What is this?

MarketPulse ML is an end-to-end MLOps system that does the following:

1. **Ingests** live financial market data (OHLCV) from Yahoo Finance
2. **Streams** it through Kafka topics in real time
3. **Stores** it in a partitioned Postgres database on Neon
4. **Engineers features** including RSI, MACD, Bollinger Bands, and rolling returns
5. **Trains** an XGBoost classifier to predict next-day price direction (up or down)
6. **Tracks** every experiment in MLflow running on Databricks
7. **Detects drift** using Evidently AI and publishes alerts to Kafka
8. **Retrains** the model automatically with champion-challenger gating
9. **Serves** predictions via a FastAPI REST API
10. **Visualizes** everything in a Streamlit monitoring dashboard
11. **Deploys** to Kubernetes with horizontal pod autoscaling

---

## Architecture

```
yfinance API
     |
     v
+-------------+    Kafka: market-raw     +-------------+
|  Ingestion  | ------------------------>|     ETL     |
|  (Phase 1)  |                          |  (Phase 1)  |
+-------------+                          +------+------+
                                                |
                                                v
                                        +-------------+
                                        |    Neon     |
                                        |  Postgres   |
                                        | (Partitioned)|
                                        +------+------+
                                                |
                                                v
                                        +-------------+    MLflow on
                                        |  Features   |    Databricks
                                        |  Training   |<------------>
                                        |  (Phase 2)  |
                                        +------+------+
                                                |
                                                v
                                        +-------------+
                                        |    Drift    |
                                        |  Detection  |
                                        |  (Phase 3)  |
                                        +------+------+
                                                |
                              Kafka: drift-alerts
                                                |
                                                v
                                        +-------------+
                                        |   FastAPI   |
                                        |  Dashboard  |
                                        |  (Phase 3)  |
                                        +-------------+
```

All flows are orchestrated by Prefect Cloud and deployed on Kubernetes in Phase 4.

---

## Tech Stack

| Layer | Tool | Why |
|---|---|---|
| Data source | yfinance | Free real financial OHLCV data, no API key needed |
| Streaming | Confluent Kafka | Industry standard at Netflix, Uber, LinkedIn |
| Orchestration | Prefect Cloud | Modern Airflow alternative, Python-native |
| Database | Neon Postgres | Serverless, time-partitioned, zero local setup |
| ML model | XGBoost | Industry standard for tabular data |
| Experiment tracking | MLflow on Databricks | Number 1 platform on MLE job descriptions |
| Drift detection | Evidently AI | Purpose-built, generates visual HTML reports |
| API | FastAPI | Replaced Flask in most modern ML stacks |
| Dashboard | Streamlit | MLE-native, no frontend experience needed |
| Containers | Docker | Phase 3 and beyond, for app containers only |
| Kubernetes | Minikube to EKS | Phase 4 deployment with autoscaling |
| CI/CD | GitHub Actions | Lint, test, and build on every push |
| Cloud | AWS S3 and ECR | Phase 4 artifact storage and container registry |

Total cost: $0 for Phases 1 through 3, and about $8/mo for Phase 4.

---

## Project Structure

```
mlops-real-time-drift-detection/
|
+-- ingestion/          # Pulls data from yfinance and sends to Kafka
+-- etl/                # Kafka consumer, cleans data, writes to Postgres
+-- db/
|   +-- migrations/     # SQL files that create and alter database tables
+-- features/           # Computes RSI, MACD, Bollinger Bands, rolling returns
+-- training/           # XGBoost training and MLflow experiment logging
+-- inference/          # Loads champion model and writes predictions to DB
+-- drift/              # Evidently AI drift reports and Kafka alert producer
+-- api/                # FastAPI endpoints: /predict /drift/status /health
+-- dashboard/          # Streamlit UI: live charts and model monitoring
+-- orchestration/      # Prefect flows with schedules and retry policies
+-- tests/
|   +-- unit/           # Tests for individual functions in isolation
|   +-- integration/    # Tests for components working together end-to-end
+-- docs/               # Architecture diagrams, data dictionary, runbook
+-- .github/
    +-- workflows/      # GitHub Actions CI/CD pipeline definitions
```

---

## Phase Roadmap

### Phase 1: MVP Pipeline (Week 1 to 2, $0)

Goal: data flows from source to database. You can demo this after two weeks.

- [x] Repo scaffold, branch strategy, conventional commits
- [x] Conda environment locked for Python 3.12 on arm64
- [ ] Neon Postgres account, partitioned schema, and migrations
- [ ] Confluent Kafka cluster with 3 topics
- [ ] Prefect Cloud account and flow scaffold
- [ ] yfinance to Kafka producer, idempotent and retryable
- [ ] Kafka consumer to ETL to Neon curated table
- [ ] Prefect flow scheduling ingest and ETL
- [ ] Streamlit dashboard v1: ingestion status and records chart

### Phase 2: ML Baseline (Week 3 to 4, $0)

Goal: a model exists, makes predictions, and everything is tracked.

- [ ] Databricks Community Edition and MLflow setup
- [ ] Feature engineering table: RSI, MACD, Bollinger Bands, rolling returns
- [ ] XGBoost classifier with rolling train and eval split
- [ ] All runs logged to Databricks MLflow with params, metrics, artifacts
- [ ] Model Registry with Staging to Production promotion using AUC gate
- [ ] Batch inference job writing predictions to Neon
- [ ] Prefect flow for features, training, and inference running nightly
- [ ] Dashboard v2: prediction chart and model performance metrics

### Phase 3: Drift and Retraining (Week 5 to 6, $0)

Goal: the system monitors itself and retrains when needed. This is the core MLOps story.

- [ ] Evidently AI drift reports: PSI per feature and data quality checks
- [ ] Drift events published to Kafka drift-alerts topic
- [ ] Retrain flow triggered by consuming drift alerts from Kafka
- [ ] Champion-challenger gating: promote only if AUC improves by 0.01 or more
- [ ] FastAPI endpoints: /predict/{ticker}, /drift/status, /model/current, /health
- [ ] Docker container for the serving layer
- [ ] Dashboard v3: drift monitor and model version history

### Phase 4: Production Polish (Week 7 to 8, about $8/mo)

Goal: deployable, tested, documented, and portfolio-ready.

- [ ] Kubernetes manifests: Deployment, Service, and HPA for autoscaling
- [ ] GitHub Actions pipeline: lint, test, Docker build, push to ECR, deploy to K8s
- [ ] AWS S3 for model artifacts and Evidently HTML reports
- [ ] Full unit and integration test suite with coverage badge
- [ ] Architecture diagram, model card, and runbook in docs/

---

## Getting Started

### Prerequisites

- Python 3.12
- Conda (miniforge recommended)
- Git

### Cloud accounts (all free for Phases 1 through 3)

| Service | Purpose | Link |
|---|---|---|
| Neon.tech | Cloud Postgres database | [neon.tech](https://neon.tech) |
| Confluent Cloud | Kafka streaming | [confluent.io/get-started](https://confluent.io/get-started) |
| Databricks CE | MLflow experiment tracking | [community.cloud.databricks.com](https://community.cloud.databricks.com) |
| Prefect Cloud | Flow orchestration and scheduling | [app.prefect.io](https://app.prefect.io) |
| AWS | Phase 4 only, S3 and ECR | [aws.amazon.com](https://aws.amazon.com) |

### Installation

```bash
# 1. Clone the repo
git clone git@github.com:hoomanesteki/mlops-real-time-drift-detection.git
cd mlops-real-time-drift-detection

# 2. Create the conda environment (takes 3 to 5 minutes)
conda env create -f environment.yml

# 3. Activate it
conda activate marketpulse

# 4. Copy and fill in your credentials
cp .env.example .env
# Open .env and add your Neon, Confluent, and Databricks values

# 5. Verify everything installed correctly
python -c "import yfinance, confluent_kafka, prefect, mlflow, ta, streamlit; print('All good')"
```

### Environment variables

Copy `.env.example` to `.env` and fill in your values:

```env
# Neon Postgres
DATABASE_URL=postgresql://user:password@host/marketpulse?sslmode=require

# Confluent Kafka
KAFKA_BOOTSTRAP_SERVERS=your-cluster.confluent.cloud:9092
KAFKA_API_KEY=your_key
KAFKA_API_SECRET=your_secret

# Databricks Community Edition
DATABRICKS_HOST=https://community.cloud.databricks.com
DATABRICKS_TOKEN=your_token
MLFLOW_TRACKING_URI=databricks

# AWS (Phase 4 only, leave blank for now)
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=us-east-1
S3_BUCKET=marketpulse-artifacts
```

Never commit your `.env` file. It is already listed in `.gitignore`.

---

## Key Concepts

### Why Kafka instead of writing directly to the database?

Instead of the ingestion script writing directly to Postgres, it publishes to a Kafka topic. The ETL service then consumes from that topic separately. This decouples ingestion from storage. If the database goes down temporarily, messages queue in Kafka and get processed when it recovers. No data is lost.

### Why partition the Postgres table?

The `raw_ohlcv` table is partitioned by year, for example `raw_ohlcv_2024` and `raw_ohlcv_2025`. When you query data from 2024, Postgres only scans that one partition instead of the entire table. This is how production systems handle large time-series data efficiently.

### Why MLflow on Databricks?

Every training run logs its parameters, metrics, and the model artifact. Six months from now you can look back and see exactly which hyperparameters produced which AUC on which dataset slice. This is what separates ML engineering from notebook science.

### What is drift?

A model trained on 2024 data learns patterns from 2024 market conditions. When conditions shift in 2025 (a crash, rate hikes, a new sector leading), the input distribution changes. Evidently AI measures this shift using PSI (Population Stability Index). When PSI exceeds a threshold, we automatically trigger retraining.

### What is champion-challenger gating?

Before promoting a newly retrained model to production, we compare it against the current champion on the same holdout dataset. If the new model's AUC does not beat the champion by at least 0.01, we do not promote it. This prevents a degraded model from ever reaching production.

---

## Branching Strategy

```
main          <- production-ready only, never commit directly here
  |
  +-- develop     <- integration branch, all features merge here first
        |
        +-- feat/ingestion-pipeline
        +-- feat/feature-engineering
        +-- feat/drift-detection
```

Commit message format using Conventional Commits:

```
feat: add yfinance kafka producer
fix: handle missing OHLCV values in ETL
chore: update environment pins for arm64
docs: add architecture diagram
test: add unit tests for feature pipeline
```

---

## Author

Hooman Esteki, Master of Data Science, ML Engineer

---

## License

MIT, see [LICENSE](LICENSE)
