# 🚀 InsureFlow Data Platform

![Azure](https://img.shields.io/badge/Azure-Data%20Platform-blue?logo=microsoftazure)
![Databricks](https://img.shields.io/badge/Databricks-Delta%20Lake-red?logo=databricks)
![PySpark](https://img.shields.io/badge/PySpark-ETL-orange?logo=apachespark)
![ADF](https://img.shields.io/badge/Azure%20Data%20Factory-Orchestration-blue)
![Status](https://img.shields.io/badge/Project-Production--Ready-success)
![Architecture](https://img.shields.io/badge/Architecture-Medallion-purple)

---

## 📌 Overview

InsureFlow is an end-to-end data platform built for a fictional UK insurance company offering Health, Life, Property, and Auto coverage.

The focus of this project was not just building pipelines, but designing a **reliable, scalable data platform** that transforms messy, fragmented data into **trusted business intelligence**.

Built on Azure + Databricks using a **Medallion Architecture**, the platform handles ingestion, transformation, governance, and analytics seamlessly.

---

## ❗ The Problem

Before this platform:

- No single source of truth  
- Data scattered across systems  
- Heavy reliance on spreadsheets  
- Inconsistent reporting  

Simple business questions were hard to answer:

- What is our **claims ratio this quarter**?  
- Which agents are driving **the most revenue**?  

---

## 🏗️ Architecture

![Architecture Diagram](Insuflux-Page.jpg)

| Layer   | Tool            | Purpose |
|--------|-----------------|--------|
| Staging | ADLS Gen2       | Raw parquet files (date-partitioned) |
| Bronze  | Delta Lake      | Raw ingestion (append-only) |
| Silver  | Delta Lake      | Cleaned + SCD Type 2 |
| Gold    | Delta Lake      | Star schema + KPIs |

**Tech Stack:**  
`Azure Data Factory · Databricks · Delta Lake · Unity Catalog · PySpark · ADLS Gen2`

---

## 📊 Dataset

- customers  
- policies  
- claims  
- payments  
- agents  
- coverage  
- risk_assessment  

### ⚠️ Data Quality Challenges

- Missing values  
- Duplicates  
- Invalid dates  
- Negative values  
- Inconsistent casing  
- Hidden characters (`\r`)  
- Mixed data types  

---

## 🔄 Data Flow (Medallion Architecture)

### 🟡 Staging Layer
- ADF ingests data into ADLS (`yyyy/MM/dd`)
- No transformations — raw landing only

---

### 🟤 Bronze Layer (Delta Lake)

- Append-only ingestion  
- Full audit trail preserved  
- No updates or deletes  

✅ **Idempotent Design**
- Re-running ingestion does not duplicate or corrupt data  

✅ **Schema Evolution**
- New columns from source are automatically handled  
- Prevents pipeline failures from schema drift  

---

### ⚪ Silver Layer (Transformation Layer)

This is where the heavy lifting happens:

- Data cleaning (trim, casing, formatting)  
- Type casting with dirty data handling  
- Duplicate removal  
- Invalid record filtering  
- PII masking  

### 🔁 SCD Type 2 with Delta Lake

- Historical tracking of changes  
- `is_current` flag for latest records  
- `effective_start_date` / `effective_end_date`  

### ⚡ Delta MERGE for Upserts

- No full reloads  
- No duplicate records  
- Fully **idempotent transformations**

---

### 🟡 Gold Layer (Analytics Ready)

Two outputs:

#### ⭐ Star Schema
- `fact_claims`  
- `dim_customers`  
- `dim_agents`  
- `dim_policies`  
- `dim_date`  

#### 📈 KPI Tables
- Pre-aggregated metrics for dashboards  

✅ Clean  
✅ No PII  
✅ BI-ready  

---

## 🔧 Key Engineering Features

### 🔹 Incremental Processing
- Bronze → loads only new files  
- Silver → filters by ingestion date  
- Gold → rebuilds lightweight aggregations  

---

### 🔹 Schema Evolution (Controlled)

Handled explicitly using:

```python
.option("mergeSchema", "true")
