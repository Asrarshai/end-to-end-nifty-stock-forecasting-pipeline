# End-to-End Nifty Stock Forecasting Pipeline

An end-to-end data engineering and machine learning pipeline for forecasting Nifty stock prices. The project is implemented using Apache Airflow, Azure Data Factory, Azure Databricks, Azure Data Lake Storage, PySpark, Prophet and LSTM.

## Project Overview

This project automates the complete stock forecasting workflow, starting from historical market-data ingestion and ending with next-trading-day forecast generation.

The current implementation uses HDFC Bank stock data (`HDFCBANK.NS`) as the working dataset. The pipeline is designed in a reusable way so that it can be extended to other Nifty-listed stocks.

## Architecture

The project follows a layered data-lake architecture:

```text
        
Apache Airflow
        |
        v
Azure Data Factory
        |
        v
Azure Databricks
        |
        v
Bronze Layer - Raw Data
        |
        v
Silver Layer - Cleaned Data
        |
        v
Gold Layer - Features and Forecasts
        |
        +-------------------+
        |                   |
        v                   v
     Prophet             LSTM
        |                   |
        +---------+---------+
                  |
                  v
       Forecasts and Model Metrics
```

## Workflow

1. Apache Airflow schedules and triggers the pipeline.
2. Airflow invokes the Azure Data Factory pipeline.
3. Azure Data Factory executes the Azure Databricks notebooks in sequence.
4. Historical stock data is stored in the bronze layer.
5. The bronze-to-silver process cleans and validates the data.
6. The silver-to-gold process creates lag and moving-average features.
7. Prophet generates a trend-based forecast.
8. LSTM generates a sequence-based forecast.
9. Forecasts and model metrics are stored in the gold layer.
10. The next forecast date is calculated using weekday logic so that weekends are skipped.

## Data Layers

### Bronze Layer

The bronze layer stores raw historical stock data without major transformations. It preserves the original source data for auditing and future reprocessing.

### Silver Layer

The silver layer contains cleaned and standardised data. The pipeline:

- Converts dates to timestamp format.
- Converts prices to numeric values.
- Converts volume to the correct numeric type.
- Removes records with missing Date or Close values.
- Removes duplicate Date and ticker combinations.
- Sorts records chronologically.

### Gold Layer

The gold layer contains model-ready features, forecasts and pipeline metrics.

Feature columns include:

- `lag_1`
- `lag_2`
- `lag_3`
- `ma_5`
- `ma_10`

## Machine Learning Models

### Prophet

Prophet is used as a trend and seasonality-based forecasting model. It uses historical Date and Close values and captures the overall movement and weekly pattern of the stock.

### LSTM

The LSTM model learns sequential patterns from the last ten closing-price observations. Close values are scaled before training and converted back to the original price scale after prediction.

The trained LSTM model is stored in Azure Data Lake Storage and reused during subsequent pipeline runs.

## Technologies Used

- Apache Airflow
- Azure Data Factory
- Azure Databricks
- Azure Data Lake Storage Gen2
- PySpark
- Python
- Pandas
- NumPy
- Prophet
- TensorFlow/Keras
- scikit-learn
- Parquet
- CSV
- JSON watermark metadata

## Repository Structure

```text
.
├── HDFC_Data/
│   ├── bronze/
│   ├── silver/
│   └── gold/
├── hdfc_forecast/
├── 01_ingestion_to_bronze.py
├── 02_bronze_to_silver.py
├── 03_silver_to_gold_train_forecast.py
├── trigger_hdfc_adf_pipeline.py
└── README.md
```

Update the filenames in this structure if your repository uses different notebook names.

## Forecast Output

The gold forecast output contains:

- Forecast date
- Ticker
- Prophet forecast
- LSTM forecast
- Latest-close fallback
- Model used
- Run timestamp

The pipeline stores forecasts as Parquet and also creates readable CSV output for analysis and reporting.

## Incremental Processing

A watermark JSON file stores the latest processed date. This allows future executions to identify the latest available data and update the pipeline incrementally.

The LSTM model is persisted in Azure Data Lake Storage so it can be loaded and reused on subsequent runs.

## Weekend Handling

The pipeline calculates the next trading day rather than simply adding one calendar day.

- If the latest data is from Wednesday, the forecast date is Thursday.
- If the latest data is from Friday, Saturday and Sunday are skipped.
- The forecast date is then set to Monday.

Exchange holidays can be added in a future version using an exchange trading calendar.

## Model Evaluation

A complete model comparison should use chronological backtesting. Prophet and LSTM predictions should be compared with actual closing prices using:

- MAE
- MAPE
- RMSE

The model with the lower error on the selected test period can then be preferred for production forecasting.

## Disclaimer

This project is developed for academic and learning purposes. Forecasts are analytical estimates and should not be considered financial or investment advice.
