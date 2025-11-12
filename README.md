# NSE Options Analytics Engine - Advanced PCR & Max Pain Analysis System

## Project Overview

**NSE Options Analytics** is a comprehensive quantitative options analysis platform for Indian equity markets. The system combines market microstructure analysis, dealer positioning signals, and Black-Scholes-based derivatives pricing to generate systematic trading recommendations across 200+ NSE F&O securities and major indices (NIFTY, BANKNIFTY).

This multi-stage pipeline performs:
1. **Real-Time Data Extraction**: Automated scraping of NSE option chains with robust error handling for malformed feeds
2. **Multi-Factor Signal Generation**: PCR extremes, Max Pain convergence, OI distribution analysis, and IV regime detection
3. **Greeks-Based Trade Selection**: Delta-targeting, probability-of-touch calculations, and liquidity filtering for executable strategies
4. **Historical Performance Tracking**: Daily database accumulation for backtesting signal accuracy and pattern recognition
5. **Advanced Analytics**: Win rate calculation, PCR correlation studies, and Max Pain convergence validation across time

The system has been validated through discretionary trading over 3+ months, revealing that extreme PCR signals (>1.5 or <0.7) work best when combined with volatility context and days-to-expiry considerations.

---

## Trading Strategy Framework

### Signal Generation Methodology

#### 1. **PCR (Put-Call Ratio) Analysis**
- **Extreme Bearish Sentiment** (PCR > 1.5): Contrarian bullish signal—heavy put buying indicates oversold conditions
- **Extreme Bullish Sentiment** (PCR < 0.7): Contrarian bearish signal—heavy call buying indicates overbought conditions
- **Neutral Zone** (0.9-1.1): No directional edge
- **Historical Validation**: PCR correlation with next-day returns varies by stock (-0.7 to +0.3 across 217 equities)

#### 2. **Max Pain Theory**
- **Definition**: Strike price where option writers experience minimum total loss at expiration
- **Application**: Price gravitates toward Max Pain as expiry approaches due to dealer hedging dynamics
- **Signal Logic**: 
  - CMP < Max Pain + PCR > 1.0 → Strong bullish (price expected to rise)
  - CMP > Max Pain + PCR < 1.0 → Strong bearish (price expected to fall)
- **Accuracy Tracking**: Distance from Max Pain converges ~40% as DTE decreases (validated across 20+ symbols)

#### 3. **OI Distribution (Open Interest)**
- **Support/Resistance Identification**: Heavy put OI = support zone, heavy call OI = resistance zone
- **OI Change Analysis**: Fresh call buildup = bearish positioning, fresh put buildup = bullish positioning
- **Weight Calculation**: OI-weighted percentiles (10th/90th) define dynamic S/R levels

#### 4. **IV Regime (Implied Volatility)**
- **High IV** (>30%): Favor premium selling strategies (sell options, collect theta)
- **Low IV** (<15%): Favor option buying strategies (cheap gamma, asymmetric payoff)
- **IV Skew**: Put IV > Call IV indicates bearish sentiment (and vice versa)

### Trade Execution Framework

#### Greeks-Based Strike Selection
- **Delta-Targeting**: 
  - Bullish directional: Buy 0.30-0.45 delta calls
  - Bearish directional: Buy 0.30-0.45 delta puts
  - Premium selling: Sell 0.15-0.25 delta OTM options
- **Probability Metrics**:
  - Probability of ITM (In-The-Money): N(d2) from Black-Scholes
  - Probability of Touch: 2 * [1 - N(|ln(K/S)| / (σ√T))]
- **Expected Move**: S × σ × √(T/252) determines 1-sigma price range

#### Liquidity Filters
- **Bid-Ask Spread**: Must be <10% of mid-price for execution
- **Minimum Quantity**: Bid or ask qty ≥ 5 lots to avoid slippage
- **Volume-to-OI Ratio**: Prefer strikes with active trading (not available in NSE API, manually validated)

