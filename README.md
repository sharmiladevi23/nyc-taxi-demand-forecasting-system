# 🚖 NYC Taxi Demand Forecasting Platform (MLOps)

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![Hopsworks](https://img.shields.io/badge/Feature_Store-Hopsworks-green)
![LightGBM](https://img.shields.io/badge/Model-LightGBM-orange)
![Streamlit](https://img.shields.io/badge/Frontend-Streamlit-red)
![Status](https://img.shields.io/badge/Status-Production_Ready-success)

## 📋 Executive Summary
This project implements an end-to-end **MLOps system** to forecast real-time taxi demand in New York City. Unlike static historical analysis, this platform provides **next-hour predictions** at a granular zone level, enabling fleet operators to optimize driver allocation and reduce passenger wait times.

The system leverages a **Feature Store architecture** to prevent training-serving skew and automates the entire lifecycle from data ingestion to model deployment and monitoring.

---

## 🏗️ System Architecture

The platform is built on a modular microservices architecture, decoupled into three independent pipelines:

```mermaid
graph LR
    A[Raw Data Source] -->|Ingest| B(Feature Pipeline)
    B -->|Write Features| C{Hopsworks Feature Store}
    C -->|Read Batch| D(Training Pipeline)
    D -->|Register| E[Model Registry]
    C -->|Read Batch| F(Inference Pipeline)
    E -->|Load Model| F
    F -->|Write Predictions| C
    C -->|Fetch Predictions| G[Streamlit Dashboard]

## 🤖 Machine Learning Approach

### 1. Feature Engineering Strategy
The model avoids reliance on complex external data (like weather) by engineering robust **Time-Series Features** that capture inherent demand cycles:

* **Temporal Features:**
    * **`hour` & `day_of_week`**: Extracted to capture daily commute patterns (e.g., AM/PM rush hours) and weekly seasonality (e.g., Friday night vs. Monday morning).
* **Lag & Window Features:**
    * **`average_rides_last_4_weeks`**: A powerful rolling-window feature that calculates the average demand for the **same hour** on the **same day** over the last 28 days. This allows the model to "remember" monthly trends and strict recurring patterns.
    * *Implementation:* `src.pipeline_utils.average_rides_last_4_weeks`

### 2. Model Training & Governance
* **Algorithm:** **LightGBM Regressor**. Chosen for its high efficiency with tabular data and ability to handle large-scale time-series datasets with minimal latency.
* **Champion/Challenger Evaluation:**
    * The training pipeline implements an automated **Gating Mechanism**.
    * Every new model candidate is evaluated against the current **Production Model** using Mean Absolute Error (MAE).
    * **Promotion Logic:** `if candidate_mae < production_mae: register_model()`. This ensures zero regression in production performance.

### 3. Production Inference Workflow
To simulate real-time production constraints, the inference pipeline runs on a strictly decoupled schedule:
1.  **Lookback Context:** Fetches the last **28 days** of batch data from the Feature Store to compute necessary lag features.
2.  **Forward Prediction:** Generates demand forecasts for the **Next Hour** (T+1).
3.  **Write-Back:** Predictions are materialized back into the Feature Store (`model_prediction` feature group), making them instantly available for the Streamlit dashboard via low-latency lookup.
