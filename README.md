# 📈 MarketPulse: An Automated S&P 500 Market Intelligence Dashboard

> **A Yahoo Finance–style stock market dashboard built with Python and Power BI,
> featuring a fully automated daily data pipeline powered by GitHub Actions.**

---

## 🎬 Demo

https://github.com/user/stock-market-dashboard/blob/main/demo/marketpulse_demo.mp4

> ▶️ *Click the video above to watch a full walkthrough of the dashboard including
> live filtering, drill-through, scatter plot analysis and date range switching.*

---

## 🌟 Why I Built This

Most stock dashboards either require expensive data subscriptions or only show
static snapshots. I wanted to build something that:

- **Looks and feels like a real financial terminal** — not a student project
- **Updates automatically every day** without any manual intervention
- **Tells a complete market story** — from individual stock performance down
  to sector-level risk and return analysis
- **Uses only free tools** — Python, Power BI Desktop, GitHub Actions

The result is **MarketPulse** — a fully automated market intelligence dashboard
that tracks the **Top 25 S&P 500 companies** by market capitalization, refreshed
daily with zero manual effort.

---

## 📸 Screenshots

| Main Dashboard | Risk vs Return |
|---|---|
| ![Main Dashboard](screenshots/01_main_dashboard.png) | ![Scatter Plot](screenshots/02_scatter_plot.png) |

| Gainers vs Losers | Volume Analysis |
|---|---|
| ![Bar Chart](screenshots/03_gainers_losers.png) | ![Volume](screenshots/04_volume_chart.png) |

---

## 🎯 Skills Demonstrated

| Category | Skills |
|---|---|
| **Data Engineering** | ETL pipeline, API integration, automated scheduling |
| **Data Modeling** | Star schema, DAX, calculated columns vs measures |
| **Visualization** | Financial dashboard design, conditional formatting |
| **Python** | pandas, yfinance, requests, data transformation |
| **Tools** | Power BI Desktop, GitHub Actions, Git |
| **Finance Domain** | Market cap, PE ratio, moving averages, volatility |

---

## ⚙️ How It Works — Architecture

```
┌─────────────────────────────────────────────────────┐
│                  EVERY DAY AT 6AM UTC               │
│                                                     │
│   GitHub Actions                                    │
│        │                                            │
│        ▼                                            │
│   Python Script (fetch_top25_sp500.py)              │
│        │                                            │
│        ├── Fetches Top 25 S&P 500 by Market Cap     │
│        ├── Downloads daily OHLCV price history      │
│        ├── Fetches fundamentals (PE, Market Cap)    │
│        │                                            │
│        ▼                                            │
│   Updated CSVs committed back to GitHub             │
│        │                                            │
│        ▼                                            │
│   Power BI Desktop reads latest CSVs               │
│   → One click refresh → Dashboard updates           │
└─────────────────────────────────────────────────────┘
```

---

## 🚀 Features

### 📊 Stock Screener Table
A Yahoo Finance–style master table showing all 25 tickers with:

| Metric | Description |
|---|---|
| Price | Latest closing price |
| Change / Change % | Daily price movement with green/red conditional formatting |
| Volume | Daily trading volume |
| Avg Volume (3M) | 3-month average volume with data bars |
| Market Cap | Total market capitalization |
| PE Ratio (TTM) | Trailing twelve months price-to-earnings ratio |
| 52W High / Low | 52-week price range |
| 52W Change % | Year-over-year performance |
| Daily Sparkline | Micro trend chart showing 90-day daily change pattern |

Includes **Top Gainers / Top Losers / Most Active** sorting — switchable
with one click using a tile slicer.

---

### 📉 Price Trend Chart with Moving Averages
A dual-layer line chart showing:
- **Current Price** — white line, primary focus
- **30-Day Moving Average** — blue, short-term trend
- **90-Day Moving Average** — orange, long-term trend

Paired with a **date range slicer** (3D / 7D / 1M / 6M / YTD / 1Y / All)
that dynamically adjusts both charts simultaneously.

---

### 📦 Volume Analysis Chart
A combo chart tracking:
- **Daily Volume** — column bars showing actual trading activity
- **Running Volume High** — line showing the cumulative peak volume,
  persisting forward until a new high is reached

This instantly flags unusual trading activity — a spike above the running
high signals institutional interest or major market events.

---

### 🎯 Risk vs Return Scatter Plot
Each of the 25 tickers plotted as a bubble where:
- **X-axis** = Volatility (standard deviation of daily returns)
- **Y-axis** = Total Return % since January 2023
- **Bubble size** = Average 3-month trading volume
- **Bubble color** = Sector (Technology, Healthcare, Financials etc.)

A reference line splits the chart at average volatility — instantly showing
which stocks delivered the best return for the least risk. This is the kind
of analysis used in portfolio management and risk assessment.

