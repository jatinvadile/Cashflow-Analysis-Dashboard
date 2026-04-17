# 💰 Cashflow Analysis Dashboard

🚀 End-to-End Automated Cashflow Reporting with Real-Time Exchange Rate Integration using Power BI, Python & API  

---

## 📌 Overview  
This project delivers an automated cashflow analytics solution by integrating real-time exchange rate data (USD ↔ TZS) with financial reporting dashboards.

It eliminates manual processes, maintains historical exchange rate data, and enables accurate multi-currency financial analysis in Power BI.

---

## 🎯 Objectives  
- Automate manual cashflow tracking process  
- Integrate real-time exchange rates using API  
- Maintain historical exchange rate dataset  
- Enable currency-wise financial analysis  
- Improve reporting accuracy and efficiency  

---

## 🏗️ Architecture  
```
Exchange Rate API
│
▼
Python Automation Script
│
▼
Excel (Historical Storage)
│
▼
Power Query (Power BI)
│
▼
Data Model
│
▼
Dashboard & KPIs
```

---

## ⚙️ Tech Stack  

- Power BI  
- Power Query (M Language)  
- Python (Pandas, Requests)  
- Excel  
- Exchange Rate API  

---

## 🔄 Data Pipeline  

1. Fetch exchange rate data from API  
2. Transform and validate using Python  
3. Store historical data in Excel  
4. Load into Power BI using Power Query  
5. Build dashboard and KPIs  

---

## 🐍 Automation Script Features  

- Fetches real-time USD → TZS exchange rate from API  
- Calculates reverse rate (TZS → USD)  
- Stores daily records with timestamp  
- Maintains historical dataset  
- Prevents duplicate entries for same date  
- Automatically updates Excel data source  

---

## 📊 Dashboard KPIs  

- Total Inflow  
- Total Outflow  
- Closing Balance  
- Net Cashflow  
- Currency-wise Balance  

---

## 📈 Key Features  

- Automated data pipeline  
- Multi-currency support (USD, TZS)  
- Historical exchange rate tracking  
- Dynamic filtering (Date, Currency, Account)  
- Monthly closing balance calculation  
- Interactive Power BI dashboard  

---

## 🔍 Insights Delivered  

- Identified cashflow trends across time  
- Highlighted negative cashflow periods  
- Improved visibility into account-level balances  
- Enabled better liquidity and financial planning  

---

## 🚀 Business Impact  

- Reduced manual effort by **~40–50%**  
- Improved data accuracy and consistency  
- Enabled near real-time financial insights  
- Centralized financial reporting system  

---

## 📂 Repository Structure  
```

cashflow-analytics/
│
├── data/
│ ├── raw/
│ ├── processed/
│
├── scripts/
│ ├── exchange_rate.py
│
├── powerbi/
│ ├── Cashflow_Dashboard.pbix
│
├── docs/
│ ├── dashboard.png
│
├── requirements.txt
└── README.md
```

---

## 🐍 Sample Script  

```python
import requests
import pandas as pd
from datetime import datetime

API_URL = "https://open.er-api.com/v6/latest/USD"

response = requests.get(API_URL).json()
rate = response["rates"]["TZS"]

today = datetime.today().date()

df = pd.DataFrame({
    "Date": [today],
    "USD_TO_TZS": [rate]
})

df.to_excel("exchange_rates.xlsx", index=False)

```

🔮 Future Enhancements
Direct integration with SharePoint / Database
Real-time API-based dashboard refresh
Cashflow forecasting models
Alert system for low balances
ERP integration
📷 Dashboard Preview

Add Power BI screenshots here

👨‍💻 Author

Jatin Vadile
