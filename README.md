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
