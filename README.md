# Comparison between text-to-SQL methods

## Team Information (Team ID: 22)

| Name                  | Student ID |
|-----------------------|------------|
| Simranjit Singh Dulku |    |
| Fotis Monanteros      |    |
| Athanasios Markou     |    |

---

This repository contains the official implementation for the project **"Comparison between text-to-SQL methods"**

We present a unified evaluation framework to benchmark compact LLMs (**Qwen2.5-1.5B** and **TinyLlama-1.1B**) on the Text-to-SQL task. The framework emphasizes resource-constrained environments and measures not just accuracy, but also generation latency and cross-dialect executability.

---

## Features

- **Dual-Database Support:** Automatically mirrors datasets between **SQLite** (for schema introspection) and **PostgreSQL** (for production-grade execution).
- **Cross-Dialect Robustness:** Ensure queries work on different RDBMS engines.
- **Multi-Dataset Support:** Integrated with **Spider**, **ATIS**, and **Geography** datasets.
- **Modular Agent:** Flexible wrapper for testing different LLMs.
- **Verification Pipeline:** Automated tools to verify data consistency across database engines.

---

## Prerequisites

Before running the project, ensure you have the following installed:

- **Python 3.10+**
- **Docker & Docker Compose** (for running Postgres locally, otherwise the Colab Notebook is sufficient)

---

## Installation & Setup

### 1. Environment Setup

Create a virtual environment to manage dependencies.

