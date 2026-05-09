# SQL & DAX Reference
## Every Query and Measure Behind the Sales Insight Dashboard

*This file documents all SQL queries used for data exploration and all DAX measures used in the Power BI model, with the business reasoning behind each one.*

---

# Part 1: SQL Queries

---

### 1. Reading the Dimension Tables

```sql
SELECT * FROM customers;
SELECT * FROM products;
```

**Why:** 

First step before any analysis, confirm the structure of the customer and product dimension tables. 

This revealed the `customer_type` field (Brick & Mortar vs E-Commerce) that later drives the channel split donut chart. 
and flagged that product codes follow a `ProdXXX` naming convention with a significant volume of nulls.

---

### 2. Counting Total Transaction Records

```sql
SELECT COUNT(*) FROM transactions;
```

**Why:** 

Establishes a control figure. After all cleaning and transformation in Power Query, the final row count in Power BI was compared against this number to confirm nothing was accidentally dropped during the ETL process.

---

### 3. Distinct Products Active in a Specific Market

```sql
SELECT DISTINCT product_code 
FROM transactions 
WHERE market_code = 'Mark001';
```

**Why:** 

Mark001 is Delhi NCR — the largest market by revenue. 

This query identified which products were driving Delhi NCR's ₹520M contribution and first surfaced the blank product code issue.

A large number of transactions had null `product_code` values, which later became the "(Blank)" category showing ₹0.47bn on the dashboard — a data quality flag requiring resolution at the source system.

---

### 4. Full Transaction Detail Joined with Date Dimension — Year Filter

```sql
SELECT transactions.*, date.*
FROM transactions
INNER JOIN date ON transactions.order_date = date.date
WHERE date.year = 2020;
```

**Why:** 

This join is the structural foundation for all time-intelligence in the model. 

By joining the fact table to the date dimension on `order_date = date`, year and month filters become possible. 

The 2020 filter here validated the record set that feeds the entire Performance Analysis page (₹142M revenue, 350K units, ₹2.1M profit margin).

---

### 5. Total Revenue for a Specific Year

```sql
SELECT SUM(transactions.sales_amount)
FROM transactions
INNER JOIN date ON transactions.order_date = date.date
WHERE date.year = 2020;
```

**Why:** 

Validated the 2020 revenue total (₹142M) before the Power BI Revenue measure was built. 

Running this in SQL first ensured the DAX result would match if they disagreed; it pointed to either a currency standardisation gap or a filtering issue in Power Query.

---

### 6. Revenue for a Specific Year, Month, and Market

```sql
SELECT SUM(transactions.sales_amount)
FROM transactions
INNER JOIN date ON transactions.order_date = date.date
WHERE date.year = 2020
  AND date.month_name = 'January'
  AND transactions.market_code = 'Mark001';
```

**Why:** 

The most granular validation query — filtering by year, month, and market simultaneously. 

This confirmed that the data model could support multi-dimensional slicing (the year/month/market filter interaction visible on every dashboard page) and validated the Delhi NCR January 2020 figure used in the Performance Analysis trend.

---

---

# Part 2: DAX Measures

---

### Revenue

```dax
Revenue = SUM('sales transactions'[new_sales_amount])
```

**Why 

##`new_sales_amount` and not `sales_amount`:** The raw `sales_amount` field contained values in mixed currencies;

Some transactions were recorded in USD. During Power Query cleaning, a conditional column `new_sales_amount` was created to convert all values to INR. 

Every revenue calculation in the model references this cleaned column. Using the raw field would produce an overstated and inaccurate ₹985M figure.

---

### Sales Qty

```dax
Sales Qty = SUM('sales transactions'[sales_qty])
```

Total units sold across all transactions. Feeds the 2M Sales Qty KPI card on Key Insights and Profit Analysis pages, and the 350K card on the Performance Analysis page when filtered to 2020.

---

### Total Profit Margin

```dax
Total Profit Margin = SUM('sales transactions'[profit_margin])
```

Sums the pre-calculated `profit_margin` field from the transactions table, which represents revenue minus cost at the transaction level. The negative values in this field for certain markets and customers are what produce the loss-making margin percentages (Bengaluru: –20.8%) when divided by revenue.

---

### Profit Margin %

```dax
Profit Margin % = DIVIDE([Total Profit Margin], [Revenue], 0)
```

Profit as a percentage of revenue. The third argument `0` in `DIVIDE()` returns zero instead of an error when revenue is null or zero — essential for robustness when market or customer filters return no revenue records. This measure feeds all three "Profit Margin %" ranked columns across the dashboard.

---

### Revenue Contribution %

```dax
Revenue Contribution % = 
DIVIDE(
    [Revenue],
    CALCULATE(
        [Revenue],
        ALL('sales products'),
        ALL('sales customers'),
        ALL('sales markets')
    )
)
```

Each market's or customer's share of total revenue. The `CALCULATE(..., ALL(...))` pattern removes the current filter context from the three-dimensional tables so the denominator always equals the full dataset total, regardless of which market or customer row is being evaluated. Without `ALL()`, the denominator would equal the numerator in every row, and every contribution % would show 100%.

This is what produces the "52.76%" for Delhi NCR, its revenue divided by the total ₹985M with all other filters removed.

---

### Profit Margin Contribution %

```dax
Profit Margin Contribution % = 
DIVIDE(
    [Total Profit Margin],
    CALCULATE(
        [Total Profit Margin],
        ALL('sales products'),
        ALL('sales customers'),
        ALL('sales markets')
    )
)
```

Each market's or customer's share of total profit. Same `ALL()` pattern as Revenue Contribution %, but applied to profit margin. This is the measure that makes the profitability inversion visible — Delhi NCR contributes 48.5% of profit (less than its 52.76% revenue share), while smaller high-margin markets like Surat punch above their weight. Bengaluru and Kanpur show negative contribution values, confirming they drain the total profit pool.

---

### Revenue LY (Last Year)

```dax
Revenue LY = CALCULATE([Revenue], SAMEPERIODLASTYEAR('sales date'[date]))
```

Prior year revenue for the equivalent time period. `SAMEPERIODLASTYEAR()` shifts the date filter context back exactly one year using the `sales date` dimension table. This measure requires a properly structured date dimension and a continuous date table with no gaps to function correctly.

On the Performance Analysis page, this drives the "Revenue LY" overlay line in the trend chart. When filtered to Jan–Jun 2020, it automatically shows Jan–Jun 2019 revenue for comparison, making the year-on-year deterioration visible month by month without any manual period selection.

---

### Target Diff

```dax
Target Diff = [Profit Margin %] - 'Profit Target'[Profit Target Value]
```

The gap between the actual profit margin % and the user-defined profit target. Negative values mean the market or customer is below target. This measure is connected to the What If parameter on the Performance Analysis page, the "Profit Target" slicer (default: 2%) feeds `[Profit Target Value]`, making the threshold dynamic. A manager can set the target to 3% and immediately see which markets fall short.

---

## Measure Dependency Map

```
Revenue
  └── Revenue Contribution %
  └── Profit Margin %
        └── Target Diff

Total Profit Margin
  └── Profit Margin %
  └── Profit Margin Contribution %

Revenue LY
  └── (Performance Analysis trend chart — Revenue LY line)
```

All measures are evaluated in the filter context set by the year/month slicers and any visual-level filters. This is why the same measure (e.g., `Revenue`) returns ₹985M on the Key Insights page with no filters, ₹142M on the Performance Analysis page filtered to 2020, and ₹520M when a market bar for Delhi NCR is selected.

---

*Power BI model: Sales Insight | Data period: June 2017 – June 2020*
