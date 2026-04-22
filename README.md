# 🚲 Citi Bike Commuter Analysis: End-to-End Medallion Pipeline

[![Databricks](https://img.shields.io/badge/Platform-Databricks_Community-blue?logo=databricks)](https://community.cloud.databricks.com/)
[![PySpark](https://img.shields.io/badge/Language-PySpark-orange?logo=apachespark)](https://spark.apache.org/docs/latest/api/python/)
[![Delta Lake](https://img.shields.io/badge/Storage-Delta_Lake-00ADD8?logo=delta)](https://delta.io/)
[![SCD Type 2](https://img.shields.io/badge/Logic-SCD_Type_2-red)](#silver-layer-scd-type-2-dimension)

## 📌 Project Overview
This project implements a scalable **Medallion Architecture** to process NYC Citi Bike trip telemetry from 2024. The core objective is to differentiate between **Commuter** and **Leisure** rider behavior while maintaining a high-fidelity historical record of station metadata using advanced data engineering techniques.

### 🚀 Key Technical Highlights
* **Historical Integrity:** Implemented **Slowly Changing Dimensions (SCD) Type 2** to track station name and location changes over time.
* **Temporal Logic:** Resolved metadata join conflicts by establishing a `2023-12-31` baseline and utilizing `9999-12-31` as the active sentinel date.
* **Feature Engineering:** Developed a specialized 6-column Gold layer optimized for urban mobility analytics.
* **Orchestration:** Built a programmatic Master Pipeline to manage multi-notebook task dependencies in the Databricks environment.

---

## 📊 Dataset Specification: NYC Citi Bike (2024)

The project utilizes the official **NYC Citi Bike Trip Data**, a high-volume public dataset representing millions of individual journeys across New York City and Jersey City. 

### 📂 Data Profile
* **Source:** [Citi Bike System Data](https://citibikenyc.com/system-data)
* **Format:** Monthly partitioned CSV files / Delta Lake (Bronze Layer)
* **Temporal Scope:** Full-year 2024 coverage
* **Granularity:** One record per individual trip

### 🧬 Core Schema (Input)
| Attribute | Type | Description |
| :--- | :--- | :--- |
| `ride_id` | String | Unique UUID for every trip. |
| `rideable_type` | String | Vehicle category: `classic_bike` or `electric_bike`. |
| `started_at` | Timestamp | Departure date and time (UTC). |
| `ended_at` | Timestamp | Arrival date and time (UTC). |
| `start_station_name` | String | Pickup location name (primary SCD2 target). |
| `end_station_name` | String | Drop-off location name. |
| `member_casual` | String | User type: `member` (annual sub) or `casual` (pass). |

### ⚠️ Engineering Challenges Addressed
* **Historical Name Drifts:** Station names are updated by city planners. SCD Type 2 ensures a trip taken in January is linked to the station name valid *at that time*.
* **Data Volume:** Processing millions of rows requires the distributed compute power of **Spark** and the optimizations of **Delta Lake**.
* **Noise Filtering:** Pipeline logic filters out "false starts" (trips < 60s) to ensure the Gold Layer reflects true urban mobility.

---

## 🏗 Architecture Flow
The data follows a structured Medallion path from raw ingestion to actionable insights:

1.  **Bronze (Raw):** Automated ingestion of 2024 trip data into Delta tables with schema enforcement.
2.  **Silver (Dimensions):** A specialized "SCD 2 Engine" that maintains a versioned history of station attributes.
3.  **Gold (Analytics):** High-density aggregations focused on commuter segments and demand metrics.

---

## 🛠 Layer Details

### 🥈 Silver Layer: SCD Type 2 Dimension
To ensure historical accuracy, I modeled the `dim_stations` table using SCD Type 2 logic. Even if a station was renamed or moved during the year, the Gold Layer correctly joins the "Point-in-Time" name based on the trip timestamp.

| Column | Description | Baseline Value |
| :--- | :--- | :--- |
| `start_date` | Record activation date | `2023-12-31` |
| `end_date` | Record expiration date | `9999-12-31` (Active) |
| `is_current` | Flag for the most recent record version | `TRUE` |

### 🥇 Gold Layer: Commuter Analysis
The final curated table consists of **6 strategic columns** designed for low-latency reporting:

* **Station Name:** Historically resolved via the Silver SCD2 join.
* **Hour of Day:** Temporal feature used to visualize "The Pulse" of the city.
* **Member Type:** Segmenting "Subscribers" (Daily Commuters) vs. "Casual" riders.
* **Commuter Segment:** A derived feature classifying **Morning Rush**, **Evening Rush**, and **Off-Peak**.
* **Total Trips:** The primary volume metric for station demand.
* **Avg Duration:** Performance metric to measure rider efficiency.

---

## ⚙️ Workflow Orchestration
I implemented **Programmatic Orchestration** using a Master Notebook pattern to enforce strict data dependencies:

```python
# Master Pipeline Orchestrator
# Enforcing order: Ingest -> Transform (SCD2) -> Aggregate
dbutils.notebook.run("./01_Bronze_Ingestion", 600)
dbutils.notebook.run("./02_Silver_SCD2", 600)
dbutils.notebook.run("./03_Gold_Analysis", 600)
