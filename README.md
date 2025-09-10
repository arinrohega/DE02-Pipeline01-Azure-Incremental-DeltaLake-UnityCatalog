# Data Engineering Pipeline - Microsoft Azure
## Introduction

This project simulates a production-grade data engineering pipeline that automates the extraction of operational data from an OLTP database, to deliver cleaned datasets in a DeltaLake environment. The pipeline is built using modern Big Data applications in Azure Cloud for enterprise-level data workflows.

(Designed by Alejandro Rodríguez for educational and skill demonstration purposes)

## Tools Used
- 🛢️ MySQL DB
- 🔄 Azure Data Factory
- ⚡ Databricks
- 🤖 PySpark
- 🗂️ Azure Data Lake Storage
- 🔐 Unity Catalog

## Case Scenario
The data consumers (stakeholders and end users) require tables to be delivered with the following conditions:

- Table Format: Delta (parquet) 
- Frecuency: Batch
- Flow: Incremental
- Environment: On-Cloud
- Source: OLTP database

- Description:

An applicants/prospects star schema tables that shows the most relevant metrics from their affiliation process, and calculating the date of first purchase which marks their transition to official affiliated status.

## Architecture

<img width="1672" height="679" alt="222 drawio" src="https://github.com/user-attachments/assets/1fc1a6db-d907-4f7d-8653-f7981ecf98b9" />

## Pipeline Preview 

<img width="1640" height="746" alt="adf pipeline" src="https://github.com/user-attachments/assets/bc556c6c-76e7-4368-abf9-a6899960cf0a" />

<img width="1044" height="306" alt="dbjon" src="https://github.com/user-attachments/assets/fb834065-a343-4517-bf07-b4bcd2ad8d0d" />

## Source Data Model
### Complete OLTP Database Model
![datamodel1](https://github.com/user-attachments/assets/cc53a22c-a9ae-44d2-9b59-fb38ac1bdeeb)

### Chart for desired Table
<img width="1561" height="842" alt="ETLLLL drawio" src="https://github.com/user-attachments/assets/8bfd29d6-1900-41f6-9dfb-394713cde4da" />

## Proyect Showcase Guide 

### 1. Azure Data Lake Storage (ADLS) 🗂️

<img width="1916" height="282" alt="LOGO1" src="https://github.com/user-attachments/assets/b78078a6-16e1-4194-a74d-5e21e21d0730" />


### 1.1 Creating Medallion Architecture🗂️

After creating a resource group in Azure Protal and a Storage Account for ADLS, the following containers were created to be the physical location of the data, naming 3 of them with a medallion hierarchy, one for landing raw data, and one for the metastore of Unity Catalog:

<img width="1224" height="558" alt="adls1" src="https://github.com/user-attachments/assets/d2d25570-79f5-4ac6-9218-356502e2e32a" />

### 1.2 Incremental Loading Setup 🗂️

To create a watermark method for incremental loading in Azure Data Factory, an empty csv dataset with only header was uploaded to the Landing Container:

<img width="1233" height="430" alt="adls solo empty" src="https://github.com/user-attachments/assets/f7ce7e7a-53ff-4a88-aefc-8fdf50eb4804" />

----------------------------------------
### 2. Azure Data Factory (ADF) 🔄

<img width="1916" height="282" alt="LOGOADF" src="https://github.com/user-attachments/assets/b3cc7682-4f2b-4a5b-882f-7affadf93ba0" />


### 2.1 Objective🔄

The main goal for the pipeline and each activity can be described like this:  

For the first batch:
1) Read the full desired source table from MySQL.
2) Write it to the ADLS Landing Bucket using Parquet format.
3) Write a separated csv with the watermark value for this batch.
4) Repeat this for each desired table.

For the following batches:
1) Read only the new and updated rows from the desired table, using the watermark csv. 
2) Write it to the ADLS Landing Bucket on Parquet format.
3) Update the watermark value for this batch.
4) Repeat this for each desired table.
   
Trigger a Databrick job after every batch.

### 2.2 Adding Integration Runtime🔄

