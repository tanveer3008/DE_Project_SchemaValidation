# DE_Project_SchemaValidation
E2E DE project in which incoming file is validated , if all validations are passed then it will be place into staging folder else it will be placed into rejected folder. Tools Used - Az Databricks,ADLS, AZ- KeyVault,AZSQL Server

## High Level Project Info.
Internal Application Sends CSV file into ADLS Storage.
Validation needs to be Applied on the incoming CSV file.
Reject the CSV File if:
1. It containt duplicate rows
2.  Date format is accurate

### Architecture Diagram
![main](Architecture.PNG)

Let's Understand the components of architecture diagram.
1. ADLS - Azure DataLake  Storage Gen2:
   ADLS follows DFS (Distributed file system) protocol which is useful for big data analytics and parallel processing.
   Query Performance- When sending a query that is only retrieving a subset of data, with a hierarchical file system      like ADLS Gen2 it is possible to leverage partition scans for data pruning
 2. Azure Data Factory:
    Azure Data Factory (ADF) is the cloud-based Extract, Transform and Load (ETL) and data integration service that       allows you to create data-driven workflows for orchestrating data movement and transforming data at scale.
 3. Azure DataBricks -
    Databricks Azure is a service on the Microsoft Azure cloud that uses Apache Spark to process and analyze large       data workloads.It supports multiple programming languages and libraries for data transformation, exploration and 
    machine  learning.  It also enables collaboration between different data roles and integrates with other Azure 
    platforms.
4. Azure key Vault - Azure Key vault is used to manage your secrets in azure. we can connect azure key vault to          different services of azure
5. Azure SQL Database: Azure SQL Database is a relational database service provided by Azure.

### High Level Understanding of Project
External Systems will place data in ADLS Gen2 Storage in CSV format(landing folder). We will be using stoage trigger of Azure data factory so that if any new files come then pipeline will be executed automatically.
actual logic will be written in the databricks notebook to reject the file if it fails and load the file into rejected folder of ADLS , if all validations are passed , then it will be loaded into staging folder and then it will be used for further analysis. Azure key vault will be used to store all the secrets such as SAS token, sqldb passwoerd, scoped credentials etc

## Step By Step Execution:
### Create ADLS  Gen2 Storage:
Steps:
1. Login to Azure portal
2. search Storage Account and click on create
3. In advance tab , select checkbox of enable hierarichal namespace.
<img src="/screenshots/adls1.png" alt="Hierarichal" title="Optional title">

4. Fill other necessary details and create a ADLS Gen2 Storage.
### Create Azure SQL Database
1. Login to Azure portal
2. search Azure SQL Database and click on create
3. If Server is not created, create SQL Database server and create SQL DB.
4. make sure Allow Azure services and resources to access this server is enable in advance tab
   <img src="/screenshots/sqldb.png" alt="sqldb">
5. Fill necessary details and create SQL Database.
   
### Create Azure Databricks:
 1. Login to Azure portal
 2. search  Azure Databricks and click on create.
 3. select pricing tier as Standard for current project and fill all necessary details.
 4. once all validations are passed click on create

### Create Azure KeyVault:
 1. Login to Azure portal
 2. search  Azure KeyVault and click on create.
 3. Fill Required Details and create KeyVault

### add Schema in SQL DB
We need to create a schema in sql db against which the incoming file will be validated .
Conect to Az SQL DB Server using SQL Server Management Studio (SSMS) Query Editor or click on query Editor in left menu  of Azure SQL db  in Azure portal.
<img src="/screenshots/db.png" alt="db">

Execute Below query to create a table

**CREATE TABLE [FileDetailsFormat](
	[FileNo] [int] NOT NULL,
	[FileName] [nvarchar](100) NOT NULL,
	[ColumnName] [nvarchar](100) NULL,
	[ColumnDateFormat] [nvarchar](108) NULL,
	[ColumnIsNull] [nvarchar](100) NULL,
	[ModifiedDate] [datetime] NOT NULL
) ON [PRIMARY]
GO**

This will create a table name FileDetailsFormat  in primary filespace

Run the below script to insert a data

**INSERT [dbo].[FileDetailsFormat] ([FileNo], [FileName], [ColumnName], [ColumnDateFormat], [ColumnIsNull], [ModifiedDate]) VALUES (1, N'Product', N'StartDate', N'MM-dd-yyyy', N'true', CAST(N'2023-06-18T22:34:09.000' AS DateTime))
INSERT [dbo].[FileDetailsFormat] ([FileNo], [FileName], [ColumnName], [ColumnDateFormat], [ColumnIsNull], [ModifiedDate]) VALUES (1, N'Product', N'EndDate', N'MM/dd/yyyy', N'true', CAST(N'2023-06-18T22:34:09.000' AS DateTime))
INSERT [dbo].[FileDetailsFormat] ([FileNo], [FileName], [ColumnName], [ColumnDateFormat], [ColumnIsNull], [ModifiedDate]) VALUES (1, N'Product', N'CreateDate', N'MM/dd/yyyy', N'true', CAST(N'2023-06-18T22:34:09.000' AS DateTime))
INSERT [dbo].[FileDetailsFormat] ([FileNo], [FileName], [ColumnName], [ColumnDateFormat], [ColumnIsNull], [ModifiedDate]) VALUES (1, N'Product', N'ModifiedDate', N'MM/dd/yyyy', N'true', CAST(N'2023-06-18T22:34:09.000' AS DateTime)) GO**


   



