# Data Analysis with Python & SQL Server

## About this project

This project demonstrates how to:

* Connect securely to a SQL Server database from Python using `pyodbc` (or `sqlalchemy`).
* Pull raw data via parameterized SQL queries.
* Clean and transform data using `pandas`.
* Perform EDA and compute key performance indicators (KPIs).
* Visualize results using `matplotlib` and `plotly` (optional interactive charts).
* Save transformed datasets locally and (optionally) push cleaned datasets back to the database or to files (CSV / Parquet).

Use cases: sales analysis, product performance, customer segmentation, inventory analysis, and reporting automation.

---

## Repository contents

```
/ (root)
├─ SQL Project.ipynb        # Main Jupyter notebook (analysis + code)
├─ requirements.txt         # Python dependencies
├─ data/                    # (optional) local copies of datasets exported from SQL
│  ├─ raw/
│  └─ processed/
├─ notebooks/               # (optional) auxiliary notebooks
└─ README.md                # This file
```

---

## Environment & Requirements

Recommended Python version: **3.9+**.

Install dependencies (example):

```bash
python -m venv venv
source venv/bin/activate   # macOS / Linux
venv\Scripts\activate     # Windows
pip install -r requirements.txt
```

Example `requirements.txt` (add to repo):

```
pandas
numpy
pyodbc
sqlalchemy
matplotlib
plotly
jupyterlab
python-dotenv
scikit-learn   # optional, for clustering or modeling
seaborn        # optional
```

---

## How to connect to SQL Server

> **Important:** Never store plaintext credentials in the notebook or in the repository. Use environment variables or a `.env` file (excluded from source control) when storing sensitive information.

### 1) Using `pyodbc` (recommended minimal example)

```python
import os
import pyodbc
from dotenv import load_dotenv

load_dotenv()  # loads variables from a .env file

server = os.getenv('SQLSERVER_HOST')
database = os.getenv('SQLSERVER_DB')
username = os.getenv('SQLSERVER_USER')
password = os.getenv('SQLSERVER_PASS')

conn_str = (
    'DRIVER={ODBC Driver 17 for SQL Server};'
    f'SERVER={server};DATABASE={database};UID={username};PWD={password}'
)

conn = pyodbc.connect(conn_str)
```

Add a `.env` file (do **not** commit) with:

```
SQLSERVER_HOST=your_server_name_or_ip
SQLSERVER_DB=your_database_name
SQLSERVER_USER=your_username
SQLSERVER_PASS=your_password
```

### 2) Using `sqlalchemy` (optional)

```python
from sqlalchemy import create_engine

engine = create_engine(
    f"mssql+pyodbc://{username}:{password}@{server}/{database}?driver=ODBC+Driver+17+for+SQL+Server"
)
conn = engine.connect()
```

---

## Notebook structure (`SQL Project.ipynb`)

The notebook is organized into the following logical sections (each as a separate Jupyter cell group):

1. **Setup & imports** — import libraries, load environment variables.
2. **Database connection** — establish connection using `pyodbc`/`sqlalchemy`.
3. **Data extraction** — parameterized SQL queries to pull relevant tables or views.
4. **Initial data inspection** — head, dtypes, missing values, unique counts.
5. **Data cleaning & transformation** — renaming columns, type conversion, handling nulls, calculated fields, joins and aggregations.
6. **Exploratory Data Analysis (EDA)** — summary statistics, distribution plots, time-series analysis.
7. **Business metrics & KPIs** — e.g., revenue, avg order value, churn, retention, inventory turns.
8. **Advanced analysis (optional)** — cohort analysis, segmentation, forecasting, clustering.
9. **Visualizations** — static and interactive charts; dashboards snapshots.
10. **Export & persistence** — save processed datasets to CSV/Parquet or write back to SQL Server.
11. **Conclusions & next steps** — summary insights and recommended actions.

---

## Example SQL queries used

**Pull recent sales data**

