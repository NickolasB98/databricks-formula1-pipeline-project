# Project Overview:
The goal of this project is to create a data analysis system for Formula-1 race results using Azure Databricks. This involves building an ETL (Extract, Transform, Load) pipeline to collect Formula 1 racing data from ergast.com, a website focused on Formula 1 statistics, and then processing and storing it in Azure Data Lake Gen2 storage. Azure Databricks is used for data transformation and analysis, and Azure Data Factory orchestrates the entire process.

## Formula1 Overview
Formula 1 (F1) is the highest level of single-seater auto racing worldwide and is governed by the FIA. It showcases advanced and powerful cars equipped with hybrid engines. The F1 season occurs once a year and consists of races that take place over the course of a weekend, typically from Friday to Sunday. These races are held at various circuits in different countries. In each race, there are 10 teams, each with two assigned drivers.

The F1 season typically includes 20 to 23 races, also known as Grands Prix. Safety is a paramount concern, with strict regulations and ongoing technological advancements to ensure the well-being of drivers and spectators. Pit stops are a common occurrence during races, allowing teams to change tires and make adjustments to the cars.

On Saturdays, a qualifying round is held to determine the starting positions of drivers on the grid for the Sunday race. The races themselves consist of a variable number of laps, usually ranging from 50 to 70 laps, depending on the circuit's length. Pit stops are available during races for tire changes or car adjustments.

The results of each race are used to calculate driver standings and constructor standings. The driver who leads the driver standings at the end of the season is crowned the drivers' champion, while the team that leads the constructor standings becomes the constructors' champion.

## Solution Architecture for the Problem Statement
<img width="1372" alt="Screenshot at Jun 13 16-02-59" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/c3c05d63-d401-455b-98f8-776756c3a727">

## ER Diagram
The structure of the database is shown in the following ER Diagram and explained in the Database User Guide ER
<img width="1290" alt="Screenshot at Jun 13 16-21-46" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/1ef7c7cf-e903-43d5-bb3a-8c43f2738504">

## Working Plan

### Source Data:
We are referring to open-source data from the website Ergast Developer API. Data was available from 1950 till 2022.
<img width="493" alt="Screenshot at Jun 13 16-22-59" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/88dadf19-efa8-4302-9e13-70ed7a53d4ac">

### Execution Overview:
Azure Data Factory (ADF) plays a crucial role in managing the execution and monitoring of Azure Databricks notebooks. Our workflow involves importing data from the Ergast API and storing it in Azure Data Lake Storage Gen2 (ADLS). Initially, the raw data is placed in the Bronze zone, which serves as a landing zone.

**Bronze Container**
<img width="517" alt="raw container" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/5201791d-6e96-4bef-af5c-5d20925a8278">

To process the data in the Bronze zone, we use an Azure Databricks notebook. This notebook is responsible for transforming the data into delta tables using an upsert operation. Once this transformation is complete, ADF takes charge of moving the processed data to the ADLS Silver zone, which functions as a standardization zone.

In the Silver zone, the ingested data undergoes further transformation through an Azure Databricks SQL notebook. This involves operations such as joining and aggregating tables to prepare the data for analytical and visualization purposes. 

**Silver Container**
<img width="524" alt="processed container" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/46f416cd-a3d9-475f-b80a-376699ab2dfc">

Ultimately, the results of these transformations are loaded into the Gold zone, which serves as the analytical zone for further analysis and reporting.

**Gold Container**
<img width="510" alt="presentation container" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/e5dac411-ca2b-4579-87a7-be1320f70fd8">

### ETL pipeline:
ETL flow comprises two parts:

* Ingestion: Process data from Bronze zone to Silver zone
* Transformation: Process data from Silver zone to Gold zone
In the first pipeline, data stored in JSON and CSV format is read using Apache Spark with minimal transformation saved into a delta table. The transformation includes dropping columns, renaming headers, applying schema, and adding audited columns (ingestion_date and file_source) and file_date as the notebook parameter. This serves as a dynamic expression in ADF.

In the second pipeline, Databricks SQL reads preprocessed delta files and transforms them into the final dimensional model tables in delta format. Transformations performed include dropping duplicates, joining tables using join, and aggregating using a window.

ADF is scheduled to run every Sunday at 10 PM and is designed to skip the execution if there is no race that week. We have another pipeline to execute the ingestion pipeline and transformation pipeline using file_date as the parameter for the tumbling window trigger.

**Ingest Pipeline**
<img width="1420" alt="ingest_pipeline" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/3dd12167-8466-4683-827b-f1c8f600cc4b">

