# Financial Data Warehouse and Market Analytics Platform

An end-to-end financial data engineering, analytics, and machine learning platform that integrates stock market data, macroeconomic indicators, and company financial statements into a multi-database architecture for analytical research and predictive modeling.

The platform was designed to address the problem of **financial data fragmentation**: market prices, economic indicators, and company fundamentals are typically distributed across separate data sources, making integrated analysis difficult and time-consuming.

The system automates the process of collecting, cleaning, transforming, storing, analyzing, and modeling these datasets while providing an interactive Streamlit dashboard for exploring analytical results and machine learning predictions.

---

## Overview

The platform combines:

* **Multi-source data ingestion** from Yahoo Finance, FRED, and Financial Modeling Prep
* **Python-based ETL pipeline** for extraction, transformation, validation, and loading
* **Multi-database architecture** using PostgreSQL, MySQL, and MongoDB
* **Dimensional data warehouse** using a PostgreSQL star schema
* **Feature engineering and ML feature store**
* **Machine learning pipeline** for forward stock predictions
* **Analytical functions** for investigating financial and macroeconomic relationships
* **Streamlit dashboard** for interactive visualization and analysis
* Modular components designed to support additional securities, indicators, and data sources

The completed system represents an end-to-end pipeline from **external financial APIs → raw data storage → data transformation → analytical warehouse → feature engineering → machine learning → interactive visualization**.

---

# Project Goals

The platform was developed around five research questions:

### 1. Interest Rates and Stock Performance

**How do changes in the federal funds rate affect the performance of large-cap technology stocks?**

The system combines daily stock price data with Federal Reserve interest-rate data to investigate historical relationships between monetary policy and equity performance.

### 2. Inflation and Market Volatility

**Does inflation influence stock market volatility?**

Macroeconomic indicators are integrated with market data to analyze whether changes in inflation are associated with changes in stock volatility.

### 3. Company Fundamentals and Performance

**Do companies with stronger financial fundamentals demonstrate better stock performance?**

Company-level financial statement data—including revenue, net income, and earnings per share—is integrated with market data for comparative analysis.

### 4. Sector Performance

**Which sectors demonstrate stronger performance under different economic conditions?**

Company metadata and market performance are combined to support sector-level analysis.

### 5. Predictive Modeling

**Can historical market, macroeconomic, and company-level data be used to generate forward-looking stock predictions?**

Engineered financial features are used by a machine learning pipeline to generate BUY, HOLD, or SELL predictions with associated class probabilities.

---

# System Architecture

```text
                  External Data Sources
        ┌─────────────────────────────────────┐
        │                                     │
        │  Yahoo Finance     FRED       FMP   │
        │  (Stock Prices)    (Macro)   (Fund.)│
        │                                     │
        └──────────────────┬──────────────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │ Python ETL Pipeline │
                │                     │
                │  Extract            │
                │  Transform          │
                │  Validate           │
                │  Load               │
                └──────────┬──────────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │ MongoDB  │  │  MySQL   │  │PostgreSQL│
        │          │  │          │  │          │
        │ Raw Data │  │Operational│ │ Warehouse│
        │ /Archive │  │   Data   │  │ Star     │
        └──────────┘  └──────────┘  │ Schema   │
                                    └─────┬────┘
                                          │
                         ┌────────────────┼────────────────┐
                         ▼                ▼                ▼
                  ┌────────────┐  ┌────────────┐  ┌────────────┐
                  │ Feature    │  │ Analytics  │  │     ML     │
                  │ Engineering│  │            │  │   Model    │
                  └────────────┘  └────────────┘  └─────┬──────┘
                                                        │
                                                        ▼
                                               ┌─────────────────┐
                                               │   Predictions   │
                                               │ fact_predictions│
                                               └────────┬────────┘
                                                        │
                                                        ▼
                                               ┌─────────────────┐
                                               │    Streamlit    │
                                               │    Dashboard    │
                                               └─────────────────┘
```

---

# Data Sources

The platform integrates three external financial data sources.

### Yahoo Finance

Historical stock market data is collected using the `yfinance` Python library.

Data includes:

* Open
* High
* Low
* Close
* Volume
* Historical daily prices

The ingestion process supports configurable tickers and date ranges.