To connect MySQL to the cloud being different network environments, Integration Runtime was added through the Manage section of ADF:

<img width="1213" height="399" alt="integrationruntime" src="https://github.com/user-attachments/assets/622a509b-d2f5-4d68-9d29-37bead1677ba" />

### 2.3 Creating Datasets🔄

Datasets for source and target storages were mounted with the following configurations:

1) **ADLS_pipeline01_table**: Targets the desired location for the source table.  

<img width="982" height="460" alt="dataset1" src="https://github.com/user-attachments/assets/55572880-fe09-40f6-8c5c-e3b81bb4750f" />
<br>
Uses the following dynamic content for the path:   
<br>

```
@concat('pipeline01/', dataset().table_name) / @concat(dataset().table_name, '_', formatDateTime(addHours(utcNow(), -7), 'yyyyMMdd_HHmmss'))
```

<br>

2) **ADLS_pipeline01_watermark**: Targets the desired location for the watermark csv.   

<br>

<img width="982" height="466" alt="dataset2" src="https://github.com/user-attachments/assets/3e1e7f9d-9cab-4229-9785-ebbab2c72aa4" />
Uses the following dynamic content for the path:   
<br>

```
@concat('pipeline01/_watermarks/', dataset().table_name) / @concat('watermark_lastBatch_',dataset().table_name)
```
<br>

3) **Empty_csv**: Targets the csv already uploaded in ADLS   

<br>
<br>

<img width="928" height="266" alt="dataset3" src="https://github.com/user-attachments/assets/67b3889e-d42c-43b5-9354-7029182fa609" />

<br>
<br>

4) **MySQL_companysimulation**: Targets the source OLTP database

<img width="866" height="268" alt="dataset4" src="https://github.com/user-attachments/assets/3acee268-83fc-4116-a13a-7aec1a280ba1" />

The dataset list looked like this after creating all of them:

<img width="1916" height="577" alt="datrasets" src="https://github.com/user-attachments/assets/a21ce129-6631-4847-ae11-a3f044ebbb34" />


### 2.4 Incremental Pipeline´s Activities🔄

1) **Por Cada Tabla (ForEach):** Loops through the list of the 10 tables to process each one.
2) **Check If WM Table Exists (GetMetadata):** Checks if the watermark file for the table exists in the **ADLS_pipeline01_watermark dataset**.
3) **If Not Exists Then True (IfCondition):** Triggers true if its the first Batch, because no watermark was found.
4) **Get Actual Max Watermark (Lookup):** Gets the actual maximum value from the "updated_at" column in the **MySQL_companysimulation** table.
5) **ReadSQL WriteADLS Complete (Copydata):** Copies of the entire table from **MySQL_companysimulation** into **ADLS_pipeline01_table** dataset
6) **Create Watermark Table (Copydata):** Copies **Empty_csv**, adds a watermark column with the max "updated_at" value, and sinks it to **ADLS_pipeline01_watermark** dataset
7) **Get Actual Max Watermark 2 (Lookup):** Gets the current max "updated_at" value from **MySQL_companysimulation** for incremental load.
8) **Get Last Watermark (Lookup):** Retrieves the last stored watermark value from **ADLS_pipeline01_watermark** dataset
9) **If New Rows Then True (IfCondition):** Compares actual and last stored watermark value.
10) **ReadSQL WriteADLS Incremental (Copydata):** Copies from **MySQL_companysimulation** filtering by only the new or changed records since the last watermark.
11) **Update Watermark Table (Copydata):** Updates the watermark file with the new max value for the next batch.
12) **Databricks Job (DatabricksJob):** Triggers a Databricks job after all tables are processed.
----------------------------------------

### 3. Databricks ⚡

<img width="1916" height="282" alt="LOGOdb" src="https://github.com/user-attachments/assets/182df7f2-eb04-4c24-8f69-56f85200a300" />

### 3.1 Setting up Resources with Unity Catalog ⚡🔐

To use Databricks with Unity Catalog, Databricks´s signature governance layer to the medallion containers, the following steps were done:  

Databricks and Access Connector for Azure Databricks resources were created from Azure Portal:  

