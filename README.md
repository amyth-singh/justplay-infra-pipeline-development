# Part 1 - Event-Driven Architecture Pipeline

#### Resource and Background :
Retrieve the math performance dataset from [DataSet](https://archive.ics.uci.edu/dataset/320/student+performance). The metadata is available in the 'student.txt' file.

Let's suppose the math performance dataset is obtained through an automated evaluation system. This data arrives on a scheduled basis (e.g., daily, weekly, monthly) and requires processing to generate student-based metrics. The objective is to read the dataset and populate the data into an RDBMS. The choice to insert the data into an open-source RDBMS is optional.

## Solution :
![Screenshot 2024-04-23 164807](https://github.com/amyth-singh/justplay-infra-pipeline-development/assets/78929302/c29b0023-11fb-48e8-9b4c-a69591ad16c3)

- Given incoming data in CSV format, the solution manages multiple input methods such as manual uploads, bulk uploads, or scripted extractions via pipelines. It's built to mimic a local event-driven architecture but is adaptable to any cloud platform, allowing for serverless, trigger-based, and automatic scaling capabilities. - Within the repository, it supervises an ```input_csv``` folder, where CSV files are expected. Upon their arrival, it initiates an automated process involving extraction, validation, and pre-processing, including removing null rows, standardizing values, and field names, and changing delimiters.
- Each CSV file is individually checked against the schema defined in ```schema.yaml``` for compliance. Validated files enter a rule-based system: _compliant_ ones are converted to Parquet and moved to ```output_parquet```, while _failures_ go to ```output_failed``` within input_csv for manual review.
- Successful Parquet files trigger an automated pipeline to upload data to a ```MySQL``` database, using credentials from ```config.yaml``` and schema from ```schema_sql.yaml```.
- Ideally, this pipeline would extend to a data warehouse like BigQuery for analytics and to an object store such as Google Cloud Storage or S3 for broader access. However, for project purposes, files are uploaded to a local database for easier viewing and SQL query execution.

> [!NOTE]
> View ```conversion_log.txt``` log details.

> [!IMPORTANT]
> Watch the 'how-this-works.mp4' video attached to the repository to see a demo.
> Note: some file names may not be the same as in the doc. (e.g ```db_creds.yaml``` aka ```config.yaml```)

### Instructions on How-To-Use:	
<p align="center">
<img src="https://github.com/amyth-singh/justplay-infra-pipeline-development/assets/78929302/06bbb073-2305-4c88-8ee0-97bed201eddd" alt="main.py" style="width:600px;"/>
</p>
<br>

> [!IMPORTANT]
> Please clone the entire repository first.
> If the folders ```input_csv```, ```output_failed```, and ```output_parquet``` do not exist after cloning, ensure to create them locally.
> To start the entire pipeline, run the ```main.py``` file
> When you see the message 'Waiting for CSV files', start dopping the files into the 'input_csv' bucket 

```main.py```
- This serves as the primary Python script containing all functionality, methods, and features. To utilise this file, begin by replacing instances of "student_data" with your desired table name.
- Ensure you create the corresponding table in your MySQL database for seamless script execution.
- Additionally, install all required import modules to enable smooth operation of the main.py script. 

Here's the list of import modules : 
```python
import pandas as pd
import yaml
import time
import os
import logging
import mysql.connector
from datetime import datetime
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler
from sqlalchemy import create_engine, text
```

```input_csv``` 
- CSV files placed in this folder are monitored.
- You can deposit bulk or individual CSVs here to activate the pipeline.
- After processing, CSVs present in this folder automatically get deleted as part of a data management lifecycle.

```output_failed```
- This folder is expected to reside within the 'input_csv' directory.
- It serves as the destination for CSV files that FAIL the validation check.
- Please review these files manually for errors and re-upload them to the input_csv folder.

```output_parquet```
- It serves as the destination for CSV files that PASS the validation check and compress to Parquet files.

```conversion_log.txt```
- A text file that records events, errors, actions, and messages generated by the system.

Example file contents :
```markdown
2024-04-24 01:46:21,111 - INFO - Watching input CSV folder...
2024-04-24 01:48:09,676 - INFO - DataFrame successfully loaded into MySQL table 'student_data'
2024-04-24 01:48:09,772 - INFO - DataFrame successfully loaded into MySQL table 'student_data'
2024-04-24 01:48:51,519 - INFO - Watching input CSV folder...
2024-04-24 01:49:04,051 - INFO - Converted input_csv\student-mat_copy31.csv to Parquet format
```

```schema_sql.yaml```
- A YAML configuration file defining the schema for the database table. In this case, for a table named "student_data"
- Within this file, each entry specifies a column name (name) alongside its associated data type (type)
- When creating the table in the MySQL database, the column data types are derived from this file

Example file contents :
```python
student_data:
  - name: school
    type: VARCHAR(10)

  - name: sex
    type: VARCHAR(1)

  - name: age
    type: Integer

  - name: address
    type: VARCHAR(100)
```

```schema.yaml```
- A YAML configuration file containing a list of field names as key-value pairs.
- Each key represents a column name, and its corresponding value specifies the data type of that column.
- During CSV schema validation, the main.py script compares CSV fields with this file to guarantee uniformity.

Example file contents :
```yaml
school: VARCHAR(10)
sex: VARCHAR(1)
age: INTEGER
address: VARCHAR(100)
famsize: VARCHAR(10)
pstatus: VARCHAR(10)
```

```config.yaml``` aka ```db_creds.yaml```
- A YAML configuration file containing database connection details.
- It specifies the host address, username, port number, password, and database name required to establish a connection to the database.
- When constructing the table in the MySQL database, the main.py script utilises these fields to establish a secure connection with the database.

Example file contents :
```yaml
database:
  host: add_ip
  user: add_mysql_username
  port: add_port
  password: add_mysql_password
  database: add_mysql_database_name
```

## For successful processing within the pipeline, the CSV must adhere to the following guidelines:
- The field names in the CSV must precisely match those specified in the 'schema.yaml' file. If any fields are added or removed, ensure consistency between the CSV and the schema file.
- Ensure that the table name specified in the 'schema_sql.yaml' file aligns consistently with the provided 'table_name' in main.py and the corresponding MySQL table (e.g., "student_name")
- Presently, the main.py script substitutes the ```;``` delimiter with a comma ```,```. If the CSVs being ingested contain a different delimiter, the script will encounter an error. Thus, it is necessary to replace ```;``` with the delimiter identified in the ingested CSVs. 

## Answering Requirements :
1. The solution should be easy to reproduce and automate across all stages: data collection, preparation, modeling, and presentation.

<table><tr><td>

<i>During the workflow of the solution, every step is meticulously documented and recorded in a ```conversion_log.txt``` file, facilitating documentation and debugging. Furthermore, each stage is designed to be accessible and reproducible, enabling seamless replication across various environments. Additionally, the utilisation of trigger-based and rule-based integrations ensures automation, consistency, and reliability throughout the entirety of the solution's lifecycle.</i>
</td></tr></table>

2. It should handle potential data quality issues like missing data.

<table><tr><td>
<i>Presently, the solution addresses basic data quality issues including missing values, null values, schema mismatches, and incorrect delimiters. However, there is potential for the solution to evolve and tackle more complex challenges such as data type inconsistencies, data formatting errors, duplicate entries, incomplete datasets, outlier and anomaly detection, and data loss prevention over time. To maintain efficiency and simplicity, the solution currently focuses on managing fundamental data quality concerns.</i>
</td></tr></table>

3. The solution should follow good data management practices, ensuring accessibility for various user profiles (e.g., scientists, business stakeholders).

<table><tr><td>
<i>The pipeline currently checks, compresses and stores valid CSVs into a folder. In an ideal scenario, this folder would be a cloud object store where access control, data security, data retention and other lifecycle management processes can be ensured. At the moment the solution incorporates some aspects of good data management practices like data validation, logging, automation, schema management, error handling and data disposal.
</td></tr></table></i>
  
4. Provide a way to serve and visualise the data, dashboards and/or plots should be runnable on open-source software, both locally and on the system.
<table><tr><td>
<i>At present, the data loads into a MySQL database, serving as a data source for any data visualization. Expanding data routing to additional storage layers, such as a cloud data warehouse like BigQuery, is straightforward. By simply incorporating a function that exports the dataframe to a table in BigQuery, this integration can be achieved seamlessly.

Alternatively, you can create another .py file that establishes a connection to the MySQL database, imports the necessary data visualization modules, and conducts analysis.
</td></tr></table></i>

## Alternative Scenarios :
What could be done if data volume increases 100x?

<table><tr><td>
Each original CSV file measures 56 KB. Following compression and conversion, each resulting Parquet file is reduced to 17 KB, representing a compression rate of approximately 69.64%. The entire conversion process of 999 CSV files concluded within 10.90 seconds, equating to an average conversion time of 0.01091 seconds per file. Scaling up by 100x would involve processing 99,900 files, requiring an estimated duration of 1088.109 seconds or approximately 18.13515 minutes. To ensure the system maintains efficiency and reliability as it scales, optimising data processing for parallelisation and distributed computing is paramount. Implementing several other measures such as elastic scaling capabilities, multi-node fault-tolerant storage, automated resource allocation, serverless deployment of the solution can help handle data volume, and finally, having more robust data quality and validation rules.
</td></tr></table>

What could be done if data is delivered frequently at 6am every two days?

<table><tr><td>
The solution is crafted with event streaming as its core focus. Deploying it onto a function-as-a-service platform like 'Google Cloud Functions' or 'AWS Glue' enables responsiveness to inbound or manual data drops of varying frequencies, recency, and volumes. If cloud deployment is not viable, operating the solution locally via a 'cronjob' facilitates a recurring scheduling mechanism as well.
</td></tr></table>

What could be done if the data has to be made available to a bigger organisation of 1000+ people?

<table><tr><td>
As scalability becomes a priority, transitioning the solution to robust cloud services such as Google DataProc, Google DataFlow, or similar big data processing platforms becomes essential. These services leverage frameworks like Apache Spark, Apache Beam, or Apache Flink to handle large-scale data processing efficiently. Additionally, employing scalable database services such as Amazon RDS or Google Cloud SQL becomes necessary to support a larger user base. Establishing access control, governance, compliance policies, performance monitoring, and self-service and data visualisation solutions should also be considered to ensure the smooth operation and management of the system.
</td></tr></table>

# Part 2 - SQL
Use the data in the RDBMS from part 1 and write SQL quries to answer the following :

1. List of unique “mother’s job” for male students younger than 20 years old.

```mysql
SELECT DISTINCT mjob
FROM just_play_db.student_data
WHERE sex = 'm' AND age < 20 AND mjob IS NOT NULL;
```
|mjob    |
|--------|
|services|
|other   |
|health  |
|teacher |
|at_home |

2. Most frequent “travel time” among students that live in rural areas

```mysql
SELECT traveltime, COUNT(*) AS count
FROM just_play_db.student_data
WHERE address = 'R'
GROUP BY traveltime
ORDER BY count DESC
LIMIT 1;
```
|traveltime|count|
|----------|-----|
|1         |35   |

3. Top 3 “father’s job” for students grouped by parent’s cohabitation status.
   
```mysql
SELECT s.pstatus, s.fjob, s.job_count
FROM (
    SELECT pstatus, fjob, job_count,
           ROW_NUMBER() OVER (PARTITION BY pstatus ORDER BY job_count DESC) AS rn
    FROM (
        SELECT pstatus, fjob, COUNT(*) AS job_count
        FROM just_play_db.student_data
        GROUP BY pstatus, fjob
    ) AS counted_jobs
) AS s
WHERE s.rn <= 3;
```
|pstatus|fjob      |job_count|
|-------|----------|---------|
|a      |other     |23       |
|a      |services  |7        |
|a      |teacher   |5        |
|t      |other     |194      |
|t      |services  |104      |
|t      |teacher   |24       |

4. Most frequent “class failures” label grouped by family sizes.
   
```mysql
SELECT s.famsize, s.failures, s.class_fail_count
FROM (
	SELECT famsize, failures, class_fail_count,
		ROW_NUMBER() OVER (PARTITION by famsize ORDER BY class_fail_count DESC) AS rn
	FROM (
		SELECT famsize, failures, COUNT(*) AS class_fail_count
        FROM just_play_db.student_data
        GROUP BY famsize, failures
    ) AS counted_count
) AS s
WHERE rn = 1;
```
|famsize  |failures  |class_fail_freq|
|---------|----------|---------------|
|gt3      |0         |222            |
|le3      |0         |90             |

5. Median “absences” for average and low family relationship qualities, group by sex.

```The query provided calculates the median "absences" for individuals with average and low family relationship qualities, grouped by sex. It ensures that the median calculation is accurate by considering both odd and even counts of absences within each group.```

```mysql
SELECT sex, famrel, 
    CASE
        WHEN COUNT(*) % 2 = 1 THEN AVG(absences)
        ELSE (SUM(absences) / 2)
    END AS median_absences
FROM (
    SELECT sex, famrel, absences,
        @rn := IF(@prev_sex = sex AND @prev_famrel = famrel, @rn + 1, 1) AS rn,
        @prev_sex := sex,
        @prev_famrel := famrel
    FROM 
        (SELECT *, (@rn := 0) FROM just_play_db.student_data WHERE famrel <= 3 ORDER BY sex, famrel, absences) sorted,
        (SELECT @prev_sex := NULL, @prev_famrel := NULL) init
) ranked
WHERE 
    rn = CEIL((SELECT COUNT(*) FROM just_play_db.student_data WHERE famrel <= 3 AND sex = ranked.sex AND famrel = ranked.famrel) / 2)
    OR rn = FLOOR((SELECT COUNT(*) FROM just_play_db.student_data WHERE famrel <= 3 AND sex = ranked.sex AND famrel = ranked.famrel) / 2) + 1
GROUP BY sex, famrel
ORDER BY sex, famrel;
```
|sex|famreal|median_absences|
|---|-------|---------------|
|f  |1	    |14             |
|f  |2	    |15             |
|f  |3	    |2              |
|m  |1	    |5              |
|m  |2	    |3              |
|m  |3	    |2              |

# Part 3 - Infrastructure
_note - this is a design exercise, no code implementation is needed._
You will be responsible for designing and implementing a data ingestion pipeline for telemetry data generated by the mobile games. This data includes information such as player actions, level progression, and in-app purchases.
The goal of this exercise is to demonstrate your ability to design and implement a scalable, reliable, and maintainable data ingestion pipeline using modern big data technologies, along with a data visualisation layer for stakeholders.

### Requirements :
Provide schematics, explanations and reasoning for the above use-cases.

## Solution :
- Mobile Client SDK:
	- Captures player actions, purchases, and stores them temporarily (buffering).
	- Bundles events for efficient network transfer (batching).
- Data Collector (Flexible):
	- Option 1 (Direct): Game sends events directly to a streaming service (e.g., Kafka). (Efficient but less flexible)
	- Option 2 (Backend API): Game sends events to a central API for validation, transformation, and forwarding to streaming service. (More flexible, requires modifying the API)
- Streaming Service (Apache Kafka):
	- Handles high-volume real-time data streams reliably.
- Data Processing (Apache Spark):
	- Cleans, enriches, and validates data received from Kafka.
- Data Storage (Data Lake):
	- Stores all processed data for historical analysis (e.g., Amazon S3, Google Cloud Storage).
- Optional Data Warehouse:
	- Faster querying and analysis (populated from the data lake with tools like Apache Hive/Impala).
- Data Visualization:
	- Tools like Tableau or Power BI create dashboards for key metrics (engagement, level completion, revenue).

## Reasoning :
- Scalability: Kafka and Spark can handle increasing data volumes as the game's user base grows.
- Reliability: Kafka ensures data delivery even during network issues. Data buffering on the mobile client and in Kafka provides additional reliability.
- Maintainability: The modular design allows for independent development and maintenance of each component. The backend API approach allows for easier integration with different streaming services without modifying the game client.
- Flexibility: The data lake allows for storing all telemetry data without pre-defining a schema, enabling future analysis based on evolving needs.

## Schematics :

```python
+--------------------+         +--------------------+         +--------------------+         +--------------------+
| Mobile Game Client |         | Data Collector     |         | Streaming Service  |         | Data Processing     |
+--------------------+         +--------------------+         +--------------------+         +--------------------+
                   |             | (Direct Integration)           |             | (Apache Spark)     |
                   |             | OR                             |             |                    |
                   v             v                               v             v                    |
+--------------------+         +--------------------+         +--------------------+         +--------------------+
| Backend API       |         | (RESTful API)       |         | Kafka              |         | Data Lake (S3/GCS) |
+--------------------+         +--------------------+         +--------------------+         +--------------------+
                                 | (Stores Telemetry Events)        |
                                 v                                 v
                             +--------------------+               +--------------------+
                             | Data Warehouse     |               | Data Visualization  |
                             +--------------------+               +--------------------+
                                 (Optional)                        (Tableaux, Power BI)
```

### Additional Considerations :
- Security: Implement data encryption at rest and in transit to protect sensitive player information.
- Data Compression: Compress data before storage to reduce storage costs.
- Monitoring: Monitor the health of the pipeline components to identify and address any issues promptly.

This design provides a scalable, reliable, and maintainable data ingestion pipeline for mobile game telemetry data. The data visualisation layer empowers stakeholders with actionable insights to improve the game and player experience.