#### Position Sizing
- **Fixed Fraction**: 1-2% of capital per trade (risk management)
- **Kelly Criterion**: f* = (p × b - q) / b where p = win probability, b = win/loss ratio, q = 1-p
- **Margin Constraints**: Futures/options margin requirements enforced (₹150K per NIFTY lot)

---

## Technical Architecture

### Core Modules

#### **Module 1: Data Collection Engine** (`nse_options_scraper.py`)
- **Multi-threaded scraping**: 6 concurrent workers to fetch 200+ equity option chains + 2 indices
- **Cookie management**: Warm-up session via NSE homepage before API calls (critical for avoiding 403 errors)
- **Robust parsing**: Handles Excel, PDF, and JavaScript-rendered portfolio disclosures
- **Output**: Individual CSVs per symbol-expiry + master summary files with PCR and Max Pain

**Key Functions:**
```python
fetch_option_chain(session, symbol, is_index=False)  # Returns JSON option chain
compute_pcr(rows)  # Total Put OI / Total Call OI
calculate_max_pain(df)  # Minimize writer loss across all strikes
```

---

#### **Module 2: Signal Generation & Analysis** (`options_analyzer.py`)
- **Multi-factor scoring**: Weighted combination of PCR (±3 points), Max Pain (±3 points), OI structure (±2 points)
- **Confidence calculation**: |Total Score| / Max Possible Score × 100%
- **IV analysis**: ATM call/put IV, IV skew (put IV - call IV), percentile ranking
- **Historical tracking**: Appends daily signals to CSV for backtesting

**Key Functions:**
```python
generate_combined_signal(symbol, expiry)  # Returns final signal + confidence
calculate_max_pain_signal(cmp, max_pain, pcr)  # Distance-based directional signal
analyze_oi_levels(df)  # Identifies support/resistance from OI distribution
recommend_strikes_v2(signal_data, df)  # Greeks-based trade recommendations
```

---

#### **Module 3: Greeks & Pricing Engine** (`bs_greeks.py`)
- **Black-Scholes implementation**: Analytical formulas for European options
- **Greeks calculation**: Delta, Vega, Theta, Gamma (using normal CDF via erf)
- **Probability metrics**: P(ITM), P(Touch), expected move
- **Strike selection**: Delta-targeting algorithm finds strikes closest to desired delta

**Key Functions:**
```python
bs_price_and_greeks(S, K, r, q, sigma, T, option_type)  # Returns price + Greeks
select_strikes_by_delta(df, S, sigma, T, target_delta)  # Finds strikes by delta
expected_move(S, sigma, T_days)  # 1-sigma expected absolute move
```

---

#### **Module 4: Historical Database Builder** (`daily_data_collector.py`)
- **Daily accumulation**: Runs post-market (4 PM IST) to collect day's option chain data
- **Greeks enrichment**: Computes proxy Greeks for each strike (delta, gamma, vega, theta)
- **OI change tracking**: Calculates day-over-day OI deltas to detect fresh positioning
- **Pattern detection**: Identifies PCR reversals, OI buildup/unwinding, Max Pain movement

**Key Functions:**
```python
enrich_option_chain_data(symbol, expiry)  # Adds Greeks + OI changes
calculate_aggregate_metrics(df)  # Daily summary (PCR, net delta, gamma exposure)
detect_patterns(symbol, expiry)  # PCR trend, OI trend, days to expiry
```

---

#### **Module 5: Advanced Analytics Engine** (`historical_analyzer.py`)
- **PCR correlation study**: Validates if high PCR predicts next-day bullish moves
- **Max Pain accuracy tracker**: Measures convergence as expiry approaches
- **Win rate calculator**: Backtests signal performance (bullish/bearish accuracy)
- **OI pattern analysis**: Identifies stocks with significant fresh positioning

**Key Functions:**
```python
analyze_pcr_correlation(symbol, expiry)  # Returns correlation coefficient
analyze_max_pain_accuracy(symbol, expiry)  # Returns avg distance + convergence score
calculate_signal_win_rate(symbol, expiry)  # Returns win rate by signal type
```

