# 🛒 End-to-End E-Commerce Data Pipeline  
### Built with Snowflake • dbt • Apache Airflow • SLA Alerting  

This project is a **real-world data engineering workflow**: an **E-Commerce Order Monitoring Pipeline** built with **Snowflake**, **dbt**, and **Apache Airflow**.  

The pipeline simulates raw order data, ingests it into **Snowflake**, applies **modular transformations with dbt**, orchestrates tasks using **Airflow**, and finally implements **SLA-driven email alerting for delayed orders**.  

This project reflects the type of **data pipelines used in production environments** for monitoring KPIs, ensuring data quality, and enabling business decision-making.

---

## 📊 Project Overview  

- **Data Simulation** → Generate dummy order, customer, and product data  
- **Data Ingestion** → Load into Snowflake raw schema  
- **Data Transformation** → Clean, standardize, and model data with dbt  
- **Orchestration** → Automate ingestion + transformation workflows with Airflow  
- **Monitoring & Alerting** → SLA checks for delayed orders, with **email notifications**  

---

## 🏗️ System Design  

```mermaid
flowchart TD
    A[Simulated Order Data (CSV)] -->|Ingest| B[Snowflake Raw Schema]
    B -->|dbt Transformations| C[Snowflake Analytics Schema]
    C -->|Monitor & Flag Delays| D[Airflow DAGs]
    D -->|Trigger SLA| E[Email Alerts for Delayed Orders]
````

---

## 📂 Project Structure

```
ecommerce-data-pipeline/
│
├── airflow_dags/                # Airflow DAG definitions
│   └── ecommerce_pipeline_dag.py
│
├── dbt/                         # dbt project folder
│   ├── models/
│   │   ├── staging/             # Standardized raw data
│   │   ├── marts/               # Business logic models
│   │   └── alerts/              # SLA alerting models
│   ├── dbt_project.yml
│   └── profiles.yml
│
├── data/                        # Dummy data
│   └── orders.csv
│
├── airflow_home/                # Airflow config & logs
│   └── airflow.cfg
│
├── scripts/                     # Python scripts
│   ├── generate_orders.py
│   └── load_to_snowflake.py
│
└── README.md
```

---

## 🗃️ E-Commerce ERD

Below is the **Entity Relationship Diagram (ERD)** for the E-Commerce dataset used in this pipeline:

![E-Commerce ERD](./Ecommerce%20ERD.png)

The ERD illustrates the relationships between **customers, orders, products, and deliveries** — the same structure that is simulated, ingested, and transformed in this pipeline.

---

## 📦 Step 1: Dummy Data Creation

We generate order data with Python (`faker`, `pandas`).

Example schema:

* `order_id`, `customer_id`, `product_id`, `order_date`, `status`, `delivery_date`

```python
from faker import Faker
import pandas as pd
import random

fake = Faker()
orders = []
for i in range(1000):
    orders.append({
        "order_id": i+1,
        "customer_id": fake.random_int(min=1, max=500),
        "product_id": fake.random_int(min=1, max=200),
        "order_date": fake.date_this_year(),
        "status": random.choice(["delivered", "pending", "shipped"]),
        "delivery_date": fake.date_this_year()
    })

df = pd.DataFrame(orders)
df.to_csv("data/orders.csv", index=False)
```

---

## ❄️ Step 2: Using Snowflake

* **Database & Schema Setup**

```sql
CREATE OR REPLACE DATABASE ecommerce_db;
CREATE OR REPLACE SCHEMA raw;
CREATE OR REPLACE SCHEMA analytics;

CREATE OR REPLACE TABLE raw.orders (
  order_id INT,
  customer_id INT,
  product_id INT,
  order_date DATE,
  status STRING,
  delivery_date DATE
);
```

* Load CSV into **raw schema** using Snowflake **COPY INTO** or Python Snowflake connector.

---

## 🛠️ Step 3: dbt Transformations

### dbt Folder Layers

* **staging/** → Clean, standardize data
* **marts/** → Business-ready metrics (delayed orders, delivery KPIs)
* **alerts/** → SLA alerting queries

### Example: Flagging Delayed Orders

```sql
-- models/marts/delayed_orders.sql
SELECT
    order_id,
    customer_id,
    product_id,
    order_date,
    delivery_date,
    CASE
        WHEN DATEDIFF(day, order_date, delivery_date) > 7 THEN 1
        ELSE 0
    END AS delayed_flag
FROM {{ ref('stg_orders') }}
```

Run transformations:

```bash
dbt run
dbt test
```

---

## 🐍 Step 4: Python Virtual Environment

```bash
# Create and activate venv
python -m venv airflow_venv
airflow_venv\Scripts\activate     # (Windows)
source airflow_venv/bin/activate  # (Linux/Mac)

# Install dependencies
pip install -r requirements.txt
```

---

## 🌬️ Step 5: Apache Airflow Orchestration

### Initialize Airflow

```bash
export AIRFLOW_HOME=./airflow_home
airflow db init
```

### Example DAG

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.email import EmailOperator
from airflow.utils.dates import days_ago

from scripts.load_to_snowflake import load_data
from scripts.run_dbt import run_dbt_models

with DAG(
    dag_id="ecommerce_pipeline",
    schedule_interval="@daily",
    start_date=days_ago(1),
    catchup=False,
) as dag:

    ingest_task = PythonOperator(
        task_id="load_orders_to_snowflake",
        python_callable=load_data
    )

    transform_task = PythonOperator(
        task_id="run_dbt_transformations",
        python_callable=run_dbt_models
    )

    alert_task = EmailOperator(
        task_id="send_delay_alert",
        to="alerts@company.com",
        subject="🚨 Delayed Orders Alert",
        html_content="<p>One or more orders exceeded the SLA threshold!</p>"
    )

    ingest_task >> transform_task >> alert_task
```

### Airflow UI

```bash
airflow webserver --port 8080
airflow scheduler
```

👉 Access at **[http://localhost:8080](http://localhost:8080)**

---

## 📧 Step 6: SLA Alerts

* SLA thresholds (e.g., >7 days delivery) flagged in **dbt models**
* Alerts configured via **Airflow EmailOperator**

This ensures **real-time monitoring** of operational KPIs.

---

## 🎯 Real-World Relevance

This project helped me **bridge the gap between theory and practice** in Data Engineering by:

*  Building an **end-to-end data pipeline** similar to enterprise workflows
*  Learning to integrate **Snowflake, dbt, and Airflow** into one cohesive system
*  Applying **data modeling principles (ERD, staging, marts, alerts)**
*  Implementing **SLA-driven monitoring**, critical for real-world pipelines
*  Understanding how to **alert stakeholders** automatically when KPIs breach thresholds

By working on this, I learned not just the tools but **how to solve real-world problems in data reliability, monitoring, and automation**.

---

## 📌 Future Enhancements

* Real-time ingestion with **Kafka** or streaming connectors
* Extend alerts to **Slack / Teams** integrations
* Deploy Airflow on **MWAA / Cloud Composer** for production-grade scaling

---

## 🤝 Contributing

Contributions are welcome! Open an issue for discussions, improvements, or bug reports.

---

## 📜 License

This project is licensed under the **MIT License**.

---

## 📬 Contact

* 🌐 [**Portfolio**](https://kibutujr.vercel.app)
* 💼 [**LinkedIn**](https://www.linkedin.com/in/fred-kibutu)
* 📧 [**Email**](mailto:kibutujr@gmail.com)

---
