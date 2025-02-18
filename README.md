### Data Engineering Project 
### Azure Cloud

# Project Name : Adventure Works_DE_Project

# Step 1 : Blueprint of Architecture

Data Source : HTTP file (APIs)
Orchestration tool : Azure Data Factory
Landing layer of data (Bronze Layer) : Data lake Gen 2 (Raw Data Store) <– Data from source
Spark clusters : Databricks (Dominating Data Engineering)
Data Lake Gen2 → Databricks
Serving layer : Synapse Analytics (Data Warehousing technology)

# Step 2 : Pre-requisite concept 

Fact table : Independent table (Hash Distribution)
Dimension Table : Dependent table (Replicated Distribution)
Project : AdventureWorks_Returns , AdentureWorks_Sales → Fact table , All the other datasets(.csv) acts as Dimension table

# Dynamic way of Copy activity:

Foreach activity == For loop → Dynamic way if building pipeline
Source → Create a source dataset & relative url = enter add dynamic content → Click ‘+’ → ‘p_relative_url’ → Create 
Follow the same for Sink activity as well 
Create a Json file ‘git.json’ in VS code & Enter the parameterized string for source & sink(destination) (Parameterization means creating one dataset for all the files. Parameterization to be done on the For each activity (inside Iterations & conditions menu)
Create another container in the Azure Datalake Gen2 Storage account → Parameters → Upload git.json 
Lookup activity = gives us to view the output 
Copy paste the Copy activity inside For Each Activity (to perform Iterations)
For source & sink database (add dynamic content with the lookup output) ex: @item().p_relative_url

# Datalake Gen2 Storage Account —>  Key Vault/ Azure Active Directory →  Databricks 

Connection to the Datalake from Databricks 
Create app registration in the Microsoft Entra ID
Application ID
Directory ID
Click on app on Microsoft Entra ID → Certificates → create a new client secret cert 
Value
On the datalake (Access control IAM) → Create a role assessment → App(name) 

# Databricks:
Launch databricks & click on Compute for performing transformations. 
Creating Authentication from databricks to Data Lake → Microsoft Entra ID → App registration
From Microsoft Documentation copy the sync command from data lake Gen2 to Databricks
Spark Command → df = spark.read.format(‘csv’).option(‘header’,TRUE).option(‘inferSchema’,TRUE).load(‘abfss://<container_name>@<Storage_account>.dfs.windows.core.net/<sink_folder_name>
From pyspark.sql.functions import * 
From pyspark.sql.types import * 

# Transformation
df .withColumn(‘Month’,month(col(‘date’))) → To create a new column
Parquet files are columnar file format
Writing the data to the Data Bricks 
df_cal.write.format(‘Parquet’).mode(‘overwrite’)
Modes are 4 types 
append() → Merges the data to the folder
overwrite() → Overwrites the data to the folder
error() → gives an error message if the same data exists
ignore() → doesn’t append nor doesn’t give error message

df_cal.withColumn(‘month’,month(col(‘date’)))
	
     10. df_cal.write.format(‘parquet’).mode(‘append’).option(‘path’,’abfss://silver-kvanam@adventureworks_kvanam.dfs.windows.core.net/AdventureWorks_Calendar).save()

# Aggregation:

11. df_cal.groupby(‘OrderDate’).agg(count(col(‘OrderDate’).alias(‘NoofOrders’)))

# Synapse Analytics

Unified Platform 
Combining : Data Factory + Spark cluster + Data Warehousing
Data Factory embedded into the Synapse Analytics. 
Spark pool is different from Data bricks 

# Synapse analytics window:

Integrate (Azure Data Factory)
Develop ( Spark Pool Cluster different from Azure Databricks) 
Data ( Data warehouse or SQL Database, Lake Database(Spark))
As databricks is an external service, we need key vault (app registrations) in-order to give authorization to access databricks, but for Azure synapse analytics we don’t need any key vault permissions 
Managed Identity (ID card for Azure resources) : Helps to connect to Azure services which are given authentication by Microsoft entra ID 
Assign the role for the Managed Identity
Serverless SQL pool : Data is stored in the form of Tables & columns as a meta data in the data lake and can be queried & copied to SQL Database or any other database with the help of T-SQL commands.. 
Dedicated SQL Pool : Data is stored in the actual traditional database, where we query the data and load the data from the MySQL,PostGreSQL database. 
We use the openrowset() function to ingest data from the data lake to the synapse workspace. 
Same as Synapse we need to assign a role for our user as an Azure blob data contributor.