<img width="936" height="93" alt="db recursos" src="https://github.com/user-attachments/assets/7383f0dc-000b-445b-aced-cfc8ffe239e9" />

"Storage blob data contributor" role was asigned to Access Connector from Storage Account Access Control:  

<img width="1219" height="372" alt="dbcontributor" src="https://github.com/user-attachments/assets/96f5d6d7-8b03-4ad5-91e4-5080ed224158" />  

On Databricks´s Account portal, a metastore was created using the ADLS path of the Metastore Container and the Resource ID from Access Connector. It was assigned to our workspace as well.  

<img width="1122" height="322" alt="zdb1" src="https://github.com/user-attachments/assets/7f5db58f-b818-442b-810b-884d5d87b23d" />  

On Databricks Workspace, the following external locations were created:  

<img width="1231" height="598" alt="zdb2" src="https://github.com/user-attachments/assets/672ba685-7036-4ead-8bc3-bc84b2f21fd9" />  

Now for any user that wants to read, write or alter any table with that location paths, Unity Catalog will validate if they have permission.  


### 3.2 Creating Catalog/Schema/Table Hierarchy ⚡

To provide a governed, SQL-friendly layer on top of raw ADLS locations, for any future user of the tables, a Catalog was created, with a schema for each medallion layer  

<img width="1589" height="422" alt="zdb34" src="https://github.com/user-attachments/assets/a16f2da7-53ca-4052-9a09-e3dc44ac1b5a" />

### 3.3 Python Notebook 1: Landing to Bronze ⚡  

The main goal for this notebook its an incremental read and upsert of only new data without transformations. It can be summarized like this:

For the first batch, the catalog table and the corresponding ADLS table have not yet been created.
1) Read the full source table on parquet from the Landing Container
2) Write it to Bronze Container using Delta Format while creating catalog.bronze.table path.
3) Repeat this for each desired table.

For following batches, a catalog table exists and a delta format table it´s storaged on ADLS location.
1) Use readstream´s checkpoint to verify if new parquets were added data to Landing Container
2) If there is no new data, terminate.
3) If there is a new batch of data, upsert it to catalog.bronze.table.
4) Repeat this for each desired table.

<img width="1344" height="2847" alt="znotebooktestt" src="https://github.com/user-attachments/assets/7c7ebaed-9cf6-457f-a0f2-d9730c0420ac" />

### 3.4 Python Notebook 2: Bronze to Silver ⚡  

The main goal for this notebook its an incremental read and upsert of only new data applying transformations. It can be summarized like this:

For the first batch, the Silver catalog table and the corresponding ADLS table have not been created yet.
1) Read the full delta table from the Bronze Container using catalog.bronze.table.
2) Apply the specific transformation that the table in turn needs.
3) Write it to Silver Container using Delta Format while creating catalog.silver.table path.
4) Repeat this for each desired table.

For following batches, the silver catalog table exists and a delta format table it´s storaged on ADLS location.
1) Use readstream´s checkpoint to verify if new delta batches were added data to Bronze Container.
2) If there is no new data, terminate.
3) If there is a new batch of data, apply the specific transformation that the table in turn needs
4) Upsert the new and transformed data into catalog.silver.table.
5) Repeat this for each desired table.

<img width="1347" height="2466" alt="znotebooktest22 1" src="https://github.com/user-attachments/assets/580ff4cb-2431-4456-949a-b79a019366e3" />

<img width="1347" height="2136" alt="znotebooktest22 2" src="https://github.com/user-attachments/assets/de445bc0-de9e-4259-8b05-00c9f6495015" />


sdfsdf


## Datasets Used
All tables were made and loaded locally to MySQL database. While all data is fictitious, each element was designed meticulously to replicate a real enterprise model, mirroring the afiliation schema of a company I've previously worked with, mantaining logical relationships and consistency across tables.

![dataset1](https://github.com/user-attachments/assets/309bd941-b8a0-40a0-ac6c-243f393cd3e7)

Theres a glossary at the end, providing English translations of the table and column names if needed.