```bash
cd text2sql-project

# Create virtual environment
python3 -m venv .venv

# Activate the environment
# On Windows use: .venv\Scripts\activate
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt
````

---

### 2. Database Configuration

The framework supports dual-engine evaluation. By default, it runs on **SQLite** (files provided in `data/`). To enable **PostgreSQL** evaluation or if you are using a custom instance:

1. Ensure PostgreSQL is installed (or use the Docker setup in Step 3).
2. Update `db/config.py` with your credentials:

```python
# db/config.py
POSTGRES_DB = "text2sql"      # Match the name used in Docker/Local setup
POSTGRES_USER = "user"        # Default user
POSTGRES_PASSWORD = "pass"    # Default password
POSTGRES_HOST = "localhost"
```

---

### 3. Database Infrastructure

Initialize the local SQLite database and start the Dockerized PostgreSQL service.

**Initialize SQLite** (run from the project root):

```bash
python db/init_sqlite.py
```

> **Expected Output:** `✅ SQLite database created and test table inserted.`

**Start PostgreSQL**:

```bash
cd db
docker compose up -d
cd ..
```

---

### 4. Agent Connectivity Check

Verify that the agent can connect to both database engines using the credentials from `db/config.py`.

```bash
python agent/agent.py
```

**Expected Successful Output:**

```text
*** Agent Setup: Connecting Databases ***
✅ Postgresql Status: Connected (Version: PostgreSQL 15.6 (Debian 15...)
✅ Sqlite Status: Connected and test table queried.
*** Agent is fully ready: Both RDBMS Engines are Available ***
```

---

## Data Ingestion & Verification

This project requires loading external datasets (Spider, ATIS, Geography) into the active databases.

### 1. Load Datasets

This script reads raw data, synchronizes schemas, and populates both SQLite and PostgreSQL.

```bash
python -m db.load_dataset
```

<details>
<summary>Click to see Expected Output</summary>

```text
Loading database: geography into postgresql...
  ✅ Database 'geography' loaded successfully.
Loading database: geography into sqlite...
  ✅ Database 'geography' loaded successfully.
Loading database: atis into postgresql...
  ✅ Database 'atis' loaded successfully.
Loading database: atis into sqlite...
  ✅ Database 'atis' loaded successfully.
...
```

</details>

---

### 2. Verify Data Integrity

Ensure that data was transferred correctly and counts match between both engines.

```bash
python -m db.verify_data
```

<details>
<summary>Click to see Expected Output</summary>

```text
🚀 Starting Database Verification...
==================== GEOGRAPHY ====================
--- [Geo Count on POSTGRESQL] ---
Query: SELECT COUNT(*) FROM city;
Result: [(46,)]
...
✅ SUCCESS: All test tables are populated and data counts match between PostgreSQL and SQLite.
```

</details>

---

## Running Experiments

Once the infrastructure is ready, you can run the benchmarking experiments.

### Run Models Sequentially

To prevent GPU memory issues (OOM) on 16GB VRAM cards (e.g., T4), run models one at a time.


```bash
# Run TinyLlama (1.1B)
python experiments/run_experiment_metrics.py \
  --model tinylama \
  --datasets spider atis geography custom \
  --n 100 \
  --results_dir "/content/drive/MyDrive/text2sql_results" \
  --flush_every 1 \
  --gt_timeout 20 \
  --gt_pg_timeout_ms 140000 \
  --pred_timeout 20 \
  --pred_pg_timeout_ms 30000 \
  --pg_timeout_ms 20000
  ```

```bash
# Run Qwen (1.5B)
python experiments/run_experiment_metrics.py \
  --model qwen \
  --datasets spider atis geography custom \
  --n 100 \
  --results_dir "/content/drive/MyDrive/text2sql_results" \
  --flush_every 1 \
  --gt_timeout 20 \
  --gt_pg_timeout_ms 140000 \
  --pred_timeout 20 \
  --pred_pg_timeout_ms 30000 \
  --pg_timeout_ms 20000
  ``` 

---

### Analyze Results

Generate summary statistics and charts comparing model performance.

```bash
python scripts/analyze_results.py
```

```bash
python scripts/resource_metrics_analyze.py
```

---

## Project Structure

- **`agent/`** — Model integration and inference logic  
  - `agent.py`: Diagnostic script to verify database connectivity.  
  - `model_wrappers.py`: Custom wrapper classes (`TinyLlamaWrapper`, `Qwen2Wrapper`) for handling model inference and tokenization.

- **`data/`** — Dataset artifacts  
  - Contains raw `.json` files (Spider, ATIS, Geography) and source `.sqlite` database files.

- **`db/`** — Database management and configuration  
  - `config.py`: Database connection settings (credentials, hosts).  
  - `init_sqlite.py`: Script to initialize the local SQLite environment.  
  - `load_dataset.py`: ETL script to migrate data from SQLite to PostgreSQL.  
  - `verify_data.py`: Utility to compare row counts and validate data integrity between engines.  
  - `docker-compose.yml`: Configuration for the PostgreSQL Docker container.

- **`experiments/`** — Core benchmarking scripts  
  - `run_experiment.py`: Main execution loop that runs the models, executes SQL on both databases, and logs performance metrics.
  - `run_experiment_metrics.py`: Execution loop that runs the models, executes SQL on both databases, and logs performance metrics including resource metrics.

- **`results/`** — Experiment artifacts  
  - Stores generated CSV logs (`results_tinylama.csv`, `results_qwen.csv`).

- **`plots and diagrams/`** — Contains all our generated plots and stats  
  - Stores generated plots and CSV with stats.
    
- **`scripts/`** — Analysis tools  
  - `analyze_results.py`: Script to process CSV logs and generate accuracy and latency visualizations.
  - `resource_metrics_analyze.py`: Script to process CSV logs and generate latency and resource visualizations.
  - `build_custom_db.py`: Utility to construct custom database schemas for testing
  - `generate_questions.py`: Script to procedurally generate synthetic SQL-question pairs.

- **`utils/`** — Helper libraries  
  - `data_utils.py`: Functions for loading prompts and dataset metadata.  
  - `sql_utils.py`: Utilities for SQL parsing, cleaning, and schema retrieval.


---

## Data 
Due to the large data size they will not be commited on GitHub. Below is the shared link on Google Drive.
https://drive.google.com/drive/folders/1d2gAFFK56cWwWES6HgkzIPURLP5yP7yU?usp=sharing

---

## License

This project is licensed under the MIT License.


