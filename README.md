# Retail Demand Forecasting for Small Businesses
### SmartBiz AI — Capstone Two | Springboard Data Science Bootcamp

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![Dataset](https://img.shields.io/badge/Dataset-UCI%20Online%20Retail-green)](https://archive.ics.uci.edu/dataset/352/online+retail)

---

## Problem Statement

Small retail businesses make inventory decisions based on intuition rather than data. Overstocking ties up cash and creates waste; understocking results in lost sales and customer dissatisfaction. Large retailers like Amazon use sophisticated demand forecasting systems — but no affordable, accessible equivalent exists for small business owners.

**SmartBiz AI** addresses this gap by building a supervised regression model that predicts weekly product demand for small retail businesses, enabling owners to make data-driven inventory decisions without any data science knowledge.

---

## Project Overview

| | |
|---|---|
| **Type** | Supervised Regression |
| **Model** | XGBoost Regressor |
| **Dataset** | UCI Online Retail (541,909 transactions) |
| **Target** | Weekly units sold per product |
| **Evaluation** | MAE, RMSE, R² vs. naive baseline |
| **Output** | Streamlit dashboard + final report |

---

## Dataset

**UCI Online Retail Dataset**
- 541,909 transactions from a UK-based retailer
- Date range: December 2010 – December 2011
- ~4,300 unique customers | ~3,700 unique products
- Available at: [archive.ics.uci.edu](https://archive.ics.uci.edu/dataset/352/online+retail) or [Kaggle](https://www.kaggle.com/datasets/vijayuv/onlineretail)
- License: Creative Commons Attribution 4.0 (CC BY 4.0)

> **Note:** The dataset is not included in this repository due to its size. Download it from the links above and place it in the `data/raw/` folder before running any notebooks.

---

## Approach

### Step 1 — Data Wrangling & EDA
- Remove cancelled transactions (Quantity < 0) and rows with missing CustomerID
- Aggregate daily transactions into weekly sales per product (StockCode + week)
- Explore sales trends by product, day of week, month, and seasonal patterns

### Step 2 — Feature Engineering
- Lag features: units sold 1, 2, and 4 weeks prior
- Rolling average: 4-week moving average of sales per product
- Calendar features: day of week, month, quarter, UK public holiday flags
- Exclude products with fewer than 4 weeks of sales history

### Step 3 — Modeling
- Model: XGBoost Regressor (single well-tuned model)
- Split: time-based train/validation/test split (no random shuffle)
- Tuning: GridSearchCV with TimeSeriesSplit
- Post-processing: predictions rounded to nearest non-negative integer

### Step 4 — Evaluation
- MAE, RMSE, and R² against a naive baseline (per-product average weekly sales)
- Business framing: % of forecasts within ±20% of actual weekly sales per product

### Step 5 — Output
- Streamlit dashboard showing actual vs. predicted weekly demand per product
- Final report submitted as PDF to this repository

---

## Repository Structure

```
retail-demand-forecasting-capstone/
│
├── proposal/
│   └── SmartBizAI_Capstone2_Proposal.pdf   ← Project proposal (start here)
│
├── data/
│   └── raw/                                ← Place UCI dataset here (not tracked)
│
├── notebooks/
│   ├── 01_data_wrangling.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_feature_engineering.ipynb
│   └── 04_modeling.ipynb
│
├── dashboard/
│   └── app.py                              ← Streamlit dashboard (coming soon)
│
├── reports/
│   └── final_report.pdf                    ← Final report (coming soon)
│
├── .gitignore
├── LICENSE
└── README.md
```

> Notebooks and dashboard will be added as the project progresses.

---

## About SmartBiz AI

SmartBiz AI is a machine learning platform built specifically for small businesses. It translates raw transaction data into plain-English predictions — no data science background needed.

This capstone focuses on the **Retail Intelligence Module**: predicting weekly product demand using the UCI Online Retail dataset. Future modules planned include restaurant staffing prediction, customer churn detection, and peak period forecasting.

---

## Author

**Jacob Paul**
[LinkedIn](https://www.linkedin.com/in/jacob25paul) · [GitHub](https://github.com/jacob25paul)
