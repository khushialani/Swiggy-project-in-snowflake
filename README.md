food-aggregator-snowflake/
├── data/
│   ├── orders.csv          # Sample order data
│   ├── customers.csv       # Sample customer data
├── scripts/
│   ├── data_pipeline.py    # Main ETL/ELT pipeline script
│   ├── snowflake_utils.py  # Utility functions for Snowflake operations
├── models/
│   ├── fact_orders.sql     # Fact table creation SQL
│   ├── dim_customers.sql   # Dimension table creation SQL
├── dashboard/
│   ├── app.py              # Streamlit dashboard code
├── config.py               # Configuration file for Snowflake credentials
├── requirements.txt        # Python dependencies
└── README.md               # Project documentation
# config.py
SNOWFLAKE_CONFIG = {
    "user": "your_user",
    "password": "your_password",
    "account": "your_account",
    "warehouse": "your_warehouse",
    "database": "your_database",
    "schema": "your_schema"
}
import snowflake.connector

from config import SNOWFLAKE_CONFIG

def get_snowflake_connection():
    return snowflake.connector.connect(
        user=SNOWFLAKE_CONFIG["user"],
        password=SNOWFLAKE_CONFIG["password"],
        account=SNOWFLAKE_CONFIG["account"],
        warehouse=SNOWFLAKE_CONFIG["warehouse"],
        database=SNOWFLAKE_CONFIG["database"],
        schema=SNOWFLAKE_CONFIG["schema"]
    )

def execute_query(query, params=None):
    conn = get_snowflake_connection()
    try:
        cursor = conn.cursor()
        cursor.execute(query, params)
        conn.commit()
    finally:
        conn.close()
import pandas as pd
from snowflake_utils import execute_query

def load_data_to_stage(file_path, stage_name):
    query = f"""
    PUT file://{file_path} @{stage_name}
    """
    execute_query(query)

def load_csv_to_table(stage_name, table_name):
    query = f"""
    COPY INTO {table_name}
    FROM @{stage_name}
    FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"' SKIP_HEADER = 1);
    """
    execute_query(query)

if __name__ == "__main__":
    # Load Orders