### FRED

Macroeconomic data is collected through the Federal Reserve Bank of St. Louis FRED API using `fredapi`.

The current implementation incorporates:

* Federal Funds Rate
* Consumer Price Index
* GDP Growth Rate
* Unemployment Rate

The indicators are normalized into a common long-format structure so additional economic series can be incorporated without requiring major schema changes.

### Financial Modeling Prep

Financial statement information is collected through the Financial Modeling Prep API.

The current analysis incorporates:

* Revenue
* Net Income
* Earnings Per Share (EPS)

Annual financial statement data is integrated with market and macroeconomic datasets for downstream analysis.

---

# ETL Pipeline

The ETL pipeline is the core of the platform and is organized into separate ingestion, processing, and loading components.

## 1. Extract

Python ingestion modules retrieve data from the three external sources.

```text
Yahoo Finance ──┐
                │
FRED API ───────┼──> Raw Data
                │
FMP API ────────┘
```

The ingestion layer handles:

* API communication
* Configurable ticker/date parameters
* Response validation
* Rate-limit considerations
* Source-specific data retrieval

Raw API responses are preserved in MongoDB to support auditing and potential reprocessing.

---

## 2. Transform

Each data source has its own processing and cleaning logic because the raw formats and data-quality issues differ between sources.

Transformations include:

* Standardizing column names
* Normalizing date formats
* Handling missing values
* Removing invalid records
* Adding ticker identifiers
* Normalizing economic indicator names
* Calculating derived financial metrics
* Structuring datasets for relational storage

For example, stock data is converted from the `yfinance` DataFrame format into a relational structure compatible with the warehouse schema.

---

## 3. Load

Processed data is distributed across three databases according to its purpose.

| Database       | Role                 | Purpose                                                                   |
| -------------- | -------------------- | ------------------------------------------------------------------------- |
| **MongoDB**    | Raw document store   | Preserve raw API responses for auditing and reprocessing                  |
| **MySQL**      | Operational store    | Maintain reference and transactional data with relational constraints     |
| **PostgreSQL** | Analytical warehouse | Support complex analytical queries, joins, aggregations, and ML workloads |

This separation of responsibilities allows raw, operational, and analytical workloads to remain logically independent.

---

# Data Warehouse Design

The PostgreSQL database uses a **dimensional star schema** designed around financial time-series analysis.

The warehouse contains:

* Fact tables for market and economic measurements
* Company and other dimension tables
* Feature-store tables for machine learning
* Prediction tables for storing model outputs

The primary market-price fact table connects to dimension tables through foreign-key relationships, while feature-store and ML tables provide a satellite layer supporting predictive analytics.

### Simplified Warehouse Structure

                 ┌─────────────────┐
                 │   dim_company   │
                 └────────┬────────┘
                          │
                          │
                ┌─────────▼──────────┐
                │  fact_stock_prices │
                └─────────┬──────────┘
                          │
                ┌─────────▼──────────┐
                │ Feature Store / ML │
                └─────────┬──────────┘
                          │
                ┌─────────▼──────────┐
                │ fact_predictions  │
                └────────────────────┘

The warehouse design allows market data, company information, macroeconomic indicators, engineered features, and model predictions to be queried within a common analytical framework.

---

# Machine Learning Pipeline

The platform extends beyond traditional data warehousing by incorporating machine learning into the analytical workflow.

The ML pipeline:

1. Retrieves historical data from the warehouse
2. Generates engineered financial features
3. Builds the model feature set
4. Trains an ensemble classification model
5. Generates forward-looking predictions
6. Produces class probabilities
7. Writes predictions back into the PostgreSQL warehouse

The current model uses a fixed set of engineered features to generate **30-day forward BUY/HOLD/SELL classifications**.

Predictions are stored in the `fact_predictions` table alongside confidence and class-probability values, allowing model outputs to be analyzed alongside the underlying financial data.

---

# Analytics

The analytical layer uses the integrated warehouse to investigate relationships between:

* Interest rates
* Inflation
* GDP
* Unemployment
* Stock returns
* Stock volatility
* Company fundamentals
* Sector performance

Because these datasets originate from different sources and have different time frequencies, the ETL process standardizes their structure before they are integrated into the warehouse.

