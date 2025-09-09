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

An applicants/prospects dataset that shows the most relevant metrics from their affiliation process, and calculating the date of first purchase which marks their transition to official affiliated status.

## Architecture

<img width="1634" height="940" alt="ETL 2025-Copia de Página-2 drawio" src="https://github.com/user-attachments/assets/b9797819-848e-443c-8492-78a3032e0a59" />



## Data Model
### Complete OLTP Database Model
![datamodel1](https://github.com/user-attachments/assets/cc53a22c-a9ae-44d2-9b59-fb38ac1bdeeb)

### Chart for desired Table
![etlclient drawio](https://github.com/user-attachments/assets/9ec63cda-8cea-4b85-8741-d7cc2f0cb402)

## Datasets Used
All tables were made and loaded locally to MySQL database. While all data is fictitious, each element was designed meticulously to replicate a real enterprise model, mirroring the afiliation schema of a company I've previously worked with, mantaining logical relationships and consistency across tables.

![dataset1](https://github.com/user-attachments/assets/309bd941-b8a0-40a0-ac6c-243f393cd3e7)

Theres a glossary at the end, providing English translations of the table and column names if needed.
















