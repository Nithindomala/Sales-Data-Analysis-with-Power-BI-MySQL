# 💰 Sales Data Analysis — Power BI & MySQL

> **Tools:** Power BI (DAX, Power Query) · MySQL · Excel
>
> **Dataset:** Multi-market sales transactions | India

### 🔗 Quick Links

- 📊 [Power BI Dashboard](https://github.com/Nithindomala/Sales-Data-Analysis-with-Power-BI-MySQL/blob/main/Sales_insights.pbix)
- 🗄️ [SQL Data File](https://github.com/Nithindomala/Sales-Data-Analysis-with-Power-BI-MySQL/blob/main/Data.sql)

---

## 📌 Business Problem

A multi-city Indian sales organisation had revenue data spread across markets but no consolidated view to answer the questions that management needed for strategic planning:

1. **Which markets generate the most revenue** — and which are the most profitable?
2. **Is there dangerous customer concentration risk** — is the business over-dependent on a single account?
3. **Why did revenue decline after February 2020** — was it seasonal, competitive, or structural?

Without a unified dashboard, leadership was reviewing fragmented market-level reports and making pricing and resource allocation decisions without a clear profit margin picture. This project delivers an end-to-end SQL-to-Power BI analytical platform to answer these questions.

---

## 🔍 Data Validation & Quality Assurance

| Validation Step | Method | Finding / Action |
|---|---|---|
| Check for duplicate transaction records | MySQL `GROUP BY` + `COUNT` audit | Duplicates identified and removed before import |
| Validate currency consistency | Filter on currency field | Mixed INR / USD records found — normalised to INR using exchange rate conversion |
| Remove zero and negative revenue records | `WHERE sales_amount > 0` filter | Invalid records excluded from KPI calculations |
| Verify market name standardisation | `SELECT DISTINCT market_name` audit | Inconsistent casing and spacing corrected via Power Query |
| Confirm date range completeness | Min/max date check across all markets | Confirmed continuous data from Jan 2018 to Jun 2020 |

> Currency normalisation was critical — without it, USD-denominated transactions would have significantly distorted revenue rankings across markets.

---

## ⚙️ Analytical Approach

### MySQL — Data Extraction & Validation

```sql
-- Validate: Check for transactions with invalid revenue values
SELECT COUNT(*) AS invalid_records
FROM transactions
WHERE sales_amount <= 0;

-- Extract: Revenue by market with transaction count
SELECT 
    m.markets_name                    AS Market,
    SUM(t.sales_amount)               AS Total_Revenue,
    COUNT(t.product_code)             AS Total_Transactions,
    ROUND(SUM(t.sales_amount) / 
          COUNT(t.product_code), 2)   AS Avg_Transaction_Value
FROM transactions t
JOIN markets m ON t.market_code = m.markets_code
WHERE t.sales_amount > 0
  AND t.currency = 'INR'
GROUP BY m.markets_name
ORDER BY Total_Revenue DESC;

-- Extract: Top customers by revenue contribution
SELECT 
    c.custmer_name                                      AS Customer,
    SUM(t.sales_amount)                                 AS Total_Revenue,
    ROUND(SUM(t.sales_amount) * 100.0 / 
          (SELECT SUM(sales_amount) FROM transactions 
           WHERE sales_amount > 0), 2)                  AS Revenue_Share_Pct
FROM transactions t
JOIN customers c ON t.customer_code = c.customer_code
WHERE t.sales_amount > 0
GROUP BY c.custmer_name
ORDER BY Total_Revenue DESC
LIMIT 10;
```

### Power BI — DAX Measures

```
Total Revenue =
    CALCULATE(
        SUM(transactions[sales_amount]),
        transactions[sales_amount] > 0
    )

Profit Margin % =
    DIVIDE(
        SUM(transactions[profit_margin]),
        SUM(transactions[sales_amount]),
        0
    )

Revenue Contribution % =
    DIVIDE(
        [Total Revenue],
        CALCULATE([Total Revenue], ALL(customers)),
        0
    )
```

---

## 📊 Key KPIs & Findings

| KPI | Value | Business Signal |
|---|---|---|
| Highest revenue market | **Delhi NCR — ₹78M** | Volume leader but not the profitability leader |
| Most profitable market | **Mumbai — 23.9% margin** | Higher margin despite lower revenue than Delhi NCR |
| Overall profit margin | **~1.4%** | ⚠️ Critically low — pricing or cost structure review needed |
| Top customer revenue share | **Electricalsara Stores ~50%** | 🔴 Severe concentration risk — single account dependency |
| Revenue decline start | **Post Feb 2020** | Likely COVID-19 disruption — requires YoY comparison with 2021 data |

---

## 📈 Strategic Recommendations

Three data-backed recommendations for sales leadership:

**1. Prioritise Mumbai's margin model — not Delhi NCR's volume model**

Delhi NCR generates the most revenue but Mumbai generates the best margins (23.9%). The business should study what drives Mumbai's superior profitability — product mix, pricing discipline, or cost structure — and replicate that model in other markets before chasing volume growth.

**2. Urgently reduce Electricalsara Stores dependency**

A single customer representing ~50% of total revenue is a critical business risk. One contract change, dispute, or competitor switch could halve revenue overnight. Immediate action: diversification targets for the sales team with incentives tied to new account revenue growth.

**3. Conduct a pricing review to address the 1.4% margin crisis**

A ~1.4% overall profit margin is unsustainable and suggests either underpricing, excessive discounting, or high operational costs eating into margins. Recommendation: SKU-level margin analysis to identify which products and markets are margin-positive vs. margin-negative, followed by a selective pricing adjustment.

---

## 🗂️ Repository Structure

```
Sales-Data-Analysis-with-Power-BI-MySQL/
├── README.md              ← Project documentation (this file)
├── Data.sql               ← MySQL database schema and raw data
└── Sales_insights.pbix    ← Power BI dashboard file
```

---

## 🛠️ Tech Stack

| Layer | Tool | Purpose |
|---|---|---|
| Data Storage & Extraction | MySQL | Schema design, data validation queries, KPI extraction |
| Data Transformation | Power Query (Power BI) | Currency normalisation, market name standardisation |
| DAX Measures & KPIs | Power BI Desktop | Revenue, profit margin, contribution % calculations |
| Visualisation | Power BI | Market comparison, customer segmentation, trend analysis |
| Documentation | This README | Stakeholder-readable project narrative |

---

*Project by [Nithin Domala](https://www.linkedin.com/in/nithin-domala) · [Portfolio](https://nithindomala.netlify.app/)*