---

### 📊 Gainers vs Losers Bar Chart
A horizontal bar chart ranking all 25 tickers by daily Change %:
- 🟢 Green bars for positive movers
- 🔴 Red bars for negative movers

Gives an instant full-market snapshot that communicates performance
across all stocks in under 2 seconds — something the table cannot do alone.

---

### 🗓️ Dynamic Date Range Slicer
Switch between time periods with one click:

```
[ 3D ]  [ 7D ]  [ 1M ]  [ 6M ]  [ YTD ]  [ 1Y ]  [ All ]
```

All charts update simultaneously. Built using a disconnected DAX table
and dynamic filter measures — no Pro license required.

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| **Python** | Data extraction and transformation |
| **yfinance** | Fetches stock prices and fundamentals from Yahoo Finance |
| **pandas** | Data cleaning and CSV generation |
| **requests** | Wikipedia scraping for S&P 500 company list |
| **Power BI Desktop** | Dashboard design, DAX modeling, visualization |
| **GitHub Actions** | Daily automated pipeline scheduling |
| **GitHub** | Version control and CSV storage |

---

## 📐 Data Model

The core data model follows a **star schema** — the industry standard
for analytical data modeling. Two additional disconnected tables sit
outside the schema to power interactive UI features without affecting
the data relationships.

```
                    ┌──────────────┐
                    │   Dim_Date   │
                    │──────────────│
                    │ Date         │
                    │ Year         │
                    │ Month        │
                    │ MonthNum     │
                    │ YearMonth    │
                    │ DayOfWeek    │
                    └──────┬───────┘
                           │ 1
                           │
                           │ *
              ┌────────────▼──────────────┐
              │       stock_prices        │◄───────────┐
              │───────────────────────────│            │
              │ Date                      │            │ *
              │ Ticker                    │   ┌────────┴──────┐
              │ Open                      │   │   Dim_Stock   │
              │ High                      │   │───────────────│
              │ Low                       │   │ StockID       │
              │ Close                     │   │ Ticker      1 │
              │ Volume                    │   │ CompanyName   │
              │ Previous Close (Col)      │   │ Sector        │
              │ Daily Change (Col)        │   │ Industry      │
              │ Daily Return (Col)        │   └───────────────┘
              └───────────────────────────┘
                           │ *
                           │
                           │ 1
                    ┌──────┴────────┐
                    │ Fundamentals  │
                    │───────────────│
                    │ Ticker        │
                    │ Company Name  │
                    │ Market Cap    │
                    │ PE Ratio(TTM) │
                    │ Avg Vol (3M)  │
                    └───────────────┘


  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
  DISCONNECTED TABLES (no relationships — UI only)
  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─

  ┌─────────────────────┐     ┌─────────────────────┐
  │    Date Range       │     │   Sort Selector     │
  │─────────────────────│     │─────────────────────│
  │ Range               │     │ Sort Type           │
  │ (3D,7D,1M,6M,       │     │ (Top Gainers,       │
  │  YTD,1Y,All)        │     │  Top Losers,        │
  │                     │     │  Most Active)       │
  │ Powers → Dynamic    │     │ Powers → Dynamic    │
  │ date range slicer   │     │ table sorting       │
  └─────────────────────┘     └─────────────────────┘
```

> **Why disconnected tables?**
> These tables are intentionally not linked to the star schema.
> They act as parameter inputs — DAX measures read their selected
> values via `SELECTEDVALUE()` to dynamically control filtering
> and sorting without modifying the underlying data model.
> This is a standard Power BI design pattern for interactive dashboards.

---

## 🧮 Key DAX Measures

```dax
-- Previous Close (handles weekends and market holidays)
Previous Close =
VAR CurrDate = MAX(stock_prices[Date])
VAR CurrTicker = SELECTEDVALUE(stock_prices[Ticker])
VAR PrevDate =
    CALCULATE(
        MAX(stock_prices[Date]),
        TOPN(
            1,
            FILTER(
                stock_prices,
                stock_prices[Ticker] = CurrTicker &&
                stock_prices[Date] < CurrDate
            ),
            stock_prices[Date], DESC
        )
    )
RETURN
CALCULATE(
    MAX(stock_prices[Close]),
    stock_prices[Date] = PrevDate,
    stock_prices[Ticker] = CurrTicker
)

-- Running Volume High (cumulative maximum, persists until new high)
Running Volume High =
VAR CurrDate = MAX(stock_prices[Date])
VAR CurrTicker = SELECTEDVALUE(stock_prices[Ticker])
RETURN
CALCULATE(
    MAX(stock_prices[Volume]),
    FILTER(
        ALL(stock_prices),
        stock_prices[Ticker] = CurrTicker &&
        stock_prices[Date] <= CurrDate
    )
)

-- Volatility (standard deviation of daily returns per ticker)
Volatility =
CALCULATE(
    STDEV.P(stock_prices[Daily Return (Col)]),
    ALL(stock_prices[Date])
)

-- Dynamic Sort (powers Top Gainers / Losers / Most Active slicer)
Dynamic Sort =
SWITCH(
    SELECTEDVALUE('Sort Selector'[Sort Type], "Top Gainers"),
    "Top Gainers",  [Change %],
    "Top Losers",  -[Change %],
    "Most Active",  [Volume],
    [Change %]
)

-- Dynamic Date Filter (powers date range slicer)
Date Filter =
IF(
    MAX(stock_prices[Date]) >= [Range Start Date],
    1,
    0
)
```

