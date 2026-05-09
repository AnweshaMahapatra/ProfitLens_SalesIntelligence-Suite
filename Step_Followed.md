# Steps Followed
## Building the Sales Insight Power BI Dashboard End to End

---

## Step 1: SQL Data Analysis & Initial Exploration

Before loading anything into Power BI, SQL was used to understand the raw data's structure, volume, and key relationships. This is a critical step that most analysts skip, and it directly shaped what transformations were needed later.

**Reading the data**

```sql
SELECT * FROM customers;
SELECT * FROM products;
```

These baseline queries confirmed the structure of the dimension tables: customer names, codes, types (Brick & Mortar vs E-Commerce), and product identifiers before any joins were attempted.

**Counting records**

```sql
SELECT COUNT(*) FROM transactions;
```

Established the total number of transaction records in the dataset. This acts as a control figure after cleaning in Power Query; the final row count was compared against this to confirm no unintended data loss.

**Filtering transactions by market**

```sql
SELECT DISTINCT product_code 
FROM transactions 
WHERE market_code = 'Mark001';
```

Identified which products were active in a specific market (Mark001 = Delhi NCR). 
This query revealed the product mix per region and flagged that a significant volume of transactions had blank/null product codes. This data quality issue was later tracked and surfaced on the dashboard as the "(Blank)" product category.

**Joining transactions with the date dimension for year-level filtering**

```sql
SELECT transactions.*, date.*
FROM transactions
INNER JOIN date ON transactions.order_date = date.date
WHERE date.year = 2020;
```

This join was essential for building the time-intelligence foundation. 
The date dimension table enables year and month-level slicing in Power BI; without it, 
the year/month filter buttons on every dashboard page would not function.

**Total revenue for a specific year**

```sql
SELECT SUM(transactions.sales_amount)
FROM transactions
INNER JOIN date ON transactions.order_date = date.date
WHERE date.year = 2020;
```

Pulled the 2020 revenue total (₹142M) directly from the source, which later validated the headline KPI card on the Performance Analysis page.

**Revenue for a specific month and market**

```sql
SELECT SUM(transactions.sales_amount)
FROM transactions
INNER JOIN date ON transactions.order_date = date.date
WHERE date.year = 2020
  AND date.month_name = 'January'
  AND transactions.market_code = 'Mark001';
```

This granular query filtering by year, month, and market simultaneously confirmed that the data model could support
multi-dimensional filtering before replicating the same logic in Power BI.
It also validated the Delhi NCR January 2020 revenue figure used in the Performance Analysis trend.

---

## Step 2: Loading Data into Power BI Desktop

Once the SQL exploration was complete and the data structure was validated, all tables were loaded into Power BI Desktop directly from the SQL database using the native SQL Server connector.

Tables loaded:
- `sales transactions` — fact table (transaction-level records)
- `sales customers` — customer dimension
- `sales products` — product dimension
- `sales markets` — market/geography dimension
- `sales date` — date dimension (enables time intelligence in DAX)

---

## Step 3: Data Cleaning & Transformation in Power Query

After loading, Power Query was used to clean and standardise the data before any modeling or visualisation work began.

**Removed irrelevant and empty records**
Filtered out rows with null or zero sales amounts, test records, and any entries that would distort KPI calculations. This included removing negative-value returns from headline revenue metrics while retaining them for accurate profit margin calculations.

**Filtered to relevant markets**
The dataset contained transactions coded to markets outside the intended scope. Filters were applied to retain only the 14 active Indian markets visible in the final dashboard.

**Standardised currency values**
The `sales_amount` field contained values in mixed formats.
Some records store amounts in USD (from international transactions) and others in INR.
A conditional column (`new_sales_amount`) was created to convert all values to INR, ensuring every financial calculation in the dashboard operates on a consistent currency basis. 
This is why the Revenue measure references `new_sales_amount` rather than the raw `sales_amount` field.

**Validated data types**
Confirmed that date fields were recognised as Date type (not text), numeric fields were set to Decimal or Whole Number, and text fields like market names and customer names were trimmed of leading/trailing spaces.

---

## Step 4: DAX Measures

All KPIs visible on the dashboard are driven by DAX measures, not raw column values. This ensures every number responds correctly to slicer selections (year, month, market filters).

