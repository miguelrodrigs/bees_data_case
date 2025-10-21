# BEES Data Engineering: Brewery Data Pipeline

## 1. Overview & Stack Choices

The goal of this project is to build an **end-to-end data pipeline** consuming the **Open Brewery DB API** and applying the **Medallion Architecture model (Bronze → Silver → Gold)**.  
The final objective is to deliver a **clean and aggregated analytical view** for BI consumption.

### 1.1 Technical Choices

- **Language:** Python & PySpark  
  - Python was used to consume the API (`requests` library).  
  - PySpark was used for data transformations (Silver & Gold layers), ensuring scalability even if the data volume is currently small.

- **Containerization:**  
  The project uses **Docker** to package the environment (PySpark, Jupyter), ensuring anyone can run it with a single command.

---

## 2. Pipeline Flow (Medallion Architecture)

The pipeline is divided into **three stages**, where data is progressively refined.

### Bronze Layer (Raw Data)
**Action:**  
Fetch paginated data from the API and persist it in its original format (**JSON**).

---

### Silver Layer (Curated Data)
**Action:**  
Read the JSON data, transform and clean it.  
**Output:** Parquet (a columnar format for efficient reading).

#### Transformation Rules (Required Explanation):

- **Partitioning:**  
  Data is saved **partitioned by location** (`country` and `state_province`) to improve BI query performance.

- **Normalization:**  
  Applied `lower()` and `trim()` on key fields (`brewery_type`, `country`, `state_province`) to ensure consistency and avoid duplicates during partitioning and aggregation.

- **Null Handling:**  
  Implemented rules to handle null or empty strings in essential fields, replacing them with a default value such as `'unknown'` to prevent PySpark failures.

---

### Gold Layer (Analytical View)
**Action:**  
Generate the **final analytical view** for BI consumption.  

**Transformation:**  
Aggregation of the number of breweries (`COUNT`) by **type and location**.  
This is the **only business logic** applied in this layer.

---

## 3. Execution Instructions

###  Prerequisites
- Docker  
- Docker Compose  

###  Clone Repository
```bash
git clone https://github.com/miguelrodrigs/bees_data_case

Build and Start Environment (Jupyter)
docker-compose up --build -d

##MONITORING AND ALERTS## Monitoring and alerting should cover operational health (success/failure, execution time, retries), data quality (checking volume, null values, schema, and freshness at each layer), and infrastructure/API issues (resource usage, source API latency). The orchestrator (e.g., Airflow) should trigger immediate alerts on failure or delay (SLA), and critical quality failures should stop the pipeline to prevent bad data. All metrics should be visualized in a dashboard for complete visibility.