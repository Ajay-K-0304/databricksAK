# Health Tracker Data Visualizer

An end-to-end Data Engineering and Analytics project that processes health and fitness activity data using Azure, Databricks, Spark, Delta Lake, and Power BI.

The project demonstrates an incremental ELT pipeline that takes raw CSV data, cleans and transforms it through a Medallion Architecture, performs incremental UPSERT operations, and produces business-ready datasets for interactive Power BI dashboards.

---

## Project Overview

The objective of this project is to build a scalable pipeline for analyzing health-related activity data such as:

- Daily steps
- Calories burned
- Distance covered
- Active minutes
- Sleep hours
- Heart rate
- Workout type
- Weather conditions
- Location
- Mood

The pipeline is designed to support both historical data and incremental data arriving through new input files.

---

## Architecture

```text
Raw CSV File
     |
     v
Azure Data Factory
(Event / Trigger)
     |
     v
Azure Databricks
     |
     v
+----------------------+
|   Bronze Layer       |
|   Raw Delta Data     |
+----------+-----------+
           |
           v
+----------------------+
|   Silver Layer       |
| Cleaning &           |
| Standardization      |
|                      |
| Delta MERGE / UPSERT |
+----------+-----------+
           |
           v
+----------------------+
|    Gold Layer        |
| Business Aggregation |
| & Analytical Tables  |
+----------+-----------+
           |
           v
Databricks SQL Warehouse
           |
           v
Power BI Dashboard
```
## How to Run the Project

The project can be executed on Azure using Azure Data Lake Storage Gen2, Azure Data Factory, and Azure Databricks.

### Prerequisites

Make sure you have access to:

- Azure Subscription
- Azure Data Lake Storage Gen2
- Azure Data Factory
- Azure Databricks Workspace
- Databricks Runtime with PySpark
- Databricks SQL Warehouse
- Power BI Desktop
- Required Azure permissions to create and access the above resources

---

### 1. Create Azure Storage Account

Create an Azure Storage Account with **Hierarchical Namespace (HNS)** enabled to use it as Azure Data Lake Storage Gen2.

Create the following containers:

```text
input/
output/
```
##Recommended Structure
The input container is used for incoming CSV files, while the output container stores the processed Delta Lake data.
input/
    health_data.csv
    incremental_01.csv
    incremental_02.csv

output/
    bronze/
    silver/
    gold/

### 2.Upload Input Data
Upload the initial health activity CSV file into the input container.

For incremental processing, additional CSV files can be uploaded later to the same container.
input/
├── health_data.csv
├── incremental_01.csv
└── incremental_02.csv
### 3.Configure Azure Databricks
Create an Azure Databricks workspace and configure a compute resource capable of running PySpark notebooks.

Import the project notebook(s) into the Databricks workspace.

Configure access to the ADLS Gen2 storage account using the appropriate Azure authentication method, such as:

- Access Connector / Managed Identity
- Service Principal
- Other supported Azure authentication mechanisms

Update the storage paths in the notebook according to your Azure Storage Account and container names.
#Example
input_path = "abfss://input@<storage-account>.dfs.core.windows.net/"
output_path = "abfss://output@<storage-account>.dfs.core.windows.net/"
### 4.Run the databricks notebook
The Databricks notebook performs the main data processing workflow:
```text
Input CSV
    ↓
Bronze Delta Layer
    ↓
Silver Transformation
    ↓
Delta MERGE / UPSERT
    ↓
Gold Transformation
    ↓
Gold Delta Tables
```
### 5.Configure Azure DataFactory
Create an Azure Data Factory instance and configure:
##Linked Services
Create linked services for:
Azure Data Lake Storage Gen2
Azure Databricks
The ADLS linked service provides access to the input/output containers, while the Databricks linked service is used to execute the processing notebook.
### 6.Create the ADF Pipeline
Create a pipeline that contains a Databricks Notebook Activity.
The pipeline should pass the incoming file name/path to the Databricks notebook as a parameter.
Example: 
```text
ADLS Gen2
   ↓
ADF Pipeline
   ↓
Databricks Notebook
   ↓
Delta Lake Processing
```
The exact parameter name should match the parameter used by the Databricks notebook.
### 7.Configure a storage event trigger
To enable automatic incremental processing, configure an ADF Storage Event Trigger.
Configure the trigger to monitor the input container for newly created files.
```text
New CSV uploaded
       ↓
Storage Event Trigger
       ↓
ADF Pipeline
       ↓
Databricks Notebook
       ↓
Bronze → Silver → Gold
```
This allows the pipeline to automatically start whenever a new input file is uploaded.
### 8.Verify the output
After the ADF pipeline completes successfully, check the output container in ADLS Gen2.
The processed Delta data should be available under the corresponding Bronze, Silver, and Gold directories.
Example:
```text
output/
├── bronze/
│
├── silver/
│   ├── userActivityDelta/
│   ├── workoutDelta/
│   ├── sleepDelta/
│   ├── locationDelta/
│   └── moodDelta/
│
└── gold/
    ├── userActivityDelta/
    ├── workoutDelta/
    ├── sleepDelta/
    ├── locationDelta/
    └── moodDelta/
```
The exact directory names depend on the paths configured in the notebook.
### 9.Configure Databricks SQL Warehouse
Create or configure a Databricks SQL Warehouse and register the Gold Delta datasets as SQL tables.
Example:
```text
CREATE DATABASE IF NOT EXISTS gold;
```
Then create external Delta tables pointing to the Gold storage locations:
```text
CREATE TABLE IF NOT EXISTS gold.user_summary
USING DELTA
LOCATION '<gold-delta-location>';
```
Repeat this for the required Gold datasets.
### 10.Connect PowerBI
Open Power BI Desktop and connect to the Databricks SQL Warehouse.

Load the Gold tables required for reporting.

The dashboard contains:

- User Summary
- Steps
- Distance
- Active Time
- Workout Analysis
- Calories Burned
- Average Heart Rate
- Workout Sessions
- Sleep & Mood Analysis
- Sleep Hours
- Mood
- Location Analysis
### End to End Execution
Once everything is configured, the normal execution flow is:
```text
Upload CSV to ADLS Gen2
          ↓
Storage Event Trigger
          ↓
Azure Data Factory
          ↓
Databricks Notebook
          ↓
Bronze Delta
          ↓
Silver Cleaning + MERGE/UPSERT
          ↓
Gold Transformations
          ↓
ADLS Gen2 Output
          ↓
Databricks SQL Warehouse
          ↓
Power BI Dashboard
```
