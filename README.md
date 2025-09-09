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

### 2. Azure Data Factory (ADF) 🔄

<img width="1916" height="282" alt="LOGOADF" src="https://github.com/user-attachments/assets/5b73cd7c-52b4-414b-91f1-f46fb7746443" />

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

1) ADLS_pipeline01_table:
Targets the desired location for the table and uses the following dynamic content for the path:

To define local development resources (Execution or real cluster will use different settings)
    
    @concat('pipeline01/', dataset().table_name)

To ensure all required libraries were available to the Python container:
    

<img width="982" height="460" alt="dataset1" src="https://github.com/user-attachments/assets/55572880-fe09-40f6-8c5c-e3b81bb4750f" />



<img width="353" height="385" alt="datrasets" src="https://github.com/user-attachments/assets/665422f3-08f4-4de4-8c54-97742ae62a4c" />

















----------------------------------------

### 3. Apache NIFI 🔄

![LOGO2](https://github.com/user-attachments/assets/28c4facd-dbc4-42b6-a54e-31bbdf0e3e68)


### 3.1 Objective🔄

The main goal of each Process Group in NIFI is:  

1) Read the desired source table from MySQL  
2) Write it to the ADLS Landing Bucket on parquet format. 
3) Simultaneously generate a log table with the path of the most recently written table  
  
NIFI doesn´t support writes on parquet or delta format, so this aproach emulates a Delta-like method, enabling PySpark scripts to identify the current table version between multiple batches.

### 3.3 Creating Process Group and Controller Services🔄

To read MySQL and write tables on Avro format, the following Controller Services need to be added and enabled to the Process Group:  

1) AvroReader   
2) AvroRecordSetWriter   
3) DBCPConnectionPool

After confirming that nifi container was running, the NIFI web interface was accessed via https://localhost:8443/ 

The Controller Services were configured by creating a new Process Group > Entering the Process Group > Opening the Controller Services menu > Adding and enabling each Controller Service.

![nifi3](https://github.com/user-attachments/assets/2dc0a4a8-c567-4a79-bebc-9acc62ee1c4f)

![4](https://github.com/user-attachments/assets/964f5a42-1719-4f21-8042-6c73516af2c1)

![nifi7](https://github.com/user-attachments/assets/0f02aec3-fea0-481e-88b3-a62793ccd891)

![nifi6](https://github.com/user-attachments/assets/9b3f01e7-7696-4a39-a9b5-9922db5ba5ec)


### 3.4 Adding the Processors to the Process Group 🔄

To execute the reads and writes, the following Processors were added to the Process Group in order:

1) **GenerateFlowFile:** Triggers the pipeline with an empty file  
2) **UpdateAttribute:** Sets table name  
3) **ExecuteSQL:** Runs Query to extract the table from MySQL, uses the controller DBCPConnectionPool  
4) **UpdateAttribute:** Sets HDFS location paths  
5) **UpdateAttribute:** Names the write´s path and folder, with the execution timestamp  
6) **ConvertRecord:** Converts table to Avro format, uses the controller AvroReader and AvroRecordSetWriter  
7) **PutHDFS:** Writes Avro Table on the HDFS path  
8) **UpdateAttribute:** Sets filename for the log table  
9) **ReplaceText:** Generates log table, adding a row with the name and time of the last batch   
10) **PutHDFS:** Writes log table

![nifi8](https://github.com/user-attachments/assets/4ee2e2af-5ed8-4ba0-bc3f-f0a490f76c19)

(The Repository File [Process_Group_Sample.json](https://github.com/arinrohega/DE01-Pipeline01-ApacheStack-DeltaLake/blob/main/Nifi%20Process%20Groups/Process%20Group%20Sample.json) it´s the same Process Group and can also be imported)

To read the 10 source tables, the Process group was replicated 10 times, each time configuring the properties of the 2nd Processor "**UpdateAttribute:**" with the table name.

![nifi99](https://github.com/user-attachments/assets/4bb94f55-f428-42b6-a5a8-1db0456dda1e)


### 3.4 Testing the Process Groups 🔄

The current Staging bucket on HDFS was empty:  

![hue vacia](https://github.com/user-attachments/assets/f5dd8149-81ab-4229-83e3-67575a8d0896)
The process group for the table "Zonas" was tested by clicking on the "play" button:  

![nifi play](https://github.com/user-attachments/assets/4a5880d9-958a-4939-bb5f-03ccbcab7297)  

After a few seconds, the folder for the table "Zonas" was created on HDFS:  

![hue vllena1](https://github.com/user-attachments/assets/6939b07f-5160-4926-aa6b-8f3205a58621)

Also the Avro Table and the log with the last write were created inside the folder:

![hue vllena2](https://github.com/user-attachments/assets/d2600fe2-0252-45c3-8f6f-f1bd5c037c4d)


The remaining Process Groups were tested as well by clicking "play"

![nifi91111](https://github.com/user-attachments/assets/103ac222-82ba-474f-9629-5cb78a223ad3)

A folder for each source table were created:

![nifi vllena34](https://github.com/user-attachments/assets/10d17948-4623-4d28-81e0-fc3187d995cf)


After testing out the execution, all source tables were properly written to HDFS, meaning the Process Groups are ready to be automated after. 

### 4. Apache Spark ⚡










