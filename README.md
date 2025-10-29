# 📊 NSE Option Analytics Automation — Max Pain & PCR Framework

### 📌 Overview  
This repository unifies **Option Pain Theory** and **Put-Call Ratio (PCR) Analysis** into a single automation and analytics framework for the **Indian derivatives market**.  
It enables users to:  
- 🔹 Track **option sentiment dynamics** across expiries  
- 🔹 Identify **price gravitation levels** via **Max Pain**  
- 🔹 Gauge **market bias** (bullish/bearish) using **PCR trends**  
- 🔹 Conduct **expiry-wise comparative analysis** for equities and indices  

Built with a focus on **scalability**, **data accuracy**, and **resilience**, this project serves as a strong foundation for **quantitative option research**, **derivatives analytics**, and **strategy backtesting**.

---

### ✨ Key Features  
- ⚡ **Automated Data Fetching:** Pulls live NSE Option Chain data (JSON → CSV) for equities and indices  
- 📈 **Max Pain Detection:** Computes strike with minimum total loss for option writers  
- 📊 **PCR Analysis:** Calculates Put-Call Ratios at symbol and expiry levels  
- 🧠 **Expiry-Wise Insights:** Generates consolidated CSV summaries for faster comparison  
- 🧩 **Scalable Architecture:** Supports hundreds of equities with multi-threaded data fetch  
- 🛠️ **Resilience:** Handles missing, malformed, or empty data files gracefully  
- 📉 **Optional Visualization:** Enable charts to visualize Max Pain and PCR distribution  

---

### 🏗️ Workflow  
1. **Data Fetching** → Fetches live NSE Option Chain data for equities and indices  
2. **Data Cleaning & Structuring** → Normalizes columns like Strike Price, Call OI, Put OI  
3. **Max Pain Computation** → Finds strike with minimum total option writer loss  
4. **PCR Computation** → Calculates PCR for each symbol and expiry  
5. **Consolidation** → Aggregates expiry-wise summaries and index-wise reports  
6. **Visualization (Optional)** → Enables charts with `ENABLE_PLOTS = True`

---

### 📂 Output Structure
```bash
monthly_options_view/
├── Equity_Expiry_MaxPain/                 # ⚡ Expiry-wise Max Pain summaries
│   ├── Equity_MaxPain_2025-08-28.csv
│   ├── Equity_MaxPain_2025-09-04.csv
│   └── Equity_MaxPain_2025-09-11.csv
│
├── PCR_2025-08-28.csv                     # 📈 Expiry-wise Put-Call Ratios
├── PCR_2025-09-04.csv
├── PCR_2025-09-11.csv
│
├── NIFTY/                                 # 🧮 NIFTY option chain data
│   ├── NIFTY_2025-08-28.csv
│   ├── NIFTY_2025-09-04.csv
│   ├── NIFTY_2025-09-11.csv
│   ├── NIFTY_MaxPain_Summary.csv
│   └── PCR_NIFTY.csv
│
└── BANKNIFTY/                             # 🧮 BANKNIFTY option chain data
    ├── BANKNIFTY_2025-08-28.csv
    ├── BANKNIFTY_2025-09-04.csv
    ├── BANKNIFTY_2025-09-11.csv
    ├── BANKNIFTY_MaxPain_Summary.csv
    └── PCR_BANKNIFTY.csv
```

---

## ⚙️ Installation

### 1. Clone the Repository
```bash
git clone https://github.com/<your-username>/OptionAnalyticsAutomation.git
cd OptionAnalyticsAutomation
```

### 2. Create a Virtual Environment (recommended)
```bash
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

---

## 🚀 Usage

### 1. Run the main script
```bash
python main.py
```

### 2. Enable plots (optional)
Edit the configuration in the script:
```python
ENABLE_PLOTS = True
```

### 3. View Outputs
All CSVs and summaries will be saved in the `monthly_options_view/` folder.  
Use Excel, Pandas, or visualization libraries (Matplotlib, Plotly) for further analysis.

---

## 🧩 Dependencies
Core libraries used:
```
requests
pandas
numpy
concurrent.futures
matplotlib (optional)
```

Ensure you have **Python 3.8+** installed.

---

## 🧠 Applications
- Quantitative Options Research
- Market Sentiment & Expiry Bias Tracking
- Strategy Backtesting (Max Pain & PCR-based filters)
- Volatility & Derivative Data Visualization

---

## 🤝 Contributing
Contributions are welcome! If you'd like to improve functionality, optimize performance, or add new features:
1. Fork the repository
2. Create a new branch (`feature/new-feature`)
3. Commit and push your changes
4. Open a Pull Request

---

## 📬 Contact
**Author:** Darshit Sarda  
**Email:** darshitsarda10@gmail.com   
**LinkedIn:** [linkedin.com/in/darshitsarda](https://www.linkedin.com/in/darshitsarda)  
**GitHub:** [github.com/DarshitSarda](https://github.com/DarshitSarda)