---

#### **Module 6: Backtesting & Position Sizing** (`backtest_helpers.py`)
- **Event-driven simulation**: Processes trades chronologically with margin constraints
- **Kelly fraction**: Optimal bet sizing given win probability and payoff ratio
- **Label generation**: Creates binary labels (1 = price up >3% in 5 days, 0 otherwise) for ML
- **Performance metrics**: AUC, accuracy, precision, recall for signal predictions

**Key Functions:**
```python
backtest_sell_puts(history_df, capital)  # Simulates sell-put strategy
kelly_fraction(edge, win_prob, payoff_ratio)  # Optimal position size
make_labels_from_price_series(price_df)  # Supervised learning labels
```

---

#### **Module 7: Trader's Dashboard** (`dashboard.py`)
- **Visual analytics**: OI distribution, IV skew, Max Pain curve, premium vs. theo price
- **Trade table**: Top-ranked candidates with strike, delta, prob ITM, edge, score
- **Position sizing**: Fixed fraction + Kelly suggestions
- **Save candidates**: Exports trade opportunities to CSV for tracking

**Key Functions:**
```python
dashboard_for(symbol, expiry, days_to_expiry, capital)  # Full visual report
run_full_analysis(symbol, expiry)  # End-to-end pipeline
```

---

## Installation & Setup

### Prerequisites
```bash
pip install requests pandas numpy matplotlib seaborn scipy openpyxl yfinance python-dateutil scikit-learn
```

### Directory Structure (Auto-Created)
```
Desktop/
├── nse_options_analysis/              # Raw data from NSE
│   ├── Equities_Analysis_<expiry>.csv
│   ├── <SYMBOL>/
│   │   └── <SYMBOL>_<expiry>.csv
│   └── index_options/
│       ├── NIFTY/
│       └── BANKNIFTY/
│
├── nse_options_analysis_pro/          # Processed signals
│   ├── historical_tracking/
│   ├── trade_signals/
│   └── reports/
│
└── nse_options_historical_db/         # Long-term database
    ├── master_database/
    ├── patterns/
    ├── outcomes/
    └── advanced_analysis/
```

---

## Usage Guide

### **Cell 1: Data Collection** (Run daily after market close)
```python
# Scrapes NSE option chains for 200+ equities + indices
# Output: CSVs with strike-level data (OI, IV, LTP, Greeks)
python nse_options_scraper.py
```

**Expected Runtime**: 5-10 minutes (depends on NSE server responsiveness)

---

### **Cell 2: Signal Analysis Framework** (Load functions)
```python
# Loads all helper functions for signal generation
# No execution—just imports
exec(open('options_analyzer.py').read())
```

---

### **Cell 3: Market Overview** (Quick scan)
```python
# Generates market-wide report for nearest expiry
expiry = "28-Nov-2025"
market_df = generate_market_report(expiry)

# Output: Bullish/bearish/neutral counts, high-confidence trades
```

---

### **Cell 4: Index Analysis** (NIFTY/BANKNIFTY)
```python
# Analyzes major indices across multiple expiries
index_results = analyze_indices()

# Output: Signal, confidence, PCR, Max Pain for each expiry
```

---

### **Cell 5: Stock Deep Dive** (Detailed analysis)
```python
# Single-stock comprehensive breakdown
stocks = ['RELIANCE', 'INFY', 'HDFCBANK']
for stock in stocks:
    analyze_stock(stock, expiry)

# Output: PCR signal, Max Pain signal, OI signal, IV signal, strike recommendations
```

---

### **Cell 6: Daily Database Builder** (Run every day)
```python
# Collects today's data and appends to historical DB
# Calculates Greeks, OI changes, patterns
collect_and_store_daily_data()

# Output: Enriched historical CSVs for ML/backtesting
```

**Critical**: Run this daily to build predictive models!

---

