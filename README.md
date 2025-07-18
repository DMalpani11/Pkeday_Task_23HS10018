Insider Trading Momentum Strategy: Backtest and Analysis
Data Sources Used
Primary Data: Insider trading disclosures from the National Stock Exchange (NSE) of India cover the past three years. The dataset, in a CSV file named 'raw_nse_insider.csv', includes transaction dates, stock symbols, insider names, categories, quantities, values, transaction types, and modes.
Supplementary Data: Retrieved historical stock price data (OHLCV) and market metrics using the yfinance library from Yahoo Finance. This included Nifty 50 benchmark data ('^NSEI') for relative performance calculations and individual stock data with NSE suffixes (e.g., 'SYMBOL.NS'), along with BSE fallbacks (‘SYMBOLS.BO) when necessary.
Time Period: Analysis ranges from 90 days before the earliest broadcast date (for look back) to 60 days after the latest date (for forward simulation), providing historical context for backtesting.
Summary of Data Cleaning Process
Initial Loading and Profiling: Loaded the CSV file and profiled it to assess structure (e.g., row/column counts, unique values). Stripped whitespace from column names and renamed for consistency (e.g., 'SYMBOL' to 'symbol', 'NO. OF SECURITIES (ACQUIRED/DISPLOSED)' to 'quantity').
Handling Inconsistencies: Replaced invalid placeholders ('-', 'Nil') with NaN. Formatted dates to datetime format (day-first parsing) and numeric fields (quantity, value, post_holding_pct) by removing commas and coercing to floats.
Validity Filters: Removed rows with missing critical fields (symbol, dates, quantity, value, transaction_type). Ensured positive quantity/value, calculated derived price (value/quantity), and filtered for positive finite prices.
Transaction-Specific Filters: Retained only 'Buy' and 'Sell' transactions. Focused on high-signal insider categories (Promoters, Directors, Promoter Group). Limited to market-based modes (Market Purchase/Sale). Grouped by symbol and broadcast date, filtering for ≥1 unique insiders per group.
Deduplication and Output: Sorted by broadcast date, removed duplicates, and saved to 'cleaned_insider_data.csv'.
Data Enrichment: Loaded cleaned data and merged with market metrics (e.g., atr_10, adv_10, volatility_60, beta_60, market_cap_cr) from yfinance using helper functions:
sanity_check_price_data: Flattened MultiIndex, verified essential columns (Open, High, Low, Close, Adj Close, Volume), forward-filled gaps, and handled zeros.
calculate_market_metrics: Computed technical indicators using rolling windows (e.g., 10-day ATR, 60-day beta/volatility) and fetched market cap.
Processed symbols in parallel with concurrent futures, merging via pd.merge_asof (backwards direction to prevent lookahead bias). Dropped NaNs in metrics.
Outcome: Reduced dataset to a focused, enriched set emphasizing valid buy signals, with dynamic row counts tracked at each step for transparency.
Strategy Description
This momentum-driven strategy leverages insider buy signals as entry triggers, aiming to capture post-disclosure price upside while incorporating risk controls. It employs simple, rule-based logic for entries, exits, and sizing, benchmarked against Nifty 50.

Entry Rules:
Trigger on broadcast dates with ≥1 unique insider buys.
Require at least 1 historical signal per stock; historical win rate ≥30% and average 30-day return > -2%.
Enter long positions the next trading day at open price.
Position sizing: 5-8% of current portfolio value, reduced for high-beta stocks (>1.5) to manage risk.
Limit to max 20 active positions (increased for diversification and trade volume).
Exit Rules:
Profit target: Entry price × (1 + max(historical average return, 15%))—aggressive to boost returns.
Stop loss: Entry price × (1 - 4 × historical average volatility), to minimize premature exits.
Trailing stop: Peak price - 1.5 × atr_10 (for greater upside potential).
Momentum decay: Exit after min 20-day hold if no new high in 15-25 days (adaptive based on volatility_60).
Max hold: 120 days to cap exposure.
Risk Management: Applies 0.1% transaction costs (reduced for realism); enables compounding by reinvesting exit proceeds. Integrates technical metrics (ATR, beta, volatility) for adaptive sizing and exits, with no short-selling.
Backtest Results
The backtest simulated trading over the enriched dataset period, starting with ₹1 crore initial capital. Metrics are hypothetical, net of costs, and equity curve-based for accuracy.
Portfolio Performance:
Total Trades: 247
Overall Compounded Return: 215.79%
Compounded Annual Growth Rate (CAGR): 47.48%
Geometric Mean Return per Trade: 6.54%
Average Holding Days: 26.7
Overall Annualized Sortino Ratio: 0.50 (focuses on downside risk)
Overall Annualized Calmar Ratio: 5.25 (return relative to max drawdown)
Technical Indicators Summary:
Average ATR: 22.954
Average Beta: 0.86
Average Volatility: 0.448
Average ADV: 22.6 Crores
Key Insights and Conclusions
What Worked: The momentum-based exit strategies, including the adaptive momentum decay rule and trailing stops tied to the Average True Range (ATR), successfully captured gains in winning trades, resulting in a compounded return of 215.79% and a CAGR of 47.48%. Incorporating technical metrics like beta for position sizing and volatility for stop-loss adjustments enhanced risk control, evident in the strong Calmar ratio of 5.25, which balances high returns with drawdowns.
What Didn't: The fixed profit targets and stop-loss rules, while adjusted for historical volatility, failed to effectively reduce downside risks in highly volatile stocks, resulting in a Sortino ratio of only 0.50 and underperformance in bearish or low-momentum markets. Furthermore, the requirement for at least one prior event with a 30% win rate limited entries in newer or less-frequent stocks, potentially causing missed opportunities and skipped trades due to data gaps from Yahoo Finance.
Commercial Potential: This approach shows promising alpha potential for quantitative funds or retail traders in India's underutilised insider disclosure ecosystem, potentially outperforming benchmarks like the Nifty 50. However, real-world factors (slippage, taxes, SEBI regulations) could reduce edges; the moderate Sortino highlights risks in live deployment. Recommend forward-testing with small capital, integrating machine learning for signal enhancement, or combining with fundamental filters for robustness. Overall, it's a viable niche strategy, but best as a portfolio component rather than a standalone.