```dax
Revenue = SUM('sales transactions'[new_sales_amount])
```
Core revenue measure. Uses the cleaned `new_sales_amount` column (post currency standardisation) rather than the raw field.

---

```dax
Sales Qty = SUM('sales transactions'[sales_qty])
```
Total units sold. Feeds the Sales Qty KPI card on all three pages.

---

```dax
Total Profit Margin = SUM('sales transactions'[profit_margin])
```
Sums the pre-calculated profit margin field from the transactions table. This field represents revenue minus cost at the transaction level.

---

```dax
Profit Margin % = DIVIDE([Total Profit Margin], [Revenue], 0)
```
Calculates margin as a percentage of revenue. The `0` in `DIVIDE()` returns zero instead of an error when revenue is 
null critical for markets like Bengaluru, where the result is negative, but must still render.

---

```dax
Revenue Contribution % = 
DIVIDE(
    [Revenue],
    CALCULATE([Revenue], ALL('sales products'), ALL('sales customers'), ALL('sales markets'))
)
```
Calculates each market's or customer's share of total revenue. 
`ALL()` removes the current filter context from the three-dimensional tables, so the denominator always reflects the full dataset total, regardless of which market or customer is selected in a slicer.

---

```dax
Profit Margin Contribution % = 
DIVIDE(
    [Total Profit Margin],
    CALCULATE([Total Profit Margin], ALL('sales products'), ALL('sales customers'), ALL('sales markets'))
)
```
Same logic as Revenue Contribution %, but applied to profit margin. 
This is what powers the "Profit Margin Contribution % by markets_name" column, the measure that revealed Delhi NCR contributes 48.5% of profit despite running at only 2.3% margin.

---

```dax
Revenue LY = CALCULATE([Revenue], SAMEPERIODLASTYEAR('sales date'[date]))
```
Prior year revenue for the same period. Uses `SAMEPERIODLASTYEAR()` 
from the date dimension to shift the filter context back exactly one year. 
This drives the "Revenue LY" overlay line in the Performance Analysis trend chart,
making year-on-year deterioration visible month by month in 2020.

---

```dax
Target Diff = [Profit Margin %] - 'Profit Target'[Profit Target Value]
```
Calculates the gap between actual profit margin % and the user-defined target (default: 2%). 
Negative values indicate markets or customers below the target. 
This measure connects to the What If parameter on the Performance Analysis page, 
making the profit target threshold dynamic and adjustable.

---

## Step 5: Report Pages Built

**Page 1 — Key Insights**
A high-level operational overview. Displays total revenue (₹985M), sales quantity (2M), 
revenue and quantity rankings by market, revenue trend from 2018–2020, customer type split (Brick & Mortar vs E-Commerce), 
top customers by revenue, and top product codes. Designed for quick scanning by any stakeholder.

**Page 2 — Profit Analysis**
The financial diagnostic layer. Shows three parallel market rankings.
Revenue Contribution %, Profit Margin Contribution %, and Profit Margin % side by side so the inversion between revenue size and margin efficiency is immediately visible. 
Includes the full customer profitability table with all four metrics per account. 
Total Profit Margin KPI card (₹24.7M) added as a third headline figure alongside Revenue and Sales Qty.

**Page 3 — Performance Analysis**
A time-filtered diagnostic for 2020. Includes a dynamic Profit Target parameter (default 2%), 
a combined trend chart overlaying current revenue, prior year revenue, 
and profit margin %, and a 2020-specific customer profitability table. 
Designed to answer: which markets and customers are above or below the minimum viability threshold right now?

---

## Insights Gained

1. Learned how to validate data at the SQL layer before loading into Power BI, catching currency inconsistencies and null product codes early prevented downstream errors in every measure.

2. Understood how `ALL()` in DAX unlocks contribution % calculations by intentionally ignoring filter context
Saw firsthand how `SAMEPERIODLASTYEAR()` requires a properly structured date dimension table; without it, the Revenue LY measure returns blank

3. Recognised that the most impactful dashboard insight (Bengaluru's –20.8% margin) only became visible because three separate    measures were placed side by side; no single measure tells that story alone.
   
4. Appreciated that data cleaning decisions (like creating `new_sales_amount` for currency standardisation) are not cosmetic.
 They directly determine whether the ₹985M headline figure is accurate or misleading