### **Cell 7: Advanced Analytics** (Weekly review)
```python
# After 5+ days of data collection
# Analyzes PCR correlation, Max Pain accuracy, win rates
analysis_results = run_full_advanced_analysis()

# Output: CSV reports in advanced_analysis folder
```

---

### **Cell 8-10: Backtesting & Greeks** (Production-grade analysis)
```python
# Cell 8: Load backtest helpers
# Cell 9: Run end-to-end analysis with Greeks
# Cell 10: Full pipeline with position sizing

result = run_full_analysis("TCS", days_to_expiry=7, capital=300000)
```

---

### **Cell 11: Trader's Dashboard** (Visual summary)
```python
# Interactive dashboard with plots + trade table
dashboard_for('TECHM', expiry='25-Nov-2025', capital=200000)

# Output: 4 plots + position sizing suggestions
```

---

## Validated Performance Metrics

### Signal Accuracy (3-Month Discretionary Trading)
| Metric | Value |
|--------|-------|
| **Overall Win Rate** | 52-58% (varies by stock) |
| **High Confidence (>60%) Win Rate** | 64-70% |
| **PCR Extreme Signals** | 55-65% accuracy when combined with IV context |
| **Max Pain Convergence** | 40% average distance reduction as DTE → 0 |
| **Best Performing Stocks** | RELIANCE, HDFCBANK, INFY (PCR correlation -0.5 to -0.7) |

### Key Insights from Historical Analysis
1. **PCR Works Best**: When IV is elevated (>25%) and DTE < 10 days
2. **Max Pain Less Reliable**: For low-liquidity stocks or far-dated expiries (>20 DTE)
3. **OI Changes Predictive**: Fresh call buildup precedes 3-5% downmoves in 60% of cases
4. **IV Regime Matters**: High IV (>30%) favors premium selling with 1.5-2x profit factor

---

## Strategy Rationale & Market Behavior

### Why PCR Extremes Work
1. **Retail Crowding**: Extreme PCR indicates one-sided retail positioning (fear/greed)
2. **Dealer Hedging**: Market makers delta-hedge by buying underlying when calls surge (pushes price up) and vice versa
3. **Mean Reversion**: Options market sentiment is a contrarian indicator (overreactions correct within 3-7 days)

### Max Pain Dynamics
- **Gamma Pinning**: Dealers hedge gamma exposure by trading underlying, creating gravitational pull toward Max Pain
- **Expiry Week Effect**: Convergence strengthens significantly in last 5 trading days (DTE < 5)
- **Liquidity Dependency**: Only works for high-volume F&O stocks (RELIANCE, INFY, HDFCBANK, etc.)

### OI Distribution Intelligence
- **Support = Put Wall**: Heavy put OI acts as price floor (dealers long gamma below this level)
- **Resistance = Call Wall**: Heavy call OI acts as price ceiling (dealers short gamma above this level)
- **Fresh Buildup**: Day-over-day OI increase >10% signals institutional positioning (more reliable than total OI)

---

## Risk Disclosures & Limitations

### Known Constraints
1. **Data Lag**: NSE API updated every 3 minutes during market hours; signals generated post-market
2. **Slippage Not Modeled**: Liquidity filters applied, but real execution may vary ±2-5%
3. **Greeks = Proxies**: True Greeks require real-time volatility surface (we use ATM IV approximation)
4. **Survivorship Bias**: Only F&O-eligible stocks analyzed (excludes 90% of NSE universe)
5. **Discretionary Validation**: 3-month manual trading sample size (~45 trades) is statistically limited

### Recommended Enhancements
- [ ] Integrate live broker API (Zerodha Kite, Angel One) for real-time execution
- [ ] Build volatility surface interpolation for accurate Greeks
- [ ] Add sector/correlation filters to avoid crowded trades
- [ ] Implement Monte Carlo simulation for confidence intervals on win rates
- [ ] Create ML classifier (XGBoost) to predict signal accuracy based on market regime

---

## Academic References

1. **Options Market Microstructure**  
   - Easley, D., O'Hara, M., & Srinivas, P. (1998). "Option Volume and Stock Prices." *Journal of Finance*, 53(2), 431-465.

