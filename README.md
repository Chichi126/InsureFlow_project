
# InsureFlow Data Platform

---

## Overview

InsureFlow is an end-to-end data platform built for a fictional UK insurance company offering Health, Life, Property, and Auto coverage.

The focus of this project was not just building pipelines, but designing a reliable, scalable data platform that transforms messy, fragmented data into trusted business intelligence.

Built on Azure + Databricks using a Medallion Architecture, the platform handles ingestion, transformation, governance, and analytics seamlessly.

---

## The Problem
Before this platform:

- No single source of truth  
- Data scattered across systems  
- Heavy reliance on spreadsheets  
- Inconsistent reporting  

Simple business questions were hard to answer:

- What is our **claims ratio this quarter**?  

This project answers that.


## 🏗️  Architecture

![](Insuflux-Page.jpg)


| Layer | Tool | Purpose |
|---|---|---|
| Staging | ADLS Gen2 | Raw parquet files, dated folders |
| Bronze | Delta Lake | Raw ingestion, append only, no changes |
| Silver | Delta Lake | Cleaned, masked, SCD Type 2 |
| Gold | Delta Lake | Star schema and KPI tables |

**Tech Stack:** 

`Azure Data Factory · Databricks · Delta Lake · Unity Catalog · PySpark · ADLS Gen2`

---

## Dataset

| Table ||
|---|---|
| customers ||
| policies ||
| claims ||
| payments ||
| agents ||
| coverage ||
| risk_assessment ||


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

### What Each Layer Does

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
- New columns from source are automatically handled  in the Bronze layer
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
```
---

## Best Practices

- No hardcoded secrets — all credentials in Azure Key Vault, accessed via service principal OAuth
- Unity Catalog for access control across all Delta tables
- One notebook per table in silver — easy to debug, rerun and explain
- ForEach loop in ADF handles all seven tables dynamically — no copy-paste pipelines
- Storage event trigger fires only when files arrive — no wasted compute on empty runs
- Append only bronze means full audit trail and safe reprocessing at any time

---

## Cost Awareness

- Parquet in staging cuts file size significantly versus CSV
- Incremental loading keeps computation proportional to new data not total data
- Job clusters used for production runs — much cheaper than keeping interactive clusters alive
- Dated folder partitioning in ADLS enables lifecycle policies to archive old staging files automatically
- Gold aggregations are lightweight rebuilds — overwrite is fast and avoids stale data accumulating

---

## Outcomes

Four business KPIs now served reliably:

- Claims ratio by insurance type
- Revenue and commission per agent
- Customer risk distribution for underwriting
- Policy lapse rate trends for retention strategy

---

## Project Structure

```
InsureFlow/
├── 01_bronze/
├── 02_silver/         ← one notebook per table
├── 03_gold/
├── 04_orchestration/
└── README.md
```

