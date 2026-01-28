# Snowflake Hotel Booking Data Engineering Project

A complete data engineering pipeline built on Snowflake implementing a **Medallion Architecture (Bronze-Silver-Gold)** for hotel booking data processing, transformation, and analytics.

![Data Pipeline Architecture](architecture_diagram.html)

## 📋 Project Overview

This project demonstrates a production-ready data engineering pipeline that processes hotel booking data through multiple layers of transformation, ensuring data quality and preparing it for business intelligence and analytics.

### Key Features

- **Medallion Architecture**: Three-layer data architecture (Bronze → Silver → Gold)
- **Data Quality Validation**: Email validation, date consistency checks, amount validations
- **Data Cleaning**: Standardization, deduplication, and error handling
- **Aggregated Analytics**: Pre-computed metrics for dashboard performance
- **Automated Dashboard Queries**: Ready-to-use SQL queries for visualization

## 🏗️ Architecture

### Medallion Architecture Layers

```
Raw Data → Stage → Bronze → Silver → Gold → Dashboard
```

#### **Bronze Layer** (Raw Data)
- Stores raw data exactly as received from the source
- All columns stored as STRING type for flexibility
- No transformations applied
- Serves as the source of truth

#### **Silver Layer** (Cleaned Data)
- Type-safe schema with proper data types
- Data quality checks applied
- Standardized formatting (capitalization, trimming)
- Invalid records filtered out
- Business rules applied

#### **Gold Layer** (Business-Ready Data)
- Aggregated tables optimized for analytics
- Pre-computed metrics for performance
- Denormalized for query efficiency
- Ready for BI tools and dashboards

## 📊 Dataset Schema

### Source Data Fields

| Field | Description | Sample Value |
|-------|-------------|--------------|
| booking_id | Unique booking identifier | bc3c2ecb-89b3-4f87-9714-355c5088b35d |
| hotel_id | Hotel identifier | 1860 |
| hotel_city | City where hotel is located | Michelleport |
| customer_id | Customer identifier | 1d541501-4a4a-479e-9d6c-91f80cf8452e |
| customer_name | Customer full name | Tracy Ortega |
| customer_email | Customer email address | invalid-email |
| check_in_date | Check-in date | 1/11/2026 |
| check_out_date | Check-out date | 1/19/2026 |
| room_type | Type of room booked | Deluxe, Suite, Standard |
| num_guests | Number of guests | 4 |
| total_amount | Booking amount | 252.49 |
| currency | Currency code | USD, EUR, INR |
| booking_status | Booking status | Confirmed, Cancelled, No-Show |

## 🚀 Getting Started

### Prerequisites

- Snowflake account with appropriate warehouse access
- CSV file with hotel booking data
- Access to create database, schemas, and tables

### Installation & Setup

1. **Clone the repository**
```bash
git clone https://github.com/II-IronMan-II/Snowflake_Hotel_Booking_DE_Projet.git
cd Snowflake_Hotel_Booking_DE_Projet
```

2. **Create Database**
```sql
CREATE DATABASE HOTEL_DB;
USE DATABASE HOTEL_DB;
```

3. **Set up File Format**
```sql
CREATE OR REPLACE FILE FORMAT FF_CSV
    TYPE = 'CSV'
    FIELD_OPTIONALLY_ENCLOSED_BY = '"'
    SKIP_HEADER = 1
    NULL_IF = ('NULL', 'null', '');
```

4. **Create Stage**
```sql
CREATE OR REPLACE STAGE STG_HOTEL_BOOKINGS
    FILE_FORMAT = (FORMAT_NAME = FF_CSV)
    ON_ERROR = 'CONTINUE';
```

5. **Upload Data to Stage**
- Upload your CSV file to the `STG_HOTEL_BOOKINGS` stage via Snowflake UI or SnowSQL

6. **Run the Pipeline**
- Execute `processing.sql` to create tables and load data
- Execute `Dashboard.sql` to generate analytics queries

## 📁 Project Structure

```
├── processing.sql              # Complete ETL pipeline (Bronze → Silver → Gold)
├── Dashboard.sql              # Pre-built dashboard queries
├── architecture_diagram.png   # Data pipeline architecture diagram
└── README.md                  # This file
```

## 🔄 Data Pipeline Details

