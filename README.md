# B2B-CRM-Sales-Pipeline-Analysis
End-to-end CRM sales pipeline analysis — data cleaning, EDA, insights, and Power BI dashboard for a B2B SaaS product team

# B2B CRM Sales Pipeline Analysis

## 📋 Project Overview

An end-to-end data analysis of a B2B CRM sales system to uncover actionable business insights for a SaaS product team. The project involved cleaning, merging, and analyzing 5 relational CRM tables (8,800+ deal records) to evaluate pipeline health, sales team performance, product effectiveness, and data quality — culminating in strategic recommendations and a Power BI dashboard.

**Tools Used:** Python, Pandas, Matplotlib, Seaborn, Power BI  
**Dataset:** 5 CRM tables with 8,800+ sales pipeline records across 85 accounts, 7 products, and 35 sales agents

---

## 📁 Repository Structure

```
├── README.md                          # Project documentation
├── CRM_Sales_Analysis.ipynb           # Jupyter Notebook with full analysis
├── cleaned_master_dataset.csv         # Final merged and cleaned dataset
├── charts/                            # Exported visualizations
│   ├── pipeline_stages.png
│   ├── sector_revenue.png
│   ├── manager_performance.png
│   └── product_revenue.png
└── dashboard/                         # Power BI dashboard screenshots
    └── dashboard_screenshots.png
```

---

## 📊 Data Description

The dataset consists of 5 interconnected CRM tables from a B2B sales system:

### 1. Sales Pipeline (8,800 rows × 8 columns)
The core transactional table — every row represents one deal/opportunity.

| Column | Type | Description |
|--------|------|-------------|
| `opportunity_id` | String | Unique identifier for each deal |
| `sales_agent` | String | Salesperson working on the deal |
| `product` | String | Product being sold (7 products across 3 lines) |
| `account` | String | Client company name (**16.2% missing**) |
| `deal_stage` | String | Current status: Prospecting → Engaging → Won / Lost |
| `engage_date` | Date | When the salesperson first engaged with the client |
| `close_date` | Date | When deal was closed (NULL for open deals) |
| `close_value` | Float | Actual closing price (NULL for open deals) |

### 2. Accounts (85 rows × 7 columns)
Client company information.

| Column | Type | Description |
|--------|------|-------------|
| `account` | String | Company name (join key) |
| `sector` | String | Industry — 10 sectors (retail, technology, medical, etc.) |
| `year_established` | Integer | Year the company was founded |
| `revenue` | Float | Annual revenue of the client (in millions) |
| `employees` | Integer | Number of employees |
| `office_location` | String | Country where the client is located |
| `subsidiary_of` | String | Parent company (70 out of 85 are independent) |

### 3. Products (7 rows × 3 columns)
Product catalog with pricing.

| Column | Type | Description |
|--------|------|-------------|
| `product` | String | Product name (join key) |
| `series` | String | Product line: GTX, MG, or GTK |
| `sales_price` | Integer | List/catalog price (ranges from 55 to 26,768) |

### 4. Sales Team (35 rows × 3 columns)
Sales organization structure.

| Column | Type | Description |
|--------|------|-------------|
| `sales_agent` | String | Agent name (join key) |
| `manager` | String | Manager name (6 managers total) |
| `regional_office` | String | Region: Central, East, or West |

### 5. Data Dictionary (21 rows × 3 columns)
Metadata describing all fields across the other 4 tables.

---

## 🧹 Data Cleaning Steps

### Step 1: Individual Table Cleaning
- **Accounts:** Fixed typo `technolgy` → `technology` in the sector column. Filled 70 missing `subsidiary_of` values with `Independent`.
- **Products:** Stripped whitespace from all text columns. No missing values found.
- **Sales Team:** Stripped whitespace. No missing values or duplicates found.
- **Sales Pipeline:** Converted `engage_date` and `close_date` to datetime format. Stripped whitespace from all categorical columns.

### Step 2: Critical Data Quality Fix — Product Name Mismatch
Discovered that `GTXPro` in the Sales Pipeline table did not match `GTX Pro` (with a space) in the Products table. This single-character difference caused **1,480 records** to lose product series and pricing data after the merge. Standardized `GTXPro` → `GTX Pro` before merging.

### Step 3: Merging into Master Table
Built the master dataset using the Sales Pipeline as the center table with sequential LEFT JOINs:

```
Sales Pipeline (8,800 rows)
    ├── LEFT JOIN Sales Team    ON sales_agent  → Added: manager, regional_office
    ├── LEFT JOIN Accounts      ON account      → Added: sector, revenue, employees, etc.
    └── LEFT JOIN Products      ON product      → Added: series, sales_price
```

**Why LEFT JOIN?** To retain all 8,800 deals even when account information is missing (16.2% of deals have no linked account).

### Step 4: Feature Engineering
Created two calculated columns:
- **`deal_duration`** = `close_date` - `engage_date` (in days) — measures how long a deal takes to close
- **`discount_pct`** = (`sales_price` - `close_value`) / `sales_price` × 100 — measures discount given vs. list price