---

## 🧠 Key Learnings & Challenges

This project pushed me well beyond standard tutorials.
Here are the real problems I solved along the way:

❌ **Problem:** `DATEADD` broke on weekends and market holidays
✅ **Solution:** Built a custom previous-close DAX using `TOPN` to always
find the last available trading day regardless of calendar gaps —
the same logic used in professional financial BI systems

❌ **Problem:** Power BI sparklines silently failed with dynamic measures
✅ **Solution:** Discovered that sparklines require physical calculated
columns rather than measures — leading to a deeper understanding
of row context vs filter context in DAX

❌ **Problem:** Wikipedia blocked automated data requests with HTTP 403
✅ **Solution:** Implemented browser-style User-Agent headers to bypass
the block — a standard real-world web scraping technique

❌ **Problem:** GitHub Actions failed with "nothing to commit" error
✅ **Solution:** Added conditional `git diff` logic to exit cleanly
when market data hasn't changed — pipeline never breaks on non-trading days

❌ **Problem:** Power BI Free version hides selected slicer state formatting
✅ **Solution:** Applied a custom dark theme JSON file that controls
all visual states globally — including slicer selected/unselected colors

---

## 📁 Repository Structure

```
stock-market-dashboard/
│
├── 📄 fetch_top25_sp500.py       # Main data extraction script
│
├── 📁 .github/
│   └── 📁 workflows/
│       └── daily_refresh.yml     # GitHub Actions automation
│
├── 📁 data/
│   ├── stock_prices.csv          # Daily OHLCV price history
│   ├── fundamentals.csv          # Market cap, PE ratio, avg volume
│   └── dim_stock.csv             # Company name, sector, industry
│
├── 📁 screenshots/
│   ├── 01_main_dashboard.png
│   ├── 02_scatter_plot.png
│   ├── 03_gainers_losers.png
│   └── 04_volume_chart.png
│
├── 📁 demo/
│   └── marketpulse_demo.mp4      # Full dashboard walkthrough video
│
├── 📄 MarketPulse.pbix           # Power BI dashboard file
├── 📄 StockDashboard_DarkTheme.json  # Custom Power BI dark theme
└── 📄 README.md
```

---

## 🔄 Automated Pipeline Setup

The data pipeline runs automatically every day at **6:00 AM UTC** via
GitHub Actions — no manual intervention required:

1. GitHub spins up a virtual Ubuntu machine
2. Python and required libraries are installed
3. `fetch_top25_sp500.py` fetches fresh market data from Yahoo Finance
4. Updated CSVs are committed back to this repository automatically
5. Power BI Desktop reads the latest CSVs on next open

**To refresh the dashboard after the pipeline runs:**

```
Open MarketPulse.pbix → Press Alt + F5
```

---

## 🏃 How to Run Locally

**1. Clone the repository**
```bash
git clone https://github.com/yourusername/stock-market-dashboard.git
cd stock-market-dashboard
```

**2. Install dependencies**
```bash
pip install yfinance pandas requests lxml
```

**3. Run the data extraction script**
```bash
python fetch_top25_sp500.py
```

**4. Open the dashboard**
```
Open MarketPulse.pbix in Power BI Desktop (free)
Press Alt + F5 to refresh data
```

---

## 📊 Data Sources

| Source | Data |
|---|---|
| Yahoo Finance (via yfinance) | Daily stock prices, fundamentals |
| Wikipedia | S&P 500 company list and sector classification |

> **Disclaimer:** This project is built for portfolio and educational
> purposes only. All data is sourced from publicly available free APIs.
> This is not financial advice.

---

## 👤 Author

**Your Name**
📧 your.email@gmail.com
💼 [LinkedIn](https://linkedin.com/in/yourprofile)
🐙 [GitHub](https://github.com/yourusername)

---

## ⭐ If you found this project interesting, please consider giving it a star!

---

*Built with 🐍 Python + 📊 Power BI + ⚡ GitHub Actions*
