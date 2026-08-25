# Better On Time Than Late: E-Commerce Delivery Delay Analysis & Prediction

> Understanding the impact of delivery delays on customer satisfaction and retention — and predicting them before they happen.

---

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Guiding Question](#guiding-question)
- [Use Case](#use-case)
- [Data](#data)
- [Methodology](#methodology)
  - [1. Data Preparation Pipeline](#1-data-preparation-pipeline)
  - [2. Feature Engineering](#2-feature-engineering)
  - [3. Statistical Analysis](#3-statistical-analysis)
  - [4. Predictive Modeling](#4-predictive-modeling)
- [Key Business Insights](#key-business-insights)
- [Model Results](#model-results)
- [Selected Model](#selected-model)
- [Dashboard](#dashboard)
- [What's Next](#whats-next)
- [Project Links](#project-links)
- [Repository Structure](#repository-structure)
- [Team](#team)

---

## Overview

In the modern e-commerce landscape, customers form strong expectations around product quality, customer service, and delivery timelines. This project investigates the relationship between **delivery performance**, **customer satisfaction**, and **repeat purchase behavior**, then builds a machine learning pipeline to **predict late deliveries before they happen** — turning delay *prediction* into delay *prevention*.

## Problem Statement

- Online shopping has become the norm, and the e-commerce industry is fiercely competitive, with countless platforms vying for the same customers.
- Customers have developed clear expectations for their purchase experience: product quality, responsive customer service, and delivery arriving on the promised date.
- Understanding the relationship between **customer satisfaction**, **repeat purchase behavior**, and **order delivery performance** is critical for e-commerce companies trying to retain customers and stay competitive.

## Guiding Question

> E-commerce companies compete on price, selection, and speed — but speed is often the most fragile of the three. A single missed delivery window can undo the value of a great product at a great price.

**How do delivery delays affect customer satisfaction and retention, and can we predict which orders are at risk of being late before they happen?**

## Use Case

**Delivery Delay Analysis** — understanding the impact of delivery delays and predicting them before they happen, so that platforms can intervene proactively rather than reactively.

## Data

The analysis draws on a relational e-commerce order dataset spanning five source tables:

| Table | Description |
|---|---|
| **Orders** | Order-level timestamps (purchase, approval, shipping, delivery) |
| **Order Items** | Line-item detail per order (price, freight value, product) |
| **Products** | Product attributes (category, weight, dimensions) |
| **Customers** | Customer and location identifiers |
| **Sellers** | Seller and location identifiers |

> **Note:** The table structure (Orders / Order Items / Products / Customers / Sellers) matches the shape of commonly used public multi-table e-commerce datasets. Confirm and link the exact data source here before publishing.

After cleaning and preparation, the final analytical dataset contains **96,454 order-level observations**.

## Methodology

### 1. Data Preparation Pipeline

Raw multi-table data was consolidated into a single order-level dataset through a five-step pipeline:

1. **Retained completed deliveries** — filtered to orders that were actually delivered
2. **Converted timestamps into delivery intervals** — turned raw datetime fields into measurable time deltas
3. **Aggregated items to one row per order** — collapsed multi-line orders to the order grain
4. **Combined product & geographic information** — joined in product and seller/customer location attributes
5. **Removed missing inputs & prevented leakage** — dropped incomplete records and excluded any fields that would leak the outcome

### 2. Feature Engineering

Two parallel feature sets were built from the cleaned data:

**Original Feature Set** (12 features)
- Estimated delivery window
- Purchase month, weekday & hour
- Order & freight value
- Item, product & seller counts
- Average weight & volume
- Same-state share (buyer/seller in the same state)

**Transformed Feature Set** (skewness-adjusted)
- Log-transformed: order value, freight value, product weight, product volume, delivery days
- Binary flags: multiple items, multiple products, multiple sellers

**Target variable:** `late_order` — `0` = on time / early, `1` = late

Both feature sets went through an **80/20 stratified train/test split**, class balancing, and scaling (for Logistic Regression and the Neural Network).

### 3. Statistical Analysis

Two hypothesis tests were run to validate the business relationship between delays and customer behavior before moving to prediction:

- **T-test (delays vs. review scores):** Compared average review score between on-time and delayed orders, and re-confirmed the result by grouping orders by review score (high vs. low) and comparing delay rates. Both directions showed a statistically significant difference — **delivery delays significantly reduce customer satisfaction.**
- **Chi-square test (delays vs. repeat purchase):** Repeat purchase is a binary (yes/no) outcome, so a chi-square test was used to compare repeat purchase rates between on-time and delayed orders. The relationship was statistically significant — **delivery delays significantly reduce customer retention.**

### 4. Predictive Modeling

Three model families were trained to predict `late_order`, each tested on both the original and transformed feature sets where applicable:

| Model | Notes |
|---|---|
| **Logistic Regression** | Interpretable linear baseline; tested on original & transformed features |
| **Random Forest** | Captures nonlinear relationships; tested on original & transformed features |
| **Neural Network** | Two hidden layers; trained on original standardized features with a balanced training sample |

## Key Business Insights

- Only **8.1%** of orders arrive delayed — but that small segment has an outsized effect on the business:
  - **Average review scores drop by 40%** for delayed orders
  - **Repeat purchase rate drops by 16%** for delayed orders
- Both the t-test and chi-square test confirm these relationships are statistically significant, not noise.

## Model Results

| Model | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| Original Logistic Regression | 0.581 | 0.118 | 0.644 | 0.199 |
| Transformed Logistic Regression | 0.588 | 0.119 | 0.640 | 0.201 |
| **Original Random Forest** | **0.792** | **0.202** | 0.528 | **0.292** |
| Transformed Random Forest | 0.791 | 0.201 | 0.533 | 0.292 |
| Neural Network | 0.683 | 0.154 | **0.650** | 0.249 |

**Key findings:**
- **Best precision:** Original Random Forest (0.202)
- **Best recall:** Neural Network (0.650)
- **Best F1:** Original & Transformed Random Forest (tied at 0.292)
- Feature transformation (log-scaling, skew correction) changed performance by **less than one percentage point** across models — the extra preprocessing did not meaningfully help.

## Selected Model

**Original Random Forest** — chosen because it matched the Transformed Random Forest's F1 performance while using the **simpler, more interpretable original feature set**, avoiding unnecessary preprocessing complexity for comparable results.

## Dashboard

A dashboard was built to demo the delay-risk model and surface delivery performance metrics interactively. *(See [Project Links](#project-links) for the live notebook/demo.)*

## What's Next

**Turning delay prediction into delay prevention:**

- **Seller & Logistics Partner Accountability**
  - Rank sellers and logistics partners by late-order rate
  - Flag chronic offenders for review
- **Real-Time Triage at Shipment**
  - Route the delay-risk model into live operations
  - Auto-flag high-risk orders for expedited handling

**Turning delay's association with dissatisfaction into good customer service:**

- **Platform Voucher for Next Order**
  - If an order is delayed, offer a small discount on the customer's next order
  - Open question: what's the optimal voucher amount that maximizes retention while minimizing platform expense?

## Project Links

1. **Databricks Notebook:** [View Notebook](https://dbc-9be97a69-c520.cloud.databricks.com/editor/notebooks/530294719074615?o=7474652451077049#command/8078495091479907)
2. **GitHub Repository:** _add link here_

## Repository Structure

> Suggested structure — update to match the actual repo layout.

```
.
├── data/                   # Raw and processed datasets (or data-loading scripts)
├── notebooks/              # EDA, statistical tests, and modeling notebooks
├── src/
│   ├── preprocessing.py    # Cleaning & feature engineering pipeline
│   ├── modeling.py         # Model training & evaluation
│   └── dashboard/          # Dashboard app code
├── reports/                # Slides, writeups, figures
├── requirements.txt
└── README.md
```

## Team

Group 3

---

*"Better late than never... but not really. Better on time than late."*
