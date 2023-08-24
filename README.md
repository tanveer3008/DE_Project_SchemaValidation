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

	CREATE TABLE [FileDetailsFormat](
	[FileNo] [int] NOT NULL,
 
	[FileName] [nvarchar](100) NOT NULL,
 
	[ColumnName] [nvarchar](100) NULL,
 
	[ColumnDateFormat] [nvarchar](108) NULL,
 
	[ColumnIsNull] [nvarchar](100) NULL,
 
	[ModifiedDate] [datetime] NOT NULL
 
	) ON [PRIMARY]

	GO

This will create a table name FileDetailsFormat  in primary filespace

Run the below script to insert a data

	INSERT [dbo].[FileDetailsFormat] ([FileNo], [FileName], [ColumnName], [ColumnDateFormat], [ColumnIsNull], 
 
 	[ModifiedDate]) VALUES (1, N'Product', N'StartDate', N'MM-dd-yyyy', N'true', CAST(N'2023-06-18T22:34:09.000'
  
  	AS DateTime))
   
	INSERT [dbo].[FileDetailsFormat] ([FileNo], [FileName], [ColumnName], [ColumnDateFormat], [ColumnIsNull],
 
 	[ModifiedDate]) VALUES (1, N'Product', N'EndDate', N'MM/dd/yyyy', N'true', CAST(N'2023-06-18T22:34:09.000' 
  
  	AS DateTime))
   
	INSERT [dbo].[FileDetailsFormat] ([FileNo], [FileName], [ColumnName], [ColumnDateFormat], [ColumnIsNull], 	
 	[ModifiedDate]) VALUES (1, N'Product', N'CreateDate', N'MM/dd/yyyy', N'true', CAST(N'2023-06-			
  	18T22:34:09.000' AS DateTime))
	
 	INSERT [dbo].[FileDetailsFormat] ([FileNo], [FileName], [ColumnName], [ColumnDateFormat], [ColumnIsNull], 	
  	[ModifiedDate]) VALUES (1, N'Product', N'ModifiedDate', N'MM/dd/yyyy', N'true', CAST(N'2023-06-			
   	18T22:34:09.000' AS DateTime)) GO

Let's understand what we are doing here.
We hav created table FileDetailsFormat which has table Filename( which will be the name of file in adls gen2 landing folder) , column name ( name of the column) and columnDateFormat( expectedformat of date in that column).
By using insert query we have inserted filename as Product and  4 columns StartDate.EndDate,CreateDate and
ModifiedDate with expected Date Format.
If there is any other file name for example payment(payment.csv) and we have to validate same things for that file,
we can use same insert with changing filename  and fileno.

### Add Secrets in KeyVault:
accesing the passwords or confidential documents using keyvault is the best practice.
for our use case we can store the sqldbpassword and sastoken of adls in key vault.
Open a created Key vault and click on secrets in left panel and add the secrets

<img src="/screenshots/keyvault.png" alt="db">

### Create Secret Scope for DataBricks
to connect from azure databricks to azure key vault to access secrets  we need to create secret scope in databricks
Steps to create secret scope.
1. start azure databricks server.
2. once azure databricks server is started go into homepage of azure databricks.
3. to access the secret scope in databricks add secrets/ceatescope in url.
   
for eg. databricks homepage url- **https://adb-**********.azuredatabricks.net/?o=2032661656888072#**
   
secretscope url - **https://adb-**********.azuredatabricks.net/?o=2032661656888072#secrets/createScope**

 <img src="/screenshots/secretscope.png" alt="secretscope.png">

**DNS Name you can get from keyVault--> createdKeyVault --> DNS name**

**ResourceId  you can get from keyVault--> createdKeyVault --> Properties --> ResourceId**

then click on create to create Secret Scope


### Create  Cluster in databricks:
1.Select Launch Workspace on the Overview page of your Azure Databricks service.
2.Select Clusters **-->** + Create Cluster.
3.Create a cluster name and accept the remaining default settings.
4.Select Create Cluster.

**Note: If you are using Free azure credit of 200$, make sure to create single node cluster with 4 cores
As Azure doesnot allow any vm greater than 4 core in azure free credit**
 <img src="/screenshots/clusterdatabricks.png" alt="secretscope.png">

 Now import the **Project.ipynb** file into databricks.
 Lets understand each cell in that file
 **cell-1**: By using  dbutils.widgets.get('fileName') we are getting filename from azure data factory once it is landed on the adls gen2 storage. 
 once we have received this file then we are removing the extension so that we can compare it with name stored in our db (Product.csv will be changed to Product)

 **Cell-2**: In this cell we are getting required things to connect to db and adls such as sqlpassword,scopedtoken,sastoken etc and assigning it to some variable so we can access those through out the notebook
 **Cell-3**: this code is used to check if a specific Azure Blob Storage container is mounted at a given mount point in a Databricks environment. If it's not already mounted, it mounts the container using a provided SAS token for authentication. If it's already mounted, it simply outputs a message indicating that it is already mounted
 **if not any(mount.mountPoint == landingMountPoint for mount in dbutils.fs.mounts()):**
 This line checks whether a certain mount point (landingMountPoint) is already mounted or not. In Databricks, you can mount external storage solutions (like Azure Data Lake Storage or Amazon S3) to specific mount points. This line is using a list comprehension and the any() function to iterate through the currently mounted points and checks if any of them matches landingMountPoint.

 If there is no storage mounted then next part of the code gets executed.
 It uses dbutils.fs.mount() to mount an Azure Blob Storage container to the specified landingMountPoint. The parameters used are:

source: The source URL of the Azure Blob Storage container. It's constructed using the provided storageContainer and storageAccount variables.
mount_point: The directory in the Databricks filesystem where the Azure Blob Storage container will be mounted.
extra_configs: Additional configurations for the mount operation. it's passing a Shared Access Signature (SAS) token for authentication to the Azure Blob Storage.
The SAS token is obtained from Databricks' secrets store using dbutils.secrets.get(). The token is retrieved using the databricksScopeName and stgAccountSASTokenKey variables as identifiers.
If the mount operation is successful, it prints 'Mounted the storage account successfully'
**Cell-4**: In this cell we are connecting to azure sql db using jdbc(Java Database Connectivity) url , Once connection is succesful we are using read.jdbc function of spark  to retrieve the data from the database and store it in dataframe df. in last line we are just printing the values which we have retrieved from database

**Cell-5**:this code reads a CSV file, filters and selects specific columns, attempts to convert the values in these columns to dates using provided formats, and counts the number of successfully converted date values. It also prints the count of successfully converted date values and the total count of rows in the DataFrame.
**df1 = spark.read.csv('/mnt/landing/'+fileName, inferSchema=True, header=True)** Here we are reading csv file from landing folder , inferschema=True Means it will try to understand the schema of data in csv such as datatype etc and as our data will have column names so we are making header as true so it will  consider first row as column name

**Cell-6:** In this cell we are validating both the conditions for our data. this code segment validates date formats in a DataFrame based on provided format information, flags errors if any column's date format is incorrect, and then moves the file to appropriate folders based on the validation outcome. It uses an errorFlag and errorMessage to keep track of errors and communicate the validation result.

Case1-  No Duplicate rows.
We are calculationg total rows and saving it in totalcount variable and we are calculating disting rows using distinct function pf python , if both values matches then we are making this condition as true passing this case
Case2- To validate this we are getting the column name and column format from our db schema and then by using to_date function we are checking whether incoming data date format matches to the date format mentioned in db.

Once both the scenario passes we are loading the data into staging folder. if any of this fails then data will be loaded into rejected folder.


   


   



