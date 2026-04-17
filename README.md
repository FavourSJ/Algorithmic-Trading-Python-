# Algorithmic-Trading-Python-

# Overview
This project is my personal implementation of all three algorithmic trading strategies taught in Nick McCullum's freeCodeCamp course. The course covers the fundamentals of quantitative finance and demonstrates how to use Python to automate investment analysis and produce actionable trade recommendations.
All three strategies culminate in an Excel trade recommendation file — a spreadsheet that tells you exactly how many shares to buy of each stock given a portfolio size of your choice.
The original course was built around the IEX Cloud API, a paid financial data service. I rewrote the data-fetching layer entirely using yfinance, an open-source library that pulls real market data directly from Yahoo Finance — no API key, no cost, no account needed.

# Key Adaptation: yfinance over IEX Cloud
The most significant engineering challenge in this project was replacing all IEX Cloud API calls with yfinance. This wasn't a simple swap — it required rethinking how data was fetched, structured, and fed into the rest of the pipeline.
# Why the change was non-trivial
The original course built its data pipeline around IEX Cloud's batch request endpoint, which let you fetch data for hundreds of stocks in a single API call, returning a structured JSON blob. yfinance works differently:

No native batch endpoint — data must be fetched per-ticker or via yf.download() for price history, with separate calls for fundamentals.
Different data structures — IEX returns flat JSON; yfinance returns pandas DataFrames with multi-level column headers when fetching multiple tickers.
Different field names — metric names (e.g. price-to-earnings ratio, price-to-book, one-year return) are named differently and live in different objects (ticker.info, ticker.fast_info, ticker.history()).
Rate limiting — fetching data for all 500+ S&P 500 constituents requires thoughtful handling to avoid hitting Yahoo Finance's informal rate limits.

# How I handled it

Used yf.Ticker(symbol).info to retrieve fundamental metrics (market cap, P/E ratio, P/B ratio, etc.) for each stock.
Used yf.download() with a list of tickers to efficiently pull historical price data for momentum calculations.
Rebuilt the momentum return calculations (1-month, 3-month, 6-month, 1-year) by computing percentage changes directly from the downloaded OHLCV DataFrames.
Replaced all JSON-parsing logic with pandas DataFrame operations to extract and clean the relevant fields.
Added error handling for tickers that yfinance cannot resolve (delistings, symbol changes, etc.) — these were skipped gracefully rather than crashing the script.

# Projects
# 1. Equal-Weight S&P 500 Index Fund
Concept: The S&P 500 is a market-cap-weighted index, meaning the largest companies (Apple, Microsoft, etc.) have an outsized influence on performance. This project builds an equal-weight alternative — every company in the index gets the same allocation, regardless of size.
What the script does:

Reads in a list of all current S&P 500 constituents.
Fetches the latest market price for each stock using yfinance.
Accepts a user-inputted portfolio value (e.g. £10,000).
Calculates how many shares of each stock to purchase so that every holding is worth exactly portfolio_value / 500.
Exports the results to a formatted Excel file.

# Key concepts covered:

Working with pandas DataFrames
Looping through tickers and building a DataFrame row by row
Using xlsxwriter to produce a formatted, colour-coded Excel output
Equal-weighting logic and position sizing


# 2. Quantitative Momentum Screener
Concept: Momentum investing is the idea that assets which have performed well recently tend to continue outperforming in the near term. This strategy buys the top 50 momentum stocks in the S&P 500.
What the script does:

Fetches historical price data for all S&P 500 tickers using yf.download().
Calculates each stock's price return over four time horizons:

1 month (approximately 21 trading days)
3 months (approximately 63 trading days)
6 months (approximately 126 trading days)
1 year (approximately 252 trading days)


For each metric, assigns a percentile rank across all 500 stocks.
Computes a High Quality Momentum (HQM) Score — the average of all four percentile ranks — to identify stocks with consistent momentum across multiple timeframes rather than a single lucky spike.
Selects the top 50 HQM stocks and calculates the number of shares to buy for a given portfolio size.
Exports to a colour-coded Excel file.

# Key concepts covered:

Multi-timeframe return calculation with pandas
Percentile ranking with scipy.stats.percentileofscore
Composite scoring (the HQM Score)
The importance of multi-metric momentum vs. single-metric momentum


# 3. Quantitative Value Screener
Concept: Value investing seeks to identify companies that are trading below their intrinsic worth. Rather than relying on a single ratio (like P/E), this strategy uses a composite of five valuation metrics to find robustly cheap stocks — reducing the risk of falling into a "value trap".
What the script does:

Fetches fundamental data for all S&P 500 tickers using yfinance's .info dictionary.
Collects five valuation metrics for each stock:

Price-to-Earnings (P/E) — how much investors pay per £1 of earnings
Price-to-Book (P/B) — how much investors pay relative to the company's book value
Price-to-Sales (P/S) — market cap relative to revenue
Enterprise Value / EBITDA (EV/EBITDA) — a debt-adjusted profitability measure
Enterprise Value / Gross Profit (EV/GP) — another profitability-adjusted valuation ratio


Calculates percentile ranks for each metric, where a lower ratio = a higher percentile (i.e. cheaper = better).
Computes a Robust Value (RV) Score — the average of all five percentile ranks.
Selects the top 50 RV stocks and sizes positions for the given portfolio.
Exports to Excel.

# Key concepts covered:

Sourcing multiple fundamental metrics from yfinance
Handling missing/None data gracefully (common with yfinance fundamentals)
Composite value scoring
Why five metrics outperforms any single ratio
