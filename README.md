# Financial_Data_Warehouse

# Overview
This project is a full-stack financial data engineering and analytics platform designed to collect, process, store, and analyze financial market data. It integrates multiple data sources to explore relationships between macroeconomic indicators, company fundamentals, and stock market performance.

The system combines:
  - A Python-based ETL pipeline
  - A PostgreSQL data warehouse
  - A Spring Boot backend API

# Project Goals
This platform is built to answer key financial questions such as:
  - How do interest rate changes impact stock performance?
  - Does inflation influence market volatility?
  - Do companies with strong earnings growth outperform others?
  - Which sectors perform best during economic downturns?
  - Can macroeconomic indicators help predict stock returns?

# System Architecture
Financial APIs ----> Python ETL Pipeline ---> PostgreSQL Data Warehouse ---> Spring Boot API ---> Machine Learning Service

# Data Pipeline
1. Extract
  - Pull stock market data from APIs
  - Retrieve macroeconomic indicators
  - Collect company financial data

2. Transform
  - Clean and normalize datasets
  - Standardize schema
  - Handle missing values

3. Load
  - Store processed data in PostgreSQL
