# 🏨 Hotel Analytics Dashboard: End-to-End Data Engineering
[![Snowflake](https://img.shields.io/badge/Data_Warehouse-Snowflake-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white)](https://www.snowflake.com/)
[![SQL](https://img.shields.io/badge/Language-SQL-F80000?style=for-the-badge&logo=microsoft-sql-server&logoColor=white)](#)

## 📖 Project Overview
The objective of this project is to transform raw, inconsistent hotel booking data into a structured analytical environment. By leveraging **Snowflake**, I built a data pipeline that standardizes fragmented information to provide management with a clear view of monthly revenue trends and city-level performance.

### 📊 Final Dashboard
![Hotel Analytics Dashboard](dashboard.png)
https://app.snowflake.com/tmnhqkx/ls18624/#/hotel_booking_analytics-dAIY1Lsv7

---

## 🎯 Business Objectives
* **Data Standardization:** Clean and normalize raw datasets for accurate reporting.
* **Trend Analysis:** Visualize monthly revenue and booking fluctuations.
* **Geographic Insights:** Identify top revenue-generating cities to optimize marketing spend.
* **Operational Metrics:** Analyze booking distribution by type and status (Cancellations vs. Completions).
* **Executive KPIs:** Display high-level metrics like **Total Revenue** and **Total Bookings** at a glance.

---

## 🏗 Technical Architecture
The project follows a modular data engineering workflow within the Snowflake ecosystem:

1.  **Ingestion:** Loading raw data into Snowflake stages.
2.  **Transformation:** Using SQL to handle date formatting, deduplication, and missing values.
3.  **Aggregation:** Creating summary tables for optimized dashboard performance.
4.  **Visualization:** Connecting the Gold-layer tables to a dashboard (Snowsight) for stakeholder review.

---

## 🛠 Functional Requirements

### 1. Data Cleaning & ETL
* **Date Normalization:** Standardize inconsistent date formats across different sources.
* **Data Integrity:** Remove duplicate entries and handle null values in critical fields.
* **Aggregation Logic:** Transform transactional data into monthly snapshots for trend analysis.

### 2. Dashboard & Visuals
* **Revenue Metrics:** Line charts showing **Revenue per Month**.
* **Volume Metrics:** Line charts tracking **Bookings per Month**.
* **Market Share:** Bar charts highlighting **Top Cities by Revenue**.
* **Status Analysis:** Breakdown of **Booking Types** and **Booking Status**.

---

## 🚀 Key Results
* **Unified Source of Truth:** Eliminated data silos by centralizing raw data in Snowflake.
* **Improved Decision Making:** Reduced time-to-insight for management from days to seconds.
* **Enhanced Data Quality:** Achieved 100% consistency in date and currency formats.

---

## 📂 Project Structure
* `/sql_scripts`: Contains DDL and DML scripts for Snowflake.
* `/data`: Sample raw datasets (if applicable).
* `dashboard.png`: Final visualization output.