### Step 1: Bronze Layer - Raw Data Ingestion

**Table**: `BRONZE_HOTEL_BOOKING`

- All columns as STRING type
- No transformations
- Direct COPY from stage
- Error handling with `ON_ERROR = 'CONTINUE'`

```sql
COPY INTO BRONZE_HOTEL_BOOKING
FROM @STG_HOTEL_BOOKINGS
FILE_FORMAT = (FORMAT_NAME = FF_CSV)
ON_ERROR = 'CONTINUE';
```

### Step 2: Data Quality Checks

Before transforming to Silver layer, the pipeline performs validation checks:

#### **Email Validation**
```sql
-- Identifies invalid email formats
WHERE NOT (customer_email LIKE '%@%.%') OR customer_email IS NULL
```

#### **Amount Validation**
```sql
-- Identifies negative amounts
WHERE TRY_TO_NUMBER(total_amount) < 0
```

#### **Date Consistency Check**
```sql
-- Ensures check-out date is after check-in date
WHERE TRY_TO_DATE(check_out_date) < TRY_TO_DATE(check_in_date)
```

#### **Status Standardization**
```sql
-- Identifies status variations (confirmeeed, confirmd)
SELECT DISTINCT booking_status FROM BRONZE_HOTEL_BOOKING;
```

### Step 3: Silver Layer - Data Cleaning & Transformation

**Table**: `SILVER_HOTEL_BOOKINGS`

Transformations applied:

1. **Type Casting**
   - String → DATE (check_in_date, check_out_date)
   - String → INTEGER (num_guests)
   - String → FLOAT (total_amount)

2. **Text Standardization**
   - `INITCAP()`: Capitalizes first letter (hotel_city, customer_name)
   - `TRIM()`: Removes leading/trailing spaces
   - `LOWER()`: Lowercase emails

3. **Data Cleaning**
   - Email validation with pattern matching
   - Absolute value for negative amounts: `ABS(TRY_TO_NUMBER(total_amount))`
   - Status standardization: 'confirmeeed', 'confirmd' → 'Confirmed'

4. **Data Filtering**
   - Removes records with NULL dates
   - Removes records where check_out < check_in
   - Handles NULL values properly

```sql
INSERT INTO SILVER_HOTEL_BOOKINGS
SELECT 
    booking_id,
    hotel_id,
    INITCAP(TRIM(hotel_city)) AS hotel_city,
    customer_id,
    INITCAP(TRIM(customer_name)) AS customer_name,
    CASE
        WHEN customer_email LIKE '%@%.%' THEN LOWER(TRIM(customer_email))
        ELSE NULL
    END AS customer_email,
    TRY_TO_DATE(NULLIF(check_in_date, '')) AS check_in_date,
    TRY_TO_DATE(NULLIF(check_out_date, '')) AS check_out_date,
    room_type,
    num_guests,
    ABS(TRY_TO_NUMBER(total_amount)) AS total_amount,
    currency,
    CASE
        WHEN LOWER(booking_status) IN ('confirmeeed', 'confirmd') THEN 'Confirmed'
        ELSE booking_status
    END AS booking_status
FROM BRONZE_HOTEL_BOOKING
WHERE
    TRY_TO_DATE(check_in_date) IS NOT NULL
    AND TRY_TO_DATE(check_out_date) IS NOT NULL
    AND TRY_TO_DATE(check_out_date) >= TRY_TO_DATE(check_in_date);
```

### Step 4: Gold Layer - Business Analytics Tables

The Gold layer contains three optimized tables:

#### 1. **GOLD_AGG_DAILY_BOOKING**
Daily aggregated booking metrics for time-series analysis.

| Column | Description |
|--------|-------------|
| date | Check-in date |
| total_booking | Count of bookings per day |
| total_revenue | Sum of booking amounts per day |

**Use Case**: Trend analysis, forecasting, seasonality detection

#### 2. **GOLD_AGG_HOTEL_CITY_SALES**
Revenue aggregated by city for geographical analysis.

| Column | Description |
|--------|-------------|
| hotel_city | City name |
| total_revenue | Total revenue per city |

**Use Case**: Market analysis, expansion planning, performance comparison

#### 3. **GOLD_BOOKING_CLEAN**
Clean, denormalized booking records ready for detailed analysis.

