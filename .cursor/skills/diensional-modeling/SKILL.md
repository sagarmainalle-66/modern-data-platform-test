---
name: star-schema
description: Interactive step-by-step assistant for building dbt models, Star Schema layers, and YAML tests incrementally on BigQuery.
---

# Incremental Star Schema & dbt Model Playbook

When asked to build or expand dbt models, execute in **distinct, step-by-step iterations**. Never generate all layers at once unless explicitly told to do so. Pause after completing each step to allow manual review and verification.

---

## 📍 Step 1: Staging Layer (`models/staging/stg_<source>_<entity>.sql`)
* **Objective:** Clean and standardize raw BigQuery source data.
* **Workflow:**
  - Read from raw sources using `{{ source('source_name', 'table_name') }}`.
  - Standardize column names to `snake_case`.
  - Apply explicit type casting using BigQuery native functions (e.g., `safe_cast(col as string)`).
  - Primary key must be `<entity>_id`.
* **Action:** Generate ONLY the requested staging SQL model, then ask if you should proceed to the next step.

---

## 📍 Step 2: Dimension Models (`models/marts/dim_<entity>.sql`)
* **Objective:** Build entity attribute tables (e.g., customers, products).
* **Workflow:**
  - Reference staging models using `{{ ref('stg_...') }}`.
  - Primary key must be named `<entity>_id`.
  - Handle potential `NULL` string values using `coalesce(status, 'Unknown')`.
* **Action:** Generate ONLY the requested dimension SQL model, then pause for review.

---

## 📍 Step 3: Fact Models (`models/marts/fct_<entity>.sql`)
* **Objective:** Build transactional models with metrics and foreign keys.
* **Workflow:**
  - Primary key must be named `<entity>_id` (e.g., `order_id`).
  - Reference dimension or staging models using `{{ ref('...') }}`.
  - Include foreign keys that match dimension primary key names to enable clean natural joins.
* **Action:** Generate ONLY the requested fact SQL model, then pause for review.

---

## 📍 Step 4: Schema Documentation & Testing (`schema.yml`)
* **Objective:** Add data quality tests and model descriptions.
* **Workflow:**
  - Add `unique` and `not_null` tests to primary keys (`<entity>_id`).
  - Add `relationships` tests to foreign keys in fact tables pointing back to dimension tables.
* **Action:** Update or create the schema `.yml` file for the model created in the previous step.