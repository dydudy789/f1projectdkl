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




**CREATING 'RACE_RESULTS' DATASET IN PRESENTATION LAYER FROM MULTIPLE DATASETS IN PROCESSED LAYER**
![alt text](https://user-images.githubusercontent.com/21047696/244936596-1f309b16-0a15-40fc-8552-4e78341fd277.png)

Steps:
* Joined races and circuits datasets to get race_id, race_year, race_name, race_date, and circuit_location
* Then joined with results, drivers, and contructors datasets to get the remaining variables (e.g. driver_name, driver_number, driver_nationality, team, race_time, points, position, results_file_date)
* Used the merge_delta_data function to

Data from the race_results dataset can be used to create the BBC F1 race results dashbaord: https://www.bbc.com/sport/formula1/drivers-world-championship/standings

![alt text](https://user-images.githubusercontent.com/21047696/244937668-a3c20c0c-e0f0-4a90-b2f9-51d02c840696.png)




**CREATING 'DRIVER_STANDINGS' DATASET IN PRESENTATION LAYER**
![alt text](https://user-images.githubusercontent.com/21047696/244939171-16d2ce34-d06e-47eb-8cfb-9fc6fbec78e6.png)

Steps:
* Grouped the race dataset by race_year, driver_name and driver_nationality to get the total points and number of wins by each driver.
* Use the window function to assign rank to drivers. Partitioned by race_year, driver were ranked by total points and then number of wins in descending order.
* Used the merge_delta_data function to

Data from the driver_standings dataset can be used to create the BBC F1 driver standings dashbaord: https://www.bbc.com/sport/formula1/drivers-world-championship/standings

![alt text](https://user-images.githubusercontent.com/21047696/244937658-60ad6466-431b-47bd-89cd-7a65e1828415.png)



**IMPLEMENTING INCREMENTAL LOAD THROUGH DELTA LAKE MERGE**
Circuits, races, contructors and drivers datasets are full loads as they do not change from race to race. 
Retults, pit_stops, lap_times, qualifying datasets are updated frequently and so incremental load is implemented.

Delta Lake doesn't support partition overwrite mode. But delta lake allows updates on individual records even when only part of of parition information is available.
The merge code checks if the dataset exists and creates the dataset if it doesn't exist. If it exists, it checks if the provided merge condition is matched and updates the record if it matches. Otherwise it inserts the new record. 

![alt text](https://user-images.githubusercontent.com/21047696/245155484-9d155e4d-a133-4678-913a-2a8e1364f7a8.png)