```sql
SELECT
    s.SaleID,
    s.SaleDate,
    s.ProductID,
    p.ProductName,
    s.Quantity,
    s.UnitPrice,
    (s.Quantity * s.UnitPrice) as TotalPrice,
    s.StoreID
FROM Sales s
JOIN Products p ON s.ProductID = p.ProductID
WHERE s.SaleDate >= DATEADD(month, -12, GETDATE());
```

**Aggregate monthly revenue**

```sql
SELECT
    YEAR(SaleDate) as yr,
    MONTH(SaleDate) as mn,
    SUM(Quantity * UnitPrice) as revenue
FROM Sales
GROUP BY YEAR(SaleDate), MONTH(SaleDate)
ORDER BY yr, mn;
```

Use parameterized queries from Python to avoid SQL injection.

---

## Common data cleaning steps performed

* Convert date columns to `datetime`.
* Handle missing `UnitPrice` by filling with median or dropping depending on context.
* Remove duplicate transaction rows.
* Standardize categorical values (strip, lower-case where necessary).
* Create derived columns (e.g., `TotalPrice = Quantity * UnitPrice`, `OrderMonth`, `OrderWeek`).

---

## Visualization examples

* Time-series of monthly revenue (line chart).
* Top products by revenue (bar chart).
* Sales by region / store (choropleth or grouped bar chart).
* Cohort retention heatmap (pivot table + heatmap).

In the notebook you will find `matplotlib` and `plotly` examples for interactive exploration.

## Dashboard
<img width="2489" height="1046" alt="Home Dashboard" src="https://github.com/Mohamed-Nofal-DataAnalysis/Data-Analysis-with-Python-SQL/blob/main/Total%20Revenue%20by%20Product%20Category.png" />
<img width="2461" height="1058" alt="Datiles Dashboard" src="https://github.com/Mohamed-Nofal-DataAnalysis/Data-Analysis-with-Python-SQL/blob/main/Total%20Monthly%20Revenue%20Over%20Time.png" />
---

## Exporting & saving results

Save cleaned data locally:

```python
df_processed.to_parquet('data/processed/sales_processed.parquet', index=False)
```

Write back to SQL Server (example):

```python
from sqlalchemy import create_engine
engine = create_engine(sqlalchemy_conn_string)
df_processed.to_sql('sales_processed', con=engine, if_exists='replace', index=False)
```

---

## Security & best practices

* Use environment variables or secret stores for credentials.
* Limit the SQL account permissions to least-privilege (read-only if possible).
* Avoid `SELECT *` in production queries; explicitly list columns.
* Parameterize queries to prevent SQL injection.
* Log and handle exceptions for the DB connection and queries.

---

## How to reproduce the analysis locally

1. Clone the repository.
2. Create and activate a virtual environment.
3. Install dependencies from `requirements.txt`.
4. Create a `.env` file with your SQL Server credentials.
5. Open `SQL Project.ipynb` with Jupyter or JupyterLab and run cells in order.

---

## Troubleshooting

* **ODBC driver not found:** Install Microsoft ODBC Driver 17 (or 18) for SQL Server for your OS.
* **Authentication issues:** Check username/password and whether SQL Server allows SQL authentication. For Windows auth, use the appropriate driver options.
* **Connection timeouts:** Ensure network reachability and that the server allows remote connections and the port (default 1433) is open.

---

## Next steps & suggestions

* Convert key charts into a Power BI or Dash app for stakeholders.
* Schedule the notebook to run regularly and store outputs.
* Add unit tests for data transformation steps (e.g., using `pytest` and small sample CSVs).
* Add a data dictionary describing each column and its meaning.

---

## Credits

Project prepared by the repository owner. Based on standard Python data analysis patterns and SQL Server integrations.

---


## Contact

If you need changes to the README or want the version in Arabic, or a LinkedIn post and `README` split into a single-file copy-friendly format, tell me what language you prefer and I will generate it.
