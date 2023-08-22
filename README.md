# DE_Project_SchemaValidation
E2E DE project in which incoming file is validated , if all validations are passed then it will be place into staging folder else it will be placed into rejected folder. Tools Used - Az Databricks,ADLS, AZ- KeyVault,AZSQL Server

## High Level Project Info.
Internal Application Sends CSV file into ADLS Storage.
Validation needs to be Applied on the incoming CSV file.
Reject the CSV File if:
1. It containt duplicate rows
2. Validate the Date format is accurate


### Architecture Diagram
![alt text](https://github.com/tanveer3008/DE_Project_SchemaValidation
/blob/main/architecture.jpg?raw=true)
![main](Architecture.PNG)