All fields from Silver layer with guaranteed data quality.

**Use Case**: Detailed reporting, customer segmentation, operational analytics

## 📈 Dashboard Queries

The project includes pre-built SQL queries for creating an analytics dashboard:

### Key Performance Indicators (KPIs)

1. **Total Revenue**
```sql
SELECT SUM(total_amount) AS total_revenue
FROM GOLD_BOOKING_CLEAN;
```

2. **Total Bookings**
```sql
SELECT COUNT(*) AS total_bookings
FROM GOLD_BOOKING_CLEAN;
```

3. **Total Guests**
```sql
SELECT SUM(num_guests) AS total_guests
FROM GOLD_BOOKING_CLEAN;
```

4. **Average Booking Value**
```sql
SELECT AVG(total_amount) AS avg_booking_value
FROM GOLD_BOOKING_CLEAN;
```

### Visualizations

#### **Line Charts**

**Monthly Revenue Trend**
```sql
SELECT date, total_revenue
FROM GOLD_AGG_DAILY_BOOKING
ORDER BY date;
```

**Monthly Bookings Trend**
```sql
SELECT date, total_booking
FROM GOLD_AGG_DAILY_BOOKING
ORDER BY date;
```

#### **Bar Charts**

**Top 5 Cities by Revenue**
```sql
SELECT hotel_city, total_revenue
FROM GOLD_AGG_HOTEL_CITY_SALES
WHERE total_revenue IS NOT NULL
ORDER BY total_revenue DESC
LIMIT 5;
```

**Bookings by Status**
```sql
SELECT booking_status, COUNT(*) AS total
FROM GOLD_BOOKING_CLEAN
GROUP BY booking_status;
```

**Bookings by Room Type**
```sql
SELECT room_type, COUNT(*) AS total_bookings
FROM GOLD_BOOKING_CLEAN
GROUP BY room_type
ORDER BY total_bookings DESC;
```

## 🛠️ Data Quality Rules

| Rule | Implementation | Purpose |
|------|----------------|---------|
| Email Format | Pattern match `%@%.%` | Ensure valid email addresses |
| Date Validity | `TRY_TO_DATE()` with NULL check | Prevent invalid date formats |
| Date Logic | check_out >= check_in | Ensure logical booking periods |
| Amount Sign | `ABS()` function | Convert negative to positive |
| Status Normalization | CASE statement | Standardize spelling variations |
| Text Formatting | `INITCAP()`, `TRIM()`, `LOWER()` | Consistent formatting |

## 💡 Key Design Decisions

### Why Medallion Architecture?

1. **Separation of Concerns**: Each layer has a specific purpose
2. **Data Lineage**: Clear tracking from raw to analytics-ready data
3. **Flexibility**: Easy to reprocess without losing raw data
4. **Performance**: Optimized tables for different use cases
5. **Governance**: Clear data quality boundaries

### Why Pre-Aggregated Tables?

1. **Query Performance**: Avoid repeated aggregation calculations
2. **Dashboard Speed**: Fast load times for BI tools
3. **Cost Optimization**: Reduced compute for frequent queries
4. **Consistency**: Same metrics across all dashboards

### Error Handling Strategy

- `ON_ERROR = 'CONTINUE'`: Prevents pipeline failure on bad records
- `TRY_TO_*()` functions: Graceful handling of type conversion errors
- `NULLIF()`: Explicit NULL handling for empty strings
- Filter conditions: Remove invalid records rather than error out

## 🔍 Data Quality Metrics

After running the pipeline, you can check data quality:

```sql
-- Records loaded into Bronze
SELECT COUNT(*) AS bronze_count FROM BRONZE_HOTEL_BOOKING;

-- Records that passed validation
SELECT COUNT(*) AS silver_count FROM SILVER_HOTEL_BOOKINGS;

-- Data quality rate
SELECT 
    (SELECT COUNT(*) FROM SILVER_HOTEL_BOOKINGS) * 100.0 / 
    (SELECT COUNT(*) FROM BRONZE_HOTEL_BOOKING) AS quality_rate_pct;

-- Invalid emails caught
SELECT COUNT(*) AS invalid_emails
FROM BRONZE_HOTEL_BOOKING
WHERE NOT (customer_email LIKE '%@%.%') OR customer_email IS NULL;

-- Invalid date ranges caught
SELECT COUNT(*) AS invalid_dates
FROM BRONZE_HOTEL_BOOKING
WHERE TRY_TO_DATE(check_out_date) < TRY_TO_DATE(check_in_date);
```

