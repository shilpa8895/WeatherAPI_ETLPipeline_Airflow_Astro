## WeatherAPI_ETLPipeline_Airflow_Astro
This project is an ETL pipeline built with Apache Airflow and managed using Astro (Astronomer CLI). It extracts weather data from the [Open-Meteo API](https://api.open-meteo.com/v1/forecast?latitude=51.5074&longitude=-0.1278&current_weather=true), transforms it into a structured format, and loads it for downstream usage.

#### -------------------------------------------------------------------------------------------------------------
### Data Archietecture 
#### -------------------------------------------------------------------------------------------------------------

1. Trigger: The DAG is scheduled (or triggered manually) via Airflow.
2. Extract: Airflow uses HttpHook to call Open-Meteo API and retrieve raw weather data (JSON).
3. Transform: The JSON is parsed, unnecessary fields are removed, and the structure is made suitable for downstream use.
4. Load: Data is logged, saved to a file (e.g., CSV), or passed to another storage system (e.g., a database).
5. Optional Extensions: The cleaned data can feed into analytics dashboards, ML pipelines, or external systems.

        ┌────────────────────┐
        │    Scheduler       │
        │  (Airflow DAG)     │
        └────────┬───────────┘
                 │
                 ▼
        ┌────────────────────┐
        │   Extract Task     │
        │  (HttpHook - GET   │
        │   Open-Meteo API)  │
        └────────┬───────────┘
                 │
                 ▼
        ┌────────────────────┐
        │   Transform Task   │
        │  (Clean + Format   │
        │   JSON Response)   │
        └────────┬───────────┘
                 │
                 ▼
        ┌────────────────────┐
        │     Load Task      │
        │ (Store or Log data │
        │   in CSV/Database) │
        └────────┬───────────┘
                 │
                 ▼
        ┌────────────────────┐
        │   Downstream Apps  │
        │  (Dashboard / ML)  │
        └────────────────────┘


#### -------------------------------------------------------------------------------------------------------------
### Project Overview
#### -------------------------------------------------------------------------------------------------------------

1. Built using Astro (Astronomer) for easier development and deployment.
2. Uses the HttpHook from Airflow’s provider packages to connect to an external weather API.
3. Loads cleaned weather data into a Postgres table for persistent storage.
4. Logs and visualizations accessible through the Airflow UI.

#### -------------------------------------------------------------------------------------------------------------
### Project Setup
#### -------------------------------------------------------------------------------------------------------------

#### Prerequisites
1. Docker Desktop (Make sure Docker is running)
2. Astro CLI - Install via Homebrew
```
brew install astro
```
#### Run the Project Locally
1. Initialize Astro Project
   ```
   astro dev init
   ```
2. Start the airflow project
   ```
   astro dev restart --no-cache
   ```
#### Set Up Connection in Airflow UI

1. Go to http://localhost:8080 (default Airflow UI).
2. Navigate to Admin → Connections → + Add.
Fill in:

  *.* Postgres 
  ``` plaintext
  Conn Id: postgres_default
  Conn Type: Postgres
  Host: Name of the Postgres container (from docker-compose.yml)
  Login/Password: As per the docker-compose.yml (usually postgres/postgres)
  Port: 5432
  ```

 *.* Open Meteo Api
  ``` plaintext
  Conn Id: open_meteo_api
  Conn Type: HTTP  
  Host: https://api.open-meteo.com
  Leave Login and Password blank
   ```

#### Trigger the DAG

1. In the Airflow UI, navigate to the DAG: weather_etl_pipeline.
2. Click the "play" button to trigger the pipeline.
3. View logs and schema from the UI under the respective task logs.

#### Test Data in PostgreSQL (Optional via DBeaver)
1. Install DBeaver and launch it.
2. Create a new connection:
   ``` plaintext
    Choose PostgreSQL
    Host: localhost
    Port: 5432
    Username/Password: postgres / postgres
   ```
4. Click Test Connection, then Finish.
5. Run the SQL query:
```
SELECT * FROM weather_data;
```
After triggering the DAG multiple times, you should see populated weather records.

#### Deployment (on AWS)
To deploy the ETL pipeline in production with AWS RDS:
1.Create a PostgreSQL Database on AWS RDS.
2. Copy the endpoint URL - Go to Airflow UI → Admin → Connections → Add. 
   Host: Use the RDS endpoint URL

#### -------------------------------------------------------------------------------------------------------------
### Repository Structure
#### -------------------------------------------------------------------------------------------------------------
``` plaintext
ETLAirflow/
├── dags/                      # DAGs (Directed Acyclic Graphs) folder for Airflow workflows
│   ├── etlweather.py          # Main DAG to extract, transform, and load weather data
│   ├── exampledag.py          # Example DAG (for reference/testing)
│   └── requirements.txt       # DAG-specific Python dependencies (optional)
│
├── include/                   # Optional folder for SQL scripts or data templates (currently empty)
│
├── plugins/                   # Custom Airflow plugins (currently unused)
│
├── tests/                     # Folder for unit or integration tests (optional)
│
├── Dockerfile                 # Docker configuration for Airflow image
├── docker-compose.yml         # Docker Compose config to spin up Airflow + Postgres services
├── airflow_settings.yaml      # Airflow-specific configurations (connections, variables, etc.)
├── requirements.txt           # Python dependencies for the whole project environment
├── packages.txt               # System-level packages (for Docker image)
├── README.md                  # Project documentation and setup guide
```




