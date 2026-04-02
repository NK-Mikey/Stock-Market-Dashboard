# MarketPulse - Project Documentation
**Project:** MarketPulse - Automated S&P 500 Market Intelligence Dashboard
<br>
**Author:** Naveen Karan Krishna
<br>
**Version:** 1.0
<br>
**Last Updated:** April 2026
<br>
**Tools Used:** Python, Power BI Desktop, GitHub Actions

---

## Table of Contents

- [1. Project Overview](#1-project-overview)
- [2. Data Sources](#2-data-sources)
- [3. Data Dictionary](#3-data-dictionary)
- [4. Report Logic](#4-report-logic)
- [5. Data Refresh Process](#5-data-refresh-process)
- [6. Process Documentation](#6-process-documentation)
- [7. Process Mapping](#7-process-mapping)
- [8. Reporting Standards](#8-reporting-standards)
- [9. Runbook and SOP](#9-runbook-and-sop)
- [10. Troubleshooting Guide](#10-troubleshooting-guide)

---

## 1. Project Overview

MarketPulse is a Yahoo Finance-style stock market intelligence dashboard that tracks the Top 25 S&P 500 companies by market capitalization. It combines a fully automated Python data pipeline with a Power BI dashboard to deliver daily updated market insights including price movements, volume analysis, risk vs return comparisons and 7-day price change breakdowns.

The dashboard is designed for analysts and portfolio reviewers who need a quick daily snapshot of large-cap market performance without relying on paid data subscriptions.

**Key Objectives:**
- Automate daily stock data collection with zero manual intervention
- Present market data in a clean, interactive financial dashboard
- Enable dynamic filtering by ticker, sector, date range and performance category
- Provide risk and return context across all 25 tracked companies

---

## 2. Data Sources

| Source | Type | Data Provided | Access Method | Refresh Frequency |
|---|---|---|---|---|
| Yahoo Finance (via yfinance) | Free public API | Daily OHLCV prices, market cap, PE ratio, avg volume | Python yfinance library | Daily via GitHub Actions |
| Wikipedia | Public web page | S&P 500 company list, sector, industry classification | Python requests + pandas read_html | On script execution |

**Data Disclaimer:**
All data is sourced from publicly available free APIs for educational and portfolio purposes only. This dashboard does not constitute financial advice.

**Known Limitations of Sources:**
- Yahoo Finance occasionally returns incomplete or missing data for certain tickers without warning
- Wikipedia's S&P 500 list may lag behind official index changes by a few days
- yfinance provides end-of-day closing prices only, no intraday or real-time data

---

## 3. Data Dictionary

### 3.1 Table: stock_prices (Fact Table)

| Column | Data Type | Description | Example | Source |
|---|---|---|---|---|
| Date | Date | Trading date for this row | 2024-03-15 | yfinance download |
| Ticker | Text | Stock ticker symbol | AAPL | yfinance download |
| Open | Decimal | Opening price for the trading day | 172.50 | yfinance download |
| High | Decimal | Highest price reached during the trading day | 175.20 | yfinance download |
| Low | Decimal | Lowest price reached during the trading day | 171.80 | yfinance download |
| Close | Decimal | Closing price at end of trading day | 174.30 | yfinance download |
| Volume | Integer | Number of shares traded during the day | 58432100 | yfinance download |
| Previous Close (Col) | Decimal | Closing price of the most recent previous trading day and handles weekends and holidays | 173.10 | Calculated column in Power BI |
| Daily Change (Col) | Decimal | Difference between Close and Previous Close | 1.20 | Calculated column in Power BI |
| Daily Return (Col) | Decimal | Daily Change expressed as a percentage of Previous Close | 0.0069 | Calculated column in Power BI |

---

### 3.2 Table: dim_stock (Dimension Table)

| Column | Data Type | Description | Example | Source |
|---|---|---|---|---|
| StockID | Integer | Unique numeric identifier for each company | 1 | Auto-generated in Python |
| Ticker | Text | Stock ticker symbol - primary key for relationships | AAPL | Wikipedia / yfinance |
| CompanyName | Text | Full legal company name | Apple Inc. | Wikipedia |
| Sector | Text | GICS sector classification | Technology | Wikipedia |
| Industry | Text | GICS sub-industry classification | Consumer Electronics | Wikipedia |

---

### 3.3 Table: fundamentals (Dimension Table)

| Column | Data Type | Description | Example | Source |
|---|---|---|---|---|
| Ticker | Text | Stock ticker symbol - foreign key to dim_stock | AAPL | yfinance |
| Company Name | Text | Full company name as returned by Yahoo Finance | Apple Inc. | yfinance .info |
| Market Cap | Integer | Total market capitalization in USD | 3673599508480 | yfinance .info |
| PE Ratio (TTM) | Decimal | Trailing twelve months price-to-earnings ratio | 31.60 | yfinance .info |
| Avg Volume (3M) | Integer | Average daily trading volume over the past 3 months | 47466349 | yfinance .info |

---

### 3.4 Table: Dim_Date (Date Dimension - created in Power BI using DAX)

| Column | Data Type | Description | Example |
|---|---|---|---|
| Date | Date | Calendar date | 2024-03-15 |
| Year | Integer | Calendar year | 2024 |
| Month | Text | Abbreviated month name | Mar |
| MonthNum | Integer | Numeric month for sorting | 3 |
| YearMonth | Text | Combined year and month for grouping | 2024-03 |
| DayOfWeek | Text | Day name | Friday |

---

### 3.5 Disconnected Table: Date Range (UI Control)

| Column | Data Type | Description |
|---|---|---|
| Range | Text | Date range label used in the slicer (3D, 7D, 1M, 6M, YTD, 1Y, All) |
| SortOrder | Integer | Numeric sort order to display ranges chronologically in the slicer |

**Purpose:** Powers the dynamic date range slicer. Not connected to any other table. Selected value is read by DAX measures via SELECTEDVALUE().

---

### 3.6 Disconnected Table: Sort Selector (UI Control)

| Column | Data Type | Description |
|---|---|---|
| Sort Type | Text | Sorting category label (Top Gainers, Top Losers, Most Active) |

**Purpose:** Powers the Top Gainers / Top Losers / Most Active tile slicer. Not connected to any other table. Selected value is read by the Dynamic Sort measure via SELECTEDVALUE().

---

### 3.7 Key DAX Measures

| Measure Name | Description | Formula Logic |
|---|---|---|
| Previous Price | Closing price of the most recent prior trading day per ticker | Finds the latest date before current date using ALL() and explicit date filtering. Handles weekends and holidays. |
| Change | Absolute difference between today's Close and Previous Price | Close minus Previous Price |
| Change % | Percentage change from Previous Price to today's Close | Change divided by Previous Price |
| 30D MA | 30-day moving average of closing prices | AVERAGEX over DATESINPERIOD of 30 days |
| 90D MA | 90-day moving average of closing prices | AVERAGEX over DATESINPERIOD of 90 days |
| Running Volume High | Cumulative maximum volume up to the current date per ticker | MAX volume over all dates up to and including current date using ALL() filter |
| Volatility | Standard deviation of daily returns per ticker | STDEV.P applied over all dates using ALL() |
| Total Return % | Percentage return from the first available date to latest date | (Latest Close minus First Close) divided by First Close |
| Avg Volatility | Average volatility across all tickers | AVERAGEX over all ticker values |
| Dynamic Sort " " | Controls table sort order based on selected slicer option | SWITCH on SELECTEDVALUE of Sort Selector returning Change %, negative Change %, or Volume |
| Date Filter | Binary flag indicating whether a date falls within the selected range | Returns 1 if date is on or after Range Start Date, else 0 |
| Range Start Date | Calculated start date based on selected date range slicer | SWITCH on SELECTEDVALUE of Date Range returning appropriate date offset |

---

## 4. Report Logic

### 4.1 Stock Screener Table

The main table displays one row per ticker showing the most recent trading day's data. All metrics are evaluated in the context of the latest available date filtered by the ticker slicer.

**Daily Change Calculation:**
Previous Price uses ALL() to remove Power BI's visual filter context, then explicitly re-applies date and ticker filters to find the most recent prior trading day. This approach correctly handles weekends and market holidays where a simple date minus one calculation would return blank.

**52-Week Metrics:**
52W High and 52W Low use DATESINPERIOD with a 365-day lookback window. 52W Change % compares the current close to the close from 365 days prior.

**Sorting Logic:**
The table is sorted by the Dynamic Sort measure (named " " to hide it visually). The Sort Selector disconnected table drives which metric controls the sort order - Change % ascending for Top Losers, descending for Top Gainers, and Volume descending for Most Active.

### 4.2 Price and Moving Average Chart

The line chart shows Close price, 30D MA and 90D MA for the selected ticker. The date range slicer filters this chart via the Date Filter measure applied as a visual-level filter set to 1.

### 4.3 Volume Chart

A combo chart showing daily Volume as columns and Running Volume High as a line. The Running Volume High persists at its peak value until a new higher volume is recorded, flagging days of unusual activity where volume exceeds the previous high.

### 4.4 Risk vs Return Scatter Plot

Each dot represents one ticker. Volatility (X-axis) is the standard deviation of daily returns calculated over all available dates. Total Return % (Y-axis) measures performance from the first to the latest available date. Bubble size represents Avg Volume (3M) from the fundamentals table. Dot color represents sector from dim_stock.

### 4.5 7-Day Price Change Waterfall Chart

Shows the daily Close price change for each of the last 7 trading days for the selected ticker. Positive days display in green and negative days in red, making it easy to identify which specific days drove the weekly movement.

---

## 5. Data Refresh Process

### 5.1 Automated Pipeline Overview

The data pipeline runs automatically every day at 6:00 AM UTC via GitHub Actions. No manual steps are required for data collection.

| Step | Action | Tool | Output |
|---|---|---|---|
| 1 | GitHub Actions triggers on schedule | GitHub Actions cron | Workflow starts |
| 2 | Repository is cloned to virtual machine | actions/checkout | Files available |
| 3 | Python environment is set up | actions/setup-python | Python 3.11 ready |
| 4 | Required libraries are installed | pip install | yfinance, pandas, requests, lxml |
| 5 | Data extraction script runs | Python | Updated CSVs |
| 6 | Changes are committed and pushed | git commit and push | data/ folder updated on GitHub |

### 5.2 Power BI Refresh Steps

After the automated pipeline runs, refresh the dashboard manually:

1. Open MarketPulse.pbix in Power BI Desktop
2. Press Alt + F5 or click Home then Refresh
3. Power BI fetches the latest CSVs from the GitHub raw URLs
4. All visuals update automatically

**Expected refresh time:** Under 30 seconds for 25 tickers with 2 years of daily data.

### 5.3 Data Connection

Power BI connects to the data using GitHub raw file URLs rather than local file paths. This means any machine with Power BI Desktop and internet access can open the file and refresh without needing local copies of the CSVs.

| File | GitHub Raw URL Pattern |
|---|---|
| stock_prices.csv | https://raw.githubusercontent.com/NK-Mikey/Stock-Market-Dashboard/refs/heads/main/data/stock_prices.csv |
| fundamentals.csv | https://raw.githubusercontent.com/NK-Mikey/Stock-Market-Dashboard/refs/heads/main/data/fundamentals.csv |
| dim_stock.csv | https://raw.githubusercontent.com/NK-Mikey/Stock-Market-Dashboard/refs/heads/main/data/dim_stock.csv |

---

## 6. Process Documentation

### 6.1 Adding a New Ticker

To add a new ticker to the dashboard:

1. Open fetch_top25_sp500.py
2. If using a hardcoded list, add the ticker symbol to the TICKERS list
3. If using the Wikipedia-based top 25 selection, the ticker will be included automatically if it enters the top 25 by market cap
4. Run the script locally to verify data is returned correctly
5. Commit the updated script to GitHub
6. The next scheduled run will include the new ticker automatically

### 6.2 Changing the Date Range

To change the historical data start date:

1. Open fetch_top25_sp500.py
2. Locate the following line: LOOKBACK_DAYS = 365 * 3 for a rolling 3-year window, or replace it with START_DATE = "2023-01-01" to define a fixed start date instead.
3. Change to the desired start date in YYYY-MM-DD format
4. Run the script and it will overwrite existing CSVs with the new date range
5. Refresh Power BI to load the updated data

### 6.3 Updating the GitHub Actions Schedule

To change the daily run time:

1. Open .github/workflows/daily_refresh.yml
2. Find the cron expression: cron: '0 6 * * *'
3. Update using cron syntax. For example 0 9 * * * runs at 9:00 AM UTC
4. Commit the change and the new schedule takes effect immediately

---

## 7. Process Mapping

### 7.1 End-to-End Data Flow

```
Wikipedia (S&P 500 list)
          |
          v
Python Script (fetch_top25_sp500.py)
          |
          |-- Filters Top 25 by Market Cap
          |-- Downloads daily OHLCV prices via yfinance
          |-- Fetches fundamentals via yfinance
          |
          v
Three CSV files saved to data/ folder
          |
          v
GitHub Actions commits and pushes updated files
          |
          v
GitHub Repository (data/ folder updated)
          |
          v
Power BI Desktop reads raw GitHub URLs
          |
          v
Power BI Data Model
          |
          |-- stock_prices (fact)
          |-- dim_stock (dimension)
          |-- fundamentals (dimension)
          |-- Dim_Date (date dimension)
          |-- Date Range (disconnected - UI)
          |-- Sort Selector (disconnected - UI)
          |
          v
DAX Measures calculate metrics on refresh
          |
          v
Dashboard visuals update and display latest data
```

### 7.2 User Interaction Flow

```
User opens MarketPulse.pbix
          |
          v
Presses Alt + F5 to refresh
          |
          v
Selects a ticker from the slicer
          |
          |-- Price and MA chart updates
          |-- Volume chart updates
          |-- Waterfall chart updates
          |
          v
Selects a date range (3D / 7D / 1M / 6M / YTD / 1Y / All)
          |
          |-- Price chart adjusts date window
          |-- Volume chart adjusts date window
          |
          v
Selects Top Gainers / Top Losers / Most Active
          |
          |-- Screener table re-sorts automatically
          |
          v
Hovers over scatter plot dot
          |
          v
Tooltip shows ticker, return, volatility and volume
```

---

## 8. Reporting Standards

### 8.1 Color Standards

| Signal | Color | Hex Code | Usage |
|---|---|---|---|
| Positive / Gain | Green | #00B050 | Change %, Daily Change |
| Negative / Loss | Red | #FF4444 | Change %, Daily Change |
| Positive | Light Green | #68CBA3 | Waterfall positive bars |
| Negative | Light Red | #E86767 | Waterfall negative bars |
| Primary data line | White | #FFFFFF | Current price line in chart |
| 30-Day MA line | Blue | #4E79A7 | 30D moving average |
| 90-Day MA line | Orange | #F28E2B | 90D moving average |
| Background | Deep navy | #0D1117 | Canvas background |
| Visual background | Dark slate | #161B22 | Individual visual backgrounds |
| Axis labels | White | #FFFFFF | All axis text |

### 8.2 Number Formatting Standards

| Metric | Format | Example |
|---|---|---|
| Price | 2 decimal places | 174.30 |
| Change | 2 decimal places with sign | +1.20 |
| Change % | Percentage 3 decimal places | +0.695% |
| Volume | Abbreviated millions | 58.4M |
| Market Cap | Abbreviated trillions | 3.67T |
| PE Ratio | 2 decimal places | 31.60 |
| Moving Averages | 2 decimal places | 172.45 |
| Volatility | Percentage 2 decimal places | 1.82% |
| Total Return % | Percentage 2 decimal places | +24.50% |

### 8.3 Naming Conventions

| Element | Convention | Example |
|---|---|---|
| DAX Measures | Title case with spaces | Previous Price, Running Volume High |
| Calculated Columns | Title case with (Col) suffix | Daily Change (Col), Daily Return (Col) |
| Disconnected Tables | Title case descriptive name | Date Range, Sort Selector |
| CSV files | Lowercase with underscores | stock_prices.csv, dim_stock.csv |
| Python variables | Lowercase with underscores | stock_prices_df, market_cap_df |
| Python constants | Uppercase with underscores | TICKERS, START_DATE, END_DATE |

### 8.4 Visual Standards

- All visuals use the dark theme style which mirrors the yahoo finance
- Chart titles are displayed for selective charts and written in sentence case
- Gridlines are turned off on all charts and reference lines are used instead where needed
- All slicers use tile orientation for button-style appearance
- Data labels are kept off on line charts to avoid clutter and tooltips provide detail on hover
- Conditional formatting is applied consistently where green for positive values, and red for negative values across all visuals

---

## 9. Runbook and SOP

### 9.1 Daily Operation Checklist

| Time | Action | Expected Result |
|---|---|---|
| 6:00 AM UTC | GitHub Actions triggers automatically | Pipeline starts running |
| 6:05 AM UTC | Python script completes | CSVs updated in data/ folder |
| 6:06 AM UTC | Git commit pushed to repository | GitHub shows latest commit |
| Morning | Open MarketPulse.pbix | Dashboard opens |
| Morning | Press Alt + F5 | Dashboard refreshes with latest data |
| Morning | Verify Last Trading Date card | Shows today's or most recent trading date |

### 9.2 Handing Over This Project to Another Analyst

If transferring this project to a colleague, share the following:

1. GitHub repository link - all code, CSVs and documentation live here
2. MarketPulse.pbix file - open in Power BI Desktop (free download)
3. This documentation file - covers all tables, measures and processes
4. Confirm they have Python installed with yfinance, pandas, requests and lxml if they want to run the script locally

The new analyst should be able to:
- Open the .pbix file and refresh data in under 5 minutes
- Understand every table and measure using this data dictionary
- Run the Python script locally if needed
- Modify the ticker list or date range following Section 6

---

## 10. Troubleshooting Guide

| Problem | Likely Cause | Solution |
|---|---|---|
| GitHub Actions shows red X | Permissions or dependency issue | Check the Actions log, look for the failing step |
| "nothing to commit" message | No data changed since last run | Not an error - pipeline ran successfully, data was unchanged |
| Power BI shows blank visuals after refresh | Connection to GitHub URL failed | Check internet connection, verify raw GitHub URLs are still valid |
| Previous Price shows blank | No prior trading day found | Check if data starts from a date where a prior day exists in the dataset |
| Scatter plot dots all in same position | Volatility or Total Return measure returning same value | Verify Daily Return (Col) column has correct values per ticker |
| GitHub Actions failing with 403 | Write permissions not set | Ensure permissions: contents: write is in the workflow file |
| yfinance returns empty data for a ticker | Yahoo Finance data gap | Check if ticker symbol is correct, re-run script after a few hours |
| Wikipedia fetch fails with HTTP 403 | User-Agent header missing or blocked | Verify headers dictionary is present in the requests.get call |

---

*This document covers all aspects of the MarketPulse project from data sourcing through to daily operation and handover. It is intended to serve as both internal technical documentation and a knowledge transfer resource.*