## 🎯 Use Cases

This pipeline supports various business use cases:

1. **Revenue Analytics**: Track daily, monthly, and city-level revenue
2. **Booking Trends**: Analyze booking patterns and seasonality
3. **Customer Segmentation**: Group customers by booking behavior
4. **Operational Metrics**: Monitor booking status distribution
5. **Market Analysis**: Compare performance across cities
6. **Capacity Planning**: Understand guest volume trends
7. **Forecasting**: Predict future bookings and revenue

## 🚀 Advanced Features

### Incremental Loading

To load new data incrementally:

```sql
-- Load only new records
COPY INTO BRONZE_HOTEL_BOOKING
FROM @STG_HOTEL_BOOKINGS
FILE_FORMAT = (FORMAT_NAME = FF_CSV)
ON_ERROR = 'CONTINUE'
FORCE = FALSE;  -- Skip already loaded files

-- Append to Silver
INSERT INTO SILVER_HOTEL_BOOKINGS
SELECT ...
FROM BRONZE_HOTEL_BOOKING
WHERE booking_id NOT IN (SELECT booking_id FROM SILVER_HOTEL_BOOKINGS);

-- Refresh Gold aggregations
TRUNCATE TABLE GOLD_AGG_DAILY_BOOKING;
INSERT INTO GOLD_AGG_DAILY_BOOKING
SELECT ... FROM SILVER_HOTEL_BOOKINGS ...;
```

### Scheduling with Tasks

```sql
-- Create task to refresh Gold layer daily
CREATE OR REPLACE TASK REFRESH_GOLD_TABLES
  WAREHOUSE = COMPUTE_WH
  SCHEDULE = 'USING CRON 0 2 * * * UTC'  -- 2 AM daily
AS
BEGIN
  TRUNCATE TABLE GOLD_AGG_DAILY_BOOKING;
  INSERT INTO GOLD_AGG_DAILY_BOOKING
  SELECT check_in_date, COUNT(*), SUM(total_amount)
  FROM SILVER_HOTEL_BOOKINGS
  GROUP BY check_in_date;
  
  -- Repeat for other Gold tables
END;

ALTER TASK REFRESH_GOLD_TABLES RESUME;
```

## 📊 Sample Insights

Based on the sample data, the pipeline can reveal:

- **Currency Distribution**: Bookings in USD, EUR, INR
- **Room Preferences**: Distribution across Standard, Deluxe, Suite
- **Booking Status**: Confirmed, Cancelled, No-Show patterns
- **Guest Patterns**: Average party size per booking
- **Revenue Trends**: Daily and city-level performance

## 🧹 Cleanup

```sql
-- Drop all tables
DROP TABLE IF EXISTS GOLD_BOOKING_CLEAN;
DROP TABLE IF EXISTS GOLD_AGG_HOTEL_CITY_SALES;
DROP TABLE IF EXISTS GOLD_AGG_DAILY_BOOKING;
DROP TABLE IF EXISTS SILVER_HOTEL_BOOKINGS;
DROP TABLE IF EXISTS BRONZE_HOTEL_BOOKING;

-- Drop stage and file format
DROP STAGE IF EXISTS STG_HOTEL_BOOKINGS;
DROP FILE FORMAT IF EXISTS FF_CSV;

-- Drop database (optional)
DROP DATABASE IF EXISTS HOTEL_DB;
```

## 📚 References

- [Snowflake Medallion Architecture](https://www.snowflake.com/guides/medallion-architecture)
- [Snowflake Data Loading](https://docs.snowflake.com/en/user-guide/data-load-overview)
- [SQL Best Practices](https://docs.snowflake.com/en/user-guide/sql-best-practices)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is for educational and demonstration purposes.

## 👤 Author

**II-IronMan-II**
- GitHub: [@II-IronMan-II](https://github.com/II-IronMan-II)

---

**Note**: This project demonstrates core data engineering principles including data quality validation, transformation pipelines, and analytics-ready data modeling using Snowflake.