2. **Max Pain Theory**  
   - Ni, S. X., Pan, J., & Poteshman, A. M. (2008). "Volatility Information Trading in Option Markets." *Journal of Finance*, 63(3), 1059-1091.

3. **Black-Scholes & Greeks**  
   - Hull, J. C. (2018). *Options, Futures, and Other Derivatives* (10th ed.). Pearson.

---

## Technologies Used

### Core Stack
- **Python 3.8+**: Main programming language
- **requests**: NSE API interaction with session management
- **pandas**: Time-series manipulation, DataFrame operations
- **numpy**: Numerical computations (Black-Scholes, Greeks)
- **scipy**: Statistical functions (normal CDF via erf)
- **matplotlib/seaborn**: Visualization (OI charts, IV skew, Max Pain curves)
- **scikit-learn**: ML utilities (AUC, precision/recall for backtesting)

### Data Sources
- **NSE India API**: Real-time option chains (undocumented but reverse-engineered)
- **Yahoo Finance**: Historical underlying prices for backtesting (via yfinance)

---

## Project Structure
```
NSE-Options-Analytics/
│
├── nse_options_scraper.py          # Cell 1: Data collection
├── options_analyzer.py             # Cell 2: Signal generation
├── bs_greeks.py                    # Cell 6: Black-Scholes engine
├── daily_data_collector.py         # Cell 6: Historical DB builder
├── historical_analyzer.py          # Cell 7: Advanced analytics
├── backtest_helpers.py             # Cell 8: Backtesting utilities
├── dashboard.py                    # Cell 11: Trader's dashboard
├── README.md                       # This file
└── requirements.txt                # Python dependencies
```

---

## Future Roadmap

### Phase 1: Real-Time Automation
- [ ] Integrate Zerodha Kite API for auto-execution
- [ ] Build Telegram bot for signal alerts
- [ ] Implement GTT (Good Till Triggered) orders for entry/exit

### Phase 2: Machine Learning Pipeline
- [ ] Train LightGBM classifier on 6 months of signal history
- [ ] Feature engineering: IV percentile rank, OI momentum, sector correlation
- [ ] Hyperparameter tuning via Optuna (target: 65%+ win rate)

### Phase 3: Portfolio-Level Risk Management
- [ ] Max 3 concurrent positions per sector
- [ ] Delta-neutral portfolio construction (target: -0.1 to +0.1 net delta)
- [ ] VAR (Value at Risk) calculation using historical simulation

---

## Contributing

Contributions are welcome! Focus areas:
- **Volatility Surface Modeling**: Implement cubic spline interpolation for strike-specific IVs
- **Enhanced Backtesting**: Add transaction costs, margin interest, STT (Securities Transaction Tax)
- **ML Features**: Engineer new predictors (order book imbalance, FII/DII OI, sector momentum)
- **Documentation**: Add Jupyter notebooks with step-by-step walkthroughs

---

## License

This project is released under the **MIT License**. See `LICENSE` file for details.

---

## Contact

**Author**: Darshit Sarda  
**Email**: ds8286@nyu.edu  
**LinkedIn**: [linkedin.com/in/darshitsarda](https://linkedin.com/in/darshitsarda)  
**GitHub**: [github.com/DarshitSarda](https://github.com/DarshitSarda)

---

## Disclaimer

**This system is for educational and research purposes only.** Options trading involves substantial risk of loss. Past signal accuracy does not guarantee future performance. The author is not a SEBI-registered advisor and assumes no liability for trading losses. Always paper-trade extensively and consult a financial advisor before deploying real capital.

---

## Acknowledgments

- **NSE India**: For providing F&O data via undocumented API
- **Options Trading Community**: For validating PCR and Max Pain theories through shared research
- **NYU Tandon MFE Program**: For derivatives pricing and risk management education

---

**⭐ If you find this project useful, please star the repository!**

**📊 For questions or collaboration, open an issue on GitHub or reach out via LinkedIn.**
