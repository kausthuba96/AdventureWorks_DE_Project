# Adventure Works Data Engineering Project

## Project Overview
This project focuses on implementing a Data Engineering pipeline using Azure Cloud services. The architecture follows a structured approach to ingest, process, and store data efficiently.

## Architecture Blueprint
### Data Flow:
1. **Data Source:** HTTP file (APIs) from Github Repo
2. **Orchestration Tool:** Azure Data Factory
3. **Landing Layer (Bronze Layer):** Data Lake Gen2 (Raw Data Store)
4. **Processing Engine:** Databricks (Spark Clusters)
5. **Serving Layer:** Synapse Analytics (Data Warehousing)

## Prerequisites
### Data Structure
- **Fact Table:** Independent table (Hash Distribution)
- **Dimension Table:** Dependent table (Replicated Distribution)
- **Project Tables:**
  - `AdventureWorks_Returns`, `AdventureWorks_Sales` → Fact tables
  - Other `.csv` datasets → Dimension tables

## Implementation Steps
### Step 1: Dynamic Data Copy using Azure Data Factory
- Use `Foreach` activity for dynamic pipeline execution
- Define source and sink datasets using dynamic parameters
- Create a `git.json` file in VS Code for parameterized data ingestion
- Use `Lookup Activity` to fetch dynamic paths for each dataset
- Implement copy activity inside `Foreach` loop for iterative execution

### Step 2: Configuring Azure Data Lake and Databricks
- **Setup Authentication:**
  - Create App Registration in Microsoft Entra ID
  - Generate `Application ID`, `Directory ID`, and `Client Secret`
  - Assign Role-based Access Control (RBAC) for Data Lake
- **Databricks Integration:**
  - Use `spark.read.format(‘csv’)` to load data
  - Apply schema inference and transformations

```python
from pyspark.sql.functions import *
from pyspark.sql.types import *

df = spark.read.format('csv').option('header', True).option('inferSchema', True).load('abfss://<container>@<storage>.dfs.windows.net/<sink_folder>')
df = df.withColumn('Month', month(col('date')))
df.write.format('parquet').mode('overwrite').save()
```

## Step 3: Aggregation & Data Storage
- Perform aggregations using Spark

```python
df.groupby('OrderDate').agg(count(col('OrderDate')).alias('NoOfOrders'))
```

### Step 4: Using Azure Synapse Analytics
- **Components:**
  - Data Factory, Spark Clusters, SQL-based Data Warehousing
- **Synapse Setup:**
  - Use **Managed Identity** for authentication
  - Assign appropriate roles for access control
- **SQL Pools:**
  - **Serverless SQL Pool:** Stores metadata in Data Lake, queried using T-SQL
  - **Dedicated SQL Pool:** Stores actual data in a structured database

```sql
SELECT * FROM OPENROWSET(
    BULK 'https://<storage>.dfs.core.windows.net/<path>/data.parquet',
    FORMAT = 'PARQUET'
) AS [result];
```

## Conclusion
This project successfully implements a scalable Data Engineering solution leveraging Azure Cloud services, optimizing data ingestion, processing, and analytics.

Connection to the Datalake from Databricks 
Create app registration in the Microsoft Entra ID
Application ID
Directory ID
Click on app on Microsoft Entra ID → Certificates → create a new client secret cert 
Value
On the datalake (Access control IAM) → Create a role assessment → App(name) 