**Final Master Dataset:** 8,800 rows × 20 columns

---

## 🔍 Key Insights

### Insight 1: Pipeline Health
- **Win Rate: 63.2%** (4,238 won / 6,711 closed deals) — healthy pipeline
- 2,089 deals remain open (1,589 Engaging + 500 Prospecting) — 24% of all deals are stalled

### Insight 2: Critical Data Quality Gap
- **16.2% of deals (1,425) have NO account linked.** These deals cannot be analyzed by sector, company size, or location. This is a CRM data entry gap that blocks all future AI/ML initiatives.
- **Root cause:** Sales reps not selecting an account when creating deals.
- **Fix:** Make the account field mandatory in the CRM before saving any deal.

### Insight 3: Product Name Mismatch (Resolved)
- `GTXPro` vs `GTX Pro` — a single space character caused 1,480 records to break during analysis. This demonstrates why data standardization (dropdowns, not free text) is essential in CRM systems.

### Insight 4: GTK 500 — High Value, Low Volume Opportunity
- Average deal size: **₹26,708** (highest by far) but only **15 deals won**
- If conversion is improved, even 50 more GTK 500 deals/year = **₹1.3M additional revenue**
- Investigation needed: Why is volume so low? Is it pricing, awareness, or market fit?

### Insight 5: Sales Team Disparity
- **Cara Losch's team:** Highest win rate (64.4%) with fewest stalled deals (219)
- **Melvin Marxen's team:** Most revenue (₹2,251,930) but most stalled deals (511)
- **Darcel Schlecht (agent):** Generates ₹1,153,214 revenue — more than 2× the next agent. Single-point-of-failure risk.

### Insight 6: Sector Performance
- **Retail** leads by total revenue (₹1,867,528) with 799 won deals
- **Entertainment** has highest win rate (64.7%) with strong avg deal size (₹2,650) despite fewer deals
- **Finance** has lowest win rate (61.2%) — may need specialized sales support or adjusted pricing

### Insight 7: Regional Balance
- **West region** leads revenue (₹3,568,647) with best win rate (63.9%)
- All three regions (Central, East, West) perform within a narrow band — no major regional concern

---

## 📈 3 Key KPIs

| KPI | Value | Why It Matters |
|-----|-------|----------------|
| **Win Rate** | 63.2% | Primary measure of sales effectiveness. Track weekly to detect trends. |
| **Avg Deal Cycle** | ~133 days | Engage-to-close duration. Monitor for early warning of deals going stale. |
| **Avg Deal Size** | Varies by product (₹55 – ₹26,708) | Track by product and region to spot pricing and market trends. |

---

## 💡 Recommendations

### Immediate Actions
1. **Fix Data Entry:** Require the account field on all new deals in CRM. 16.2% of pipeline being unanalyzable is a showstopper for any AI/ML initiative.
2. **Address Stalled Deals:** 2,089 open deals need attention. Implement automated alerts when deals stay in Engaging stage for >30 days. Prioritize Melvin Marxen's 511 stalled deals first.
3. **Explore GTK 500 Opportunity:** At ₹26,708 per deal, even a small volume increase creates massive revenue impact. Investigate why volume is low.

### Future AI/ML Roadmap
**Top 5 Fields for a Churn/Account-Health Model:**
1. `deal_duration` — velocity signals engagement health
2. `close_value` vs `sales_price` — discount patterns predict churn risk
3. `sector` — industry-specific churn patterns
4. `deal_stage` progression speed — stalled deals = at-risk accounts
5. `manager` / `regional_office` — team quality affects retention

**Additional Data Sciqus Should Start Collecting:**
1. Product login/usage frequency (are customers actually using the platform?)
2. Support ticket count per account (high tickets = churn risk)
3. NPS / satisfaction survey scores
4. Contract renewal dates (predict churn 90 days before expiry)
5. Number of active users per account (low usage = churn risk)

### Daily Product Manager Dashboard
- **Page 1:** Pipeline funnel (deal count + value by stage) with slicers for manager/region/sector
- **Page 2:** Deals expected to close this week/month
- **Page 3:** Stalled deals alert (>30 days in Engaging/Prospecting)
- **Page 4:** Win rate trend (weekly rolling average)
- **Page 5:** Top accounts by revenue and risk score

---

## 🛠️ How to Run

1. Clone this repository
2. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn
   ```
3. Open `CRM_Sales_Analysis.ipynb` in Jupyter Notebook or Google Colab
4. Update the file path in the first cell to point to your data files
5. Run all cells sequentially

---

## 👤 Author

**Garima Yadav**  
MSc Data Science | Symbiosis Skills and Professional University, Pune  
📧 garimayadav2929@gmail.com  
🔗 [LinkedIn](https://linkedin.com/in/garimayadav2911) | [GitHub](https://github.com/Garimayadav2911)

---

## 📄 License

This project is for educational and portfolio purposes. Dataset used was provided as part of a technical assessment.
