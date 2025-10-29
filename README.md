📈 Options Market Analyzer – Max Pain & Put-Call Ratio Framework
📊 Overview

This Python-based framework automates the fetching, processing, and analysis of NSE Option Chain (OC) data for equities and indices (NIFTY & BANKNIFTY).
It integrates two key analytical modules:

🔹 Max Pain Analysis: Identifies the strike price where option writers experience the least combined loss.

🔹 Put-Call Ratio (PCR) Analysis: Measures market sentiment across strikes, expiries, and instruments.

The project provides traders and analysts with structured data, expiry-wise summaries, and visual insights for informed decision-making.

⚙️ Key Features

✅ Automated fetching of NSE option chain data
✅ Handles 200+ equities and major indices concurrently
✅ Max Pain calculation for equities and indices
✅ Put-Call Ratio computation at symbol and expiry levels
✅ Expiry-wise summary generation for streamlined analysis
✅ Built-in retries, backoff, and JSON error handling
✅ Optional visualization for Max Pain and PCR trends

🧩 Workflow

Data Fetching & Session Management

Persistent HTTP session with correct headers and cookies

Automatic retries and backoff for failed requests

Option Chain Normalization

Cleans and structures data: Strike Price, Call OI, Put OI, Change in OI, etc.

Max Pain Computation

Calculates total loss for option writers at each strike

Identifies the strike with minimum total loss (Max Pain)

Exports expiry-wise summaries

Put-Call Ratio Computation

Computes PCR = Total Put OI / Total Call OI

Generates symbol-level and master PCR summaries

Output Consolidation

Organizes expiry-wise data for equities and indices

Visualization (Optional)

Enable ENABLE_PLOTS = True to generate Max Pain and PCR charts

📂 Output Structure
monthly_options_view/
├── Equity_MaxPain_Summary/
│   ├── Equity_MaxPain_2025-08-28.csv
│   ├── Equity_MaxPain_2025-09-04.csv
│   └── Equity_MaxPain_2025-09-11.csv
│
├── PCR_Summaries/
│   ├── PCR_2025-08-28.csv
│   ├── PCR_2025-09-04.csv
│   └── PCR_2025-09-11.csv
│
├── Equities/
│   ├── RELIANCE/
│   │   ├── RELIANCE_2025-08-28.csv
│   │   ├── RELIANCE_2025-09-04.csv
│   │   └── RELIANCE_2025-09-11.csv
│   └── ...
│
└── Indices/
    ├── NIFTY/
    │   ├── NIFTY_2025-08-28.csv
    │   ├── NIFTY_2025-09-04.csv
    │   ├── NIFTY_2025-09-11.csv
    │   └── PCR_NIFTY.csv
    │
    └── BANKNIFTY/
        ├── BANKNIFTY_2025-08-28.csv
        ├── BANKNIFTY_2025-09-04.csv
        ├── BANKNIFTY_2025-09-11.csv
        └── PCR_BANKNIFTY.csv

🚀 Installation

1. Clone the Repository

git clone https://github.com/<your-username>/Options_Market_Analyzer.git
cd Options_Market_Analyzer


2. Create a Virtual Environment (Recommended)

python -m venv venv
source venv/bin/activate      # macOS/Linux
venv\Scripts\activate         # Windows


3. Install Dependencies

pip install -r requirements.txt

🧠 Usage

Run the main script (replace with your entry script name if different):

python main.py


After execution, you’ll get:

Expiry-wise Max Pain summaries in Equity_MaxPain_Summary/

PCR summaries in PCR_Summaries/

Individual equity and index CSVs in their respective folders

Optional visualization:

python main.py --plot

📦 Requirements

Core dependencies:

pandas  
numpy  
requests  
matplotlib  
tqdm  
concurrent.futures

🧠 About

This project integrates Option Pain Theory and Put-Call Ratio (PCR) Analysis into a unified framework for the Indian derivatives market, enabling users to track option sentiment, identify Max Pain levels, and assess market bias with a scalable, data-driven approach.

📬 Contact

For questions or collaboration opportunities:
📧 darshitsarda10@gmail.com