The resulting platform allows financial and economic questions to be investigated without repeatedly performing manual data collection and preprocessing.

---

# Streamlit Dashboard

A Streamlit application provides an interactive interface for exploring the warehouse and model outputs.

The dashboard supports:

* Financial data exploration
* Market analytics
* Macroeconomic analysis
* Company/sector analysis
* Machine learning predictions
* Visualization of historical trends

The dashboard communicates directly with the underlying databases and provides the primary user-facing analytical interface for the completed implementation.

---

# Engineering Challenges

Developing the platform required solving several implementation problems across the data pipeline, database layer, machine learning workflow, and dashboard.

### Multi-Database Connectivity

The application maintains connections to PostgreSQL, MySQL, and MongoDB simultaneously.

This required managing different database clients and connection lifecycles while ensuring that cached database connections remained valid across Streamlit page renders.

### Data Quality and Schema Alignment

Differences between external data sources required source-specific transformation logic.

One issue occurred when sector and industry information was missing from the PostgreSQL company dimension. The solution involved enriching the PostgreSQL dimension using authoritative reference data stored in MySQL.

### API and Rate-Limit Constraints

Different APIs imposed different access patterns and limitations.

The ingestion layer was designed around the behavior of each individual source, including rate-limit handling for Financial Modeling Prep.

### Visualization Scaling

Macroeconomic indicators with substantially different scales initially caused visualization problems.

For example, CPI and interest-rate values could not be meaningfully displayed on a single y-axis, requiring a dual-axis Plotly visualization.

### Scope and Architecture Decisions

The original project proposal included a Spring Boot REST API.

During development, the API layer was ultimately removed from the implemented architecture because maintaining a separate Java service would introduce additional infrastructure complexity without providing sufficient benefit to the analytical objectives of the project.

The Streamlit application instead communicates directly with the database and provides the user-facing analytical layer.

---

# Technology Stack

### Programming

* Python
* SQL

### Data Engineering

* Pandas
* NumPy
* SQLAlchemy
* ETL pipelines
* API integration
* Data cleaning and transformation

### Databases

* PostgreSQL
* MySQL
* MongoDB

### Machine Learning

* Scikit-learn
* Feature engineering
* Ensemble classification
* Predictive modeling
* Model evaluation

### APIs & Data Sources

* Yahoo Finance / `yfinance`
* FRED / `fredapi`
* Financial Modeling Prep

### Visualization & Application

* Streamlit
* Plotly
* Matplotlib

---

# Project Structure

```text
financial-data-warehouse/
│
├── data ingestion/
│   ├── stock data.py
│   ├── fred data.py
│   └── fmp data.py
│
├── data processing/
│   ├── clean stock data.py
│   ├── clean fred data.py
│   └── clean financials.py
│
├── data loading/
│   └── ...
│
├── ml/
│   ├── feature engineering
│   ├── model training
│   ├── scoring
│   └── analytics
│
├── dashboard/
│   └── Streamlit application
│
├── main.py
└── README.md
```

---

# Scalability and Future Development

The architecture was intentionally designed so that additional data can be incorporated without fundamentally redesigning the system.

Potential extensions include:

* Expanding the number of tracked companies and sectors
* Adding additional macroeconomic indicators
* Automated model retraining
* Sentiment analysis from financial news
* LSTM or TCN-based time-series models
* Real-time market data ingestion
* Event-driven processing
* REST API integration
* Containerized deployment
* Cloud infrastructure
* Automated data-quality monitoring
* Expanded dashboard functionality

The current batch architecture could also be extended toward a real-time system using streaming market data and WebSocket-based ingestion.

---

# Key Takeaways

This project demonstrates experience across the complete data and ML lifecycle:

```text
Data Sources
     ↓
API Integration
     ↓
Data Ingestion
     ↓
Data Cleaning
     ↓
ETL
     ↓
Multi-Database Architecture
     ↓
Dimensional Data Warehouse
     ↓
Feature Engineering
     ↓
Machine Learning
     ↓
Prediction Storage
     ↓
Analytics
     ↓
Interactive Dashboard
```

Rather than treating data collection, warehousing, analytics, and machine learning as separate tasks, the project integrates them into a single end-to-end platform for financial analysis and predictive modeling.
