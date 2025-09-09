# Data Engineering Pipeline - Microsoft Azure
## Introduction

This project simulates a production-grade data engineering pipeline that automates the extraction of operational data from an OLTP database, to deliver cleaned datasets in a DeltaLake+Warehouse environment. The pipeline is built using modern Big Data applications in Azure Cloud for enterprise-level data workflows.

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

- Table Format: Delta (parquet) Star Schema
- Frecuency: Batch
- Flow: Incremental
- Environment: On-Cloud
- Source: OLTP database

- Description:

An applicants/prospects star schema tables that shows the most relevant metrics from their affiliation process, and calculating the date of first purchase which marks their transition to official affiliated status.

## Architecture

<img width="1672" height="679" alt="222 drawio" src="https://github.com/user-attachments/assets/1fc1a6db-d907-4f7d-8653-f7981ecf98b9" />

## Data Model
### Complete OLTP Database Model
![datamodel1](https://github.com/user-attachments/assets/cc53a22c-a9ae-44d2-9b59-fb38ac1bdeeb)

### Chart for desired Table
![etlclient drawio](https://github.com/user-attachments/assets/9ec63cda-8cea-4b85-8741-d7cc2f0cb402)

## Datasets Used
All tables were made and loaded locally to MySQL database. While all data is fictitious, each element was designed meticulously to replicate a real enterprise model, mirroring the afiliation schema of a company I've previously worked with, mantaining logical relationships and consistency across tables.

![dataset1](https://github.com/user-attachments/assets/309bd941-b8a0-40a0-ac6c-243f393cd3e7)

Theres a glossary at the end, providing English translations of the table and column names if needed.


### 1. Azure Data Lake Storage (ADLS) 🗂️

<img width="1916" height="282" alt="LOGO1" src="https://github.com/user-attachments/assets/b78078a6-16e1-4194-a74d-5e21e21d0730" />


### 1.1 Creating Medallion Architecture🗂️

After creating a resource group in Azure Protal and a Storage Account for ADLS, the following containers were created to be the physical location of the data, naming 3 of them with a medallion hierarchy, one for landing raw data, and one for the metastore of Unity Catalog:

<img width="1224" height="558" alt="adls1" src="https://github.com/user-attachments/assets/d2d25570-79f5-4ac6-9218-356502e2e32a" />

### 1.2 Incremental Loading Setup 🗂️

To create a watermark method for incremental loading in Azure Data Factory, an empty csv dataset with only header was uploaded to the Landing Container:

<img width="1233" height="430" alt="adls solo empty" src="https://github.com/user-attachments/assets/f7ce7e7a-53ff-4a88-aefc-8fdf50eb4804" />












