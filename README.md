# E-commerce Order ETL Pipeline

A Python-based ETL (Extract, Transform, Load) pipeline that processes e-commerce order data, validates data quality, transforms valid records, exports results to CSV files, and loads processed and rejected records into PostgreSQL.

## Project Overview

This project demonstrates a basic data engineering workflow using Python, Pandas, SQLAlchemy, and PostgreSQL.

The pipeline extracts order records from a CSV file, standardizes the data, validates records against defined business rules, calculates tax and total amounts, and loads the results into CSV files and PostgreSQL tables.

## Architecture

```text
orders.csv
    |
    v
Extract
    |
    v
Clean and Standardize
    |
    v
Validate Data
    |
    +--------------------+
    |                    |
    v                    v
Valid Records       Rejected Records
    |                    |
    v                    |
Transform                 |
    |                    |
    +----------+---------+
               |
               v
       Load to CSV Files
               |
               v
       Load to PostgreSQL
               |
               v
      Verify Record Counts
```

## Technologies Used

- Python
- Pandas
- PostgreSQL
- SQLAlchemy
- psycopg2
- python-dotenv
- Python logging

## Project Structure

```text
python-ecommerce-etl/
├── .env                  # Local database credentials (not committed)
├── .gitignore
├── requirements.txt
├── README.md
├── data/
│   └── orders.csv
├── output/
│   ├── processed_orders.csv
│   └── rejected_orders.csv
├── logs/
└── src/
    └── pipeline.ipynb    # ETL implementation and execution
```

## Pipeline Steps

### 1. Extract

- Reads the source order data from `data/orders.csv`.
- Logs the extraction process and record count.
- Handles missing input files.

### 2. Clean and Standardize

- Standardizes column names by removing surrounding whitespace and converting them to lowercase.
- Removes surrounding whitespace from customer names.
- Standardizes customer names and order statuses.
- Converts amounts to numeric values, coercing invalid values to null.
- Converts order dates into datetime values, coercing invalid dates to null.

### 3. Validate

Records are accepted only when they satisfy the following rules:

- Order ID must not be null or empty.
- Order IDs must be unique across the dataset.
- Customer name must not be null or blank.
- Amount must be numeric and greater than zero.
- Status must be `completed`, `pending`, or `cancelled`.
- Order date must be valid.

Invalid records are separated into a rejected dataset. Both occurrences of a duplicated order ID are rejected.

### 4. Transform

For valid records, the pipeline calculates a 5% tax and the total order amount.

- **Tax:** `amount × 0.05`
- **Total amount:** `amount + tax`

### 5. Load

The pipeline exports:

- Valid and transformed records to `output/processed_orders.csv`.
- Rejected records to `output/rejected_orders.csv`.

It also loads the datasets into PostgreSQL tables:

- `processed_orders`
- `rejected_orders`

The current implementation uses `if_exists="replace"`, so the tables are replaced on each successful load.

### 6. Logging and Reconciliation

The pipeline logs extraction, cleaning, validation, transformation, loading, and failure events.

It also verifies that the number of valid records plus rejected records equals the source record count.

## Sample Data Quality Results

The supplied sample dataset contains 15 source records.

| Metric | Records |
|---|---:|
| Source records | 15 |
| Valid and processed | 8 |
| Rejected | 7 |

The reconciliation check is:

`15 = 8 + 7`

These results correspond to the supplied sample data and validation rules.

## Prerequisites

Install Python and PostgreSQL. Ensure the PostgreSQL server is running and that you have credentials for a database you can access.

## Setup and Installation

### 1. Clone the repository

```bash
git clone https://github.com/nareshkumars-mit/python-ecommerce-etl-pipeline.git
cd python-ecommerce-etl
```

### 2. Create and activate a virtual environment

```bash
python -m venv .venv
```

On Windows:

```bash
.venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure database credentials

Create a local `.env` file in the project root:

```dotenv
DB_HOST=localhost
DB_PORT=5432
DB_NAME=postgres
DB_USER=your_postgres_username
DB_PASSWORD=your_postgres_password
```

Replace the example values with your local PostgreSQL configuration.

**Security:** Never commit `.env` or actual database credentials to GitHub. The project uses `python-dotenv` to load local environment variables.

### 5. Run the notebook

Open `src/pipeline.ipynb` in VS Code or Jupyter Notebook, select your Python environment, and run the cells in order.

Ensure that the notebook's project-root detection matches your working directory.

### 6. Verify the PostgreSQL results

Run the following SQL in your PostgreSQL client:

```sql
SELECT COUNT(*) FROM processed_orders;

SELECT COUNT(*) FROM rejected_orders;
```

For the supplied sample data, the expected counts are 8 and 7, respectively.

## Error Handling and Data Quality

The project includes:

- Missing-file handling.
- Null and invalid-value checks.
- Duplicate order ID detection.
- Logging of pipeline stages and failures.
- Separate outputs for valid and rejected records.
- Record-count reconciliation.
- Database connection checks and database-load error logging.

## Future Improvements

- Move reusable ETL functions from the notebook into a standalone Python module.
- Add automated unit tests with `pytest`.
- Implement structured rejection reasons for each invalid record.
- Use database transactions and safer incremental loading.
- Add command-line configuration and environment-specific settings.
- Introduce orchestration and scheduling for automated pipeline runs.

## Learning Outcomes

This project demonstrates practical experience with Python-based ETL development, Pandas data cleaning, data quality validation, exception handling, logging, environment-variable management, SQLAlchemy, PostgreSQL integration, and record reconciliation.