**Transform Pipeline**
<img width="1119" alt="transform_pipeline" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/cd6beb62-d128-4113-b0d2-9cd797cef014">

**Process Pipeline (Contains Both Pipelines and the Trigger)**
<img width="1431" alt="process_pipeline" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/d6e74a9f-486c-4e3d-b918-ad27cf5ad2ef">

**Trigger of the Process Pipeline**
<img width="879" alt="trigger_pipeline" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/30cb9797-16aa-436a-bddd-40ba51c6cee1">


## Azure Resources Used for this Project:
* Azure Data Lake Storage
* Azure Data Factory
* Azure Databricks
* Azure Key Vault

## Project Requirements:
The requirements for this project are broken down into six different parts which are

1. Data Ingestion Requirements
    * Ingest all 8 files into Azure data lake.
    * Ingested data must have the same schema applied.
    * Ingested data must have audit columns.
    * Ingested data must be stored in columnar format (i.e., parquet).
    * We must be able to analyze the ingested data via SQL.
    * Ingestion Logic must be able to handle the incremental load.

2. Data Transformation Requirements
    * Join the key information required for reporting to create a new table.
    * Join the key information required for analysis to create a new table.
    * Transformed tables must have audit columns.
    * We must be able to analyze the transformed data via SQL.
    * Transformed data must be stored in columnar format (i.e., parquet).
    * Transformation logic must be able to handle the incremental load.
  
3. Data Reporting Requirements
    * We want to be able to know Driver Standings.
    * We should be able to know Constructor Standings.

4. Data Analysis Requirements
    * Find the Dominant drivers.
    * Find the Dominant Teams.
    * Visualize the Outputs.
    * Create Databricks dashboards.

5. Scheduling Requirements
    * Scheduled to run every Sunday at 10 pm.
    * Ability to monitor pipelines.
    * Ability to rerun failed pipelines.
    * Ability to set up alerts on failures

6. Other Non-Functional Requirements
    * Ability to delete individual records
    * Ability to see history and time travel
    * Ability to roll back to a previous version

## Visual Analysis
<img width="1035" alt="Dominant Drivers Visualization" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/892ca21e-2572-4860-a83d-da197cea3364">

<img width="1037" alt="Dominant Teams Visualization" src="https://github.com/NickolasB98/databricks-formula1-pipeline/assets/157819544/d0c3f2df-1233-4904-b0cb-47815f83e36e">

## Tasks performed:

• Built a solution architecture for a data engineering solution using Azure Databricks, Azure Data Lake Gen2, Azure Data Factory, and Power BI.
• Created and used Azure Databricks service and the architecture of Databricks within Azure.
• Worked with Databricks notebooks and used Databricks utilities, magic commands, etc.
• Passed parameters between notebooks as well as created notebook workflows.
• Created, configured, and monitored Databricks clusters, cluster pools, and jobs.
• Mounted Azure Storage in Databricks using secrets stored in Azure Key Vault.
• Worked with Databricks Tables, Databricks File System (DBFS), etc.
• Used Delta Lake to implement a solution using Lakehouse architecture.
• Created dashboards to visualize the outputs.
• Connected to the Azure Databricks tables from PowerBI.

### Spark (Only PySpark and SQL)
• Spark architecture, Data Sources API, and Dataframe API.
• PySpark - Ingested CSV, simple, and complex JSON files into the data lake as parquet files/ tables.
• PySpark - Transformations such as Filter, Join, Simple Aggregations, GroupBy, Window functions etc.
• PySpark - Created global and temporary views.
• Spark SQL - Created databases, tables, and views.
• Spark SQL - Transformations such as Filter, Join, Simple Aggregations, GroupBy, Window functions etc.
• Spark SQL - Created local and temporary views.
• Implemented full refresh and incremental load patterns using partitions.

### Delta Lake
• Performed Read, Write, Update, Delete, and Merge to delta lake using both PySpark as well as SQL.
• History, Time Travel, and Vacuum.
• Converted Parquet files to Delta files.
• Implemented incremental load pattern using delta lake.

### Azure Data Factory
• Created pipelines to execute Databricks notebooks.
• Designed robust pipelines to deal with unexpected scenarios such as missing files.
• Created dependencies between activities as well as pipelines.
• Scheduled the pipelines using data factory triggers to execute at regular intervals.
• Monitored the triggers/ pipelines to check for errors/ outputs.

### Technologies/Tools Used:
• Pyspark
• Spark SQL
• Delta Lake
• Azure Databricks
• Azure Data Factory
• Azure Date Lake Storage Gen2
• Azure Key Fault
• Power BI (Optional)
Azure Key Fault
Power BI
