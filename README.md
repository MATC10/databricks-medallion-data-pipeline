# Databricks Medallion Data Pipeline
 
End-to-end data engineering pipeline built with **Databricks, PySpark and Delta Lake**, following the **Medallion Architecture (Bronze, Silver, Gold)**.
 
The project demonstrates how raw datasets stored in **AWS S3** can be ingested into Databricks, transformed with PySpark, and structured into analytics-ready tables.
 
This architecture replicates a **typical lakehouse data engineering workflow used in production environments**.
 
---
 
# Architecture
 
The pipeline follows the Medallion Architecture pattern where data progressively improves in quality across layers.
 
```
Data Source
    │
    ▼
AWS S3 (Raw Data Storage)
    │
    ▼
Databricks
    │
    ▼
Bronze Layer (Raw Data)
    │
    ▼
Silver Layer (Cleaned Data)
    │
    ▼
Gold Layer (Analytics Tables)
    │
    ▼
BI / Analytics
```
 
### Bronze Layer
The Bronze layer stores raw ingested data from AWS S3 without transformations.
 
### Silver Layer
The Silver layer applies cleaning, validation and schema standardization.
 
### Gold Layer
The Gold layer contains aggregated datasets optimized for analytics and reporting.
 
---
 
# Tech Stack
 
- Databricks
- PySpark
- Delta Lake
- AWS S3
- Spark SQL
- Medallion Architecture
 
Optional integrations:
 
- Power BI
- Tableau
- dbt
- Apache Airflow
 
---
 
# Project Structure
 
```
databricks-medallion-data-pipeline
│
├── notebooks
│   ├── 01_bronze_ingestion.py
│   ├── 02_silver_transformations.py
│   ├── 03_gold_aggregations.py
│   └── calendar_dimension.py
│
├── data
│   ├── trips_data.csv
│   └── city_dimension.csv
│
├── architecture
│   └── architecture_diagram.png
│
└── README.md
```
 
---
 
# Data Storage – AWS S3
 
The raw datasets are stored in **Amazon S3**, which acts as the data lake storage layer.
 
Databricks connects to the S3 bucket to read the files and ingest them into the Bronze layer.
 
This setup simulates a **real-world data engineering architecture where storage and compute are separated**.
 
### Architecture Flow
 
```
AWS S3 (Raw Files)
        │
        ▼
Databricks
        │
        ▼
Bronze Tables
        │
        ▼
Silver Tables
        │
        ▼
Gold Tables
```
 
---
 
# Dataset
 
The project uses two datasets.
 
### Trips Dataset
 
Contains trip-level transactional data.
 
Example columns:
 
- trip_id
- city_id
- trip_date
- fare_amount
- distance_travelled
- driver_rating
- customer_rating
 
### City Dimension
 
Contains reference data for cities.
 
Example columns:
 
- city_id
- city_name
 
---
 
# Bronze Layer – Raw Data Ingestion
 
The Bronze layer ingests raw data from AWS S3 and stores it in Delta tables.
 
Example ingestion:
 
```python
df = spark.read.format("csv") \
    .option("header", True) \
    .load("s3://your-bucket-name/trips_data.csv")
 
df.write.format("delta").saveAsTable("bronze_trips")
```
 
Characteristics:
 
- raw schema preserved
- historical data maintained
- stored in Delta format
 
---
 
# Silver Layer – Data Transformation
 
The Silver layer applies transformations and data cleaning.
 
Typical operations include:
 
- schema standardization
- type casting
- null handling
- deduplication
 
Example transformation:
 
```python
from pyspark.sql.functions import col
 
silver_df = bronze_df \
    .withColumn("fare_amount", col("fare_amount").cast("double")) \
    .dropDuplicates()
 
silver_df.write.format("delta").mode("overwrite").saveAsTable("silver_trips")
```
 
---
 
# Calendar Dimension
 
A calendar dimension table is generated programmatically to enable time-based analysis.
 
Example fields:
 
- date
- year
- month
- quarter
- weekday
- weekend_flag
 
Example implementation:
 
```python
from pyspark.sql.functions import *
 
calendar_df = spark.range(0, 365) \
    .withColumn("date", expr("date_add('2024-01-01', id)")) \
    .withColumn("year", year("date")) \
    .withColumn("month", month("date"))
```
 
---
 
# Gold Layer – Analytics Tables
 
The Gold layer contains curated datasets optimized for analytics.
 
Example metrics include:
 
- total trips
- average fare
- trips per city
- distance travelled
- average ratings
 
Example aggregation:
 
```python
from pyspark.sql.functions import count, avg
 
gold_df = silver_trips \
    .groupBy("city_id") \
    .agg(
        count("*").alias("total_trips"),
        avg("fare_amount").alias("avg_fare")
    )
 
gold_df.write.format("delta").mode("overwrite").saveAsTable("gold_trip_metrics")
```
 
---
 
# Example Query
 
```sql
SELECT
    city_id,
    COUNT(*) AS total_trips,
    AVG(fare_amount) AS avg_fare
FROM gold_trip_metrics
GROUP BY city_id
```
 
---
 
# Running the Project
 
### 1. Create Databricks Workspace
 
Create a Databricks account and workspace.
 
### 2. Upload Data to AWS S3
 
Upload the raw CSV datasets to your S3 bucket.
 
Example structure:
 
```
s3://your-bucket-name/trips_data.csv
s3://your-bucket-name/city_dimension.csv
```
 
### 3. Configure Databricks Access to S3
 
Configure credentials or IAM roles to allow Databricks to access the S3 bucket.
 
### 4. Import Notebooks
 
Upload the notebooks from this repository into Databricks.
 
### 5. Run the Pipeline
 
Execute notebooks in order:
 
1. Bronze ingestion  
2. Silver transformations  
3. Gold aggregations  
 
---
 
# Potential Improvements
 
Possible improvements for production-grade pipelines:
 
- streaming ingestion with Auto Loader
- orchestration with Apache Airflow
- data transformations with dbt
- automated data quality checks
- CI/CD deployment for pipelines
 
---
 
# Use Cases
 
This pipeline can support analytical workloads such as:
 
- transportation demand analysis
- revenue monitoring
- customer rating insights
- city performance comparisons
 
---
