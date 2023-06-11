# General Info
This is a data engineering project regarding f1 racing data, assisted by a data engineering course by Ramesh Retnasamy


# Datasets
Ergast API is a third party developer API called Ergaste that makes data available from all F1 races from 1950 onwards. CSV Database Tables were downloaded from the Ergast website. 8 datasets were used for this project. Some were converted to other formats (Single Line JSON, Multi Line JSON, Split CSV files, etc) for practising ingestion of various data types.

Circuits:     CSV
Races:        CSV
Constructors: Single Linke JSON
Drivers:      Single Line Nested JSON
Results:      Single Line JSON
PitStops:     Multi Line JSON
LapTimes:     Split CSV Files
Qualifying:   Split Multi Line JSON Files


# ER Diagram
![alt text](https://user-images.githubusercontent.com/21047696/244870072-1be325de-eee2-45e7-9c54-89ca974da799.png)

# Technologies
**Storage:**
Azure Datalake Gen2
Azure Blob Storage

**Transformations:**
Databricks

**Orchestration:**
Azure Data Factory

# Setup

Save the 8 datasets into Azure Blob Storage 'raw' container. 

Follow the instructions in this link to mount the ADLS to Databricks: https://learn.microsoft.com/en-us/azure/databricks/dbfs/mounts

# Architecture

![alt text](https://user-images.githubusercontent.com/21047696/244872399-d5e08bfa-1bf2-4fc7-b46a-87e8df76302d.png)

Races data is incrementally loaded as races data can get updated each week. Other files are fullly loaded. 

The raw data is then processed using Databricks Notebooks to ingest into the 'processed' layer. The data will have schema applied, and stored as Delta Lake to allow time travel and GDPR (General Data Projection Regulation).

Ingested data is then transformed via Azure Databricks notebooks again, and results are stored in the presentation layer. Power BI is connected to the presentation layer. 

Azure Data Factory will be set as a scheduling solution for this project. 




Other model architectures can be seen here for reference: https://learn.microsoft.com/en-us/azure/architecture/browse/?filter-products=databr&products=azure-databricks


### Processing and Transformations:

Datasets were adjusted (e.g. variables renamed, date variables transformed to timestamp, and ingestion dates added) and put in the 'processed' folder. They were then combined and further transformed and put in the 'presentation' folder. 

**RAW POPULATION FILE**
![alt text](https://user-images.githubusercontent.com/21047696/241617306-0cb8ef26-9fa9-4130-9af8-62b637b0c1ab.png)

**TRANSFORMED POPULATION FILE**

![alt text](https://user-images.githubusercontent.com/21047696/241617300-6328f60b-7088-4b82-9764-a57ffc25620c.png)

Transformations:
* Split columns
* Renamed columns
* Treated missing values as null
* Joined with lookup table to get country name
* Unwanted years removed
* Pivotted table to make age variables as columns instead of values.



### Confirmed cases and hospital admissions data:

ADF tumbling window triggers run everyday at midnight to access the website for data. 
ADF pipeline goes through a lookup list of files to get from the websiite and copies them to data lake. raw file. 
Additional triggers for proccessing in Data flow is invokes when the first trigger is successful. Data is transformed in Data flow and saved in processed file in data lake.
ADF also copies the files into Azure SQL Database so it can be accessed for analysis and dashboarding. 

**EXAMPLE DATA FLOW TRANSFORMATION IN ADF**
![alt text](https://user-images.githubusercontent.com/21047696/241617322-a4300dc1-4b2a-4f14-b986-3dd01e7f848a.png)

**RAW CASES AND DEATHS DATA**
![alt text](https://user-images.githubusercontent.com/21047696/241617336-faff80de-9b76-4880-a60b-5900903adf84.png)

**TRANSFORMED CASES AND DEATHS DATA**
![alt text](https://user-images.githubusercontent.com/21047696/241617347-66fbae1e-2f6a-433d-bbcf-a655c1fff54c.png)

Transformations:
* Filterd for Europe data only
* Selected only required fields and renamed them
* Pivoted the 'indicator' column to make cases_count and deaths_count as columns instead of values

### Testing data:
For practise testing data was transformed in HD Insight.
Transformed files were copied into Azure SQL Database for analysis and dashboarding.
 
 Process within HD Insight Script:
* Create external tables for lookup files (dim_date, dim_country) and raw data (raw testing data). These tables are built on files from Azure Data Lake.
* Also create external table to the destination data, testing data in the processed file.
* Join lookup tables to the raw testing data and apply transformations and INSER TABLE into the destination location.


**EXAMPLE SQL TRANSFORMATION WITHIN HD INSIGHT**
![alt text](https://user-images.githubusercontent.com/21047696/241744814-88916c7c-7e2f-4bc1-9191-edf23407c2b6.png)

# Triggers 
**Population Data:** Events trigger was used to detect the population data landing on Azure Blob Storage to then process the ingestion into data lake, deletion from Blob storage, transformation, and copying to SQL.

**Cases/Deaths Data:**
* Tumbling window trigger is used which runs everyday at midnight to get cases/deaths data from the website and ingest it into Data lake. 
* Tumbling windows triggers for procesing is dependant on the ingestion trigger, and trigger to copying to SQL DB is dependent on the processing trigger. 

# Power BI Dashboard

Power BI directly import the datasets from SQL database using server and database name and the database credentials.

The charts show trend of covid cases, deaths, hospital and and icu occupancy. They are filterable by country and date. 

![alt text](https://user-images.githubusercontent.com/21047696/243354299-173a6f18-4dc2-4394-b6cd-13ee58513cd0.png)

![alt text](https://user-images.githubusercontent.com/21047696/243354367-8c11ad90-5472-4637-bc11-14bf8cfbe947.png)


