# SQL Queries Behind the Insights

## *How SQL Quietly Powered Every Business Decision Inside the Dashboard*

*Behind every KPI card, trend line, and profitability insight in the dashboard was a layer of SQL logic transforming raw transactions into business intelligence. These were not written as textbook exercises — they were built to answer real operational questions, uncover hidden risks, and support strategic decision-making.*

---

# 📌 Query 1: Total Revenue & Sales Quantity — Measuring the Scale of the Business

### 🎯 What this powers:

The headline KPI cards:

* **₹985M Revenue**
* **2M Sales Quantity**

These metrics establish the size of the business before deeper analysis begins.

```sql id="4l7vfx"
SELECT
    SUM(s.sales_amount) AS total_revenue,
    SUM(s.sales_qty)    AS total_sales_qty
FROM sales_transactions s
WHERE s.sales_amount > 0;
```

### 💡 Why this query matters

At first glance, the query looks simple.

But the filter:

```sql id="f5rqvj"
WHERE s.sales_amount > 0
```

was critical.

The raw transaction table contained:

* return orders,
* negative transactions,
* and test entries with zero value.

Without filtering those records:

* revenue would appear artificially lower,
* and the dashboard’s headline KPIs would lose credibility.

This query created the foundation for every downstream analysis.

---

# 📌 Query 2: Revenue by Market — Revealing Business Dependency

### 🎯 What this powers:

* Revenue by Markets chart
* Sales Quantity by Markets chart

```sql id="gn9z1y"
SELECT
    m.markets_name,
    SUM(s.sales_amount) AS revenue,
    SUM(s.sales_qty)    AS sales_qty,

    ROUND(
        SUM(s.sales_amount) * 100.0 /
        SUM(SUM(s.sales_amount)) OVER (),
        2
    ) AS revenue_contribution_pct

FROM sales_transactions s
JOIN markets m
ON s.market_code = m.markets_code

WHERE s.sales_amount > 0

GROUP BY m.markets_name
ORDER BY revenue DESC;
```

### 💡 What the business learned

This query exposed something leadership had never fully visualized before:

# ⚠️ Delhi NCR alone contributed more than 52% of total revenue.

The window function:

```sql id="h9h8ek"
SUM(...) OVER ()
```

allowed each market’s contribution percentage to be calculated dynamically against the overall revenue pool.

Without needing nested subqueries, the business could instantly compare:

* scale,
* contribution,
* and market dominance.

This was the query that revealed how dependent the organization had become on a single geography.

---

# 📌 Query 3: Profitability by Market — The Query That Changed the Story

### 🎯 What this powers:

The Profitability Analysis table by market.

```sql id="yv2b6s"
SELECT
    m.markets_name,

    SUM(s.sales_amount)  AS revenue,
    SUM(s.profit_margin) AS total_profit,

    ROUND(
        SUM(s.profit_margin) * 100.0 /
        SUM(s.sales_amount),
        2
    ) AS profit_margin_pct,

    ROUND(
        SUM(s.sales_amount) * 100.0 /
        SUM(SUM(s.sales_amount)) OVER (),
        2
    ) AS revenue_contribution_pct,

    ROUND(
        SUM(s.profit_margin) * 100.0 /
        SUM(SUM(s.profit_margin)) OVER (),
        2
    ) AS profit_margin_contribution_pct

FROM sales_transactions s
JOIN markets m
ON s.market_code = m.markets_code

WHERE s.sales_amount > 0

GROUP BY m.markets_name
ORDER BY profit_margin_pct DESC;
```

### 💡 Why this query was powerful

This was the moment the business narrative changed.

The dashboard stopped asking:

> “Which market sells the most?”

and started asking:

> “Which market actually makes money?”

That distinction uncovered the:

# 📉 Profitability Inversion

Smaller cities like:

* Surat
* Patna

rose to the top on efficiency,
while larger markets:

* Delhi NCR
* Bengaluru

collapsed in ranking.

The query transformed raw transaction data into strategic insight.

---

# 📌 Query 4: Customer Profitability — Identifying the Real Value Drivers

### 🎯 What this powers:

The customer profitability scorecard.

```sql id="e73iqr"
SELECT
    c.custmer_name,

    SUM(s.sales_amount) AS revenue,

    ROUND(
        SUM(s.sales_amount) * 100.0 /
        SUM(SUM(s.sales_amount)) OVER (),
        2
    ) AS revenue_contribution_pct,

    ROUND(
        SUM(s.profit_margin) * 100.0 /
        SUM(SUM(s.profit_margin)) OVER (),
        2
    ) AS profit_margin_contribution_pct,

    ROUND(
        SUM(s.profit_margin) * 100.0 /
        NULLIF(SUM(s.sales_amount),0),
        2
    ) AS profit_margin_pct

FROM sales_transactions s
JOIN customers c
ON s.customer_code = c.customer_code

WHERE s.sales_amount > 0

GROUP BY c.custmer_name
ORDER BY revenue DESC;
```

### 💡 Hidden insight behind the query

This query exposed one of the most dangerous business dependencies:

# ⚠️ Electricalsara Stores generated nearly 42% of total revenue.

But despite that scale:

* its margins remained weak.

The query forced leadership to confront an uncomfortable reality:

> The company’s largest customer was not its healthiest customer.

The:

```sql id="vjlwm7"
NULLIF(...)
```

logic protected the dashboard from division-by-zero issues, ensuring profitability calculations remained stable even for edge-case records.

---

# 📌 Query 5: Monthly Revenue Trend — Detecting the Slowdown Early

### 🎯 What this powers:

The Revenue Trend line chart.

```sql id="v9xk1o"
SELECT
    DATE_TRUNC('month', s.order_date) AS revenue_month,
    SUM(s.sales_amount)               AS monthly_revenue

FROM sales_transactions s

WHERE s.sales_amount > 0

GROUP BY DATE_TRUNC('month', s.order_date)

ORDER BY revenue_month ASC;
```

### 💡 Why monthly granularity mattered

Annual reports would have hidden the problem.

But monthly analysis revealed:

# 📉 A sustained decline beginning after mid-2018

The business could finally visualize:

* when the slowdown started,
* how long it persisted,
* and how severe the deterioration became.

This query transformed historical transactions into an operational timeline.

---

# 📌 Query 6: Revenue vs Prior Year — Measuring Business Deterioration

### 🎯 What this powers:

The Revenue vs Revenue LY comparison trend.

```sql id="ff1ptd"
SELECT
    curr.revenue_month,
    curr.monthly_revenue AS revenue,
    prev.monthly_revenue AS revenue_ly,

    ROUND(
        (curr.monthly_revenue - prev.monthly_revenue) * 100.0 /
        NULLIF(prev.monthly_revenue,0),
        2
    ) AS yoy_growth_pct

FROM (
    SELECT
        DATE_TRUNC('month', order_date) AS revenue_month,
        SUM(sales_amount) AS monthly_revenue
    FROM sales_transactions
    WHERE sales_amount > 0
    GROUP BY DATE_TRUNC('month', order_date)
) curr

LEFT JOIN (
    SELECT
        DATE_TRUNC('month', order_date) AS revenue_month,
        SUM(sales_amount) AS monthly_revenue
    FROM sales_transactions
    WHERE sales_amount > 0
    GROUP BY DATE_TRUNC('month', order_date)
) prev

ON curr.revenue_month = prev.revenue_month + INTERVAL '1 year'

ORDER BY curr.revenue_month;
```

### 💡 Business impact

This query made year-over-year deterioration impossible to ignore.

The comparison showed:

# ❌ No month in 2020 outperformed its prior-year equivalent.

That insight shifted the dashboard from:

* descriptive reporting,
  to:
* performance diagnosis.

---

# 📌 Query 7: Market Profitability in 2020 — Stress Testing the Business

### 🎯 What this powers:

2020 Market Profitability Analysis.

```sql id="8qtr7t"
SELECT
    m.markets_name,

    SUM(s.sales_amount)  AS revenue,
    SUM(s.profit_margin) AS total_profit,

    ROUND(
        SUM(s.profit_margin) * 100.0 /
        NULLIF(SUM(s.sales_amount),0),
        2
    ) AS profit_margin_pct

FROM sales_transactions s
JOIN markets m
ON s.market_code = m.markets_code

WHERE s.sales_amount > 0
AND s.order_date BETWEEN '2020-01-01' AND '2020-06-30'

GROUP BY m.markets_name
ORDER BY profit_margin_pct DESC;
```

### 💡 Why this mattered

This query showed which markets remained resilient during business decline.

Markets like:

* Bhubaneshwar
* Hyderabad

maintained strong margins despite difficult conditions.

Meanwhile:

* Delhi NCR margins collapsed,
* Bengaluru continued generating losses.

The dashboard evolved from performance reporting into operational risk analysis.

---

# 📌 Query 8: Identifying Loss-Making Customers in 2020

### 🎯 What this powers:

The 2020 Customer Profitability table.

```sql id="1n4mgh"
SELECT
    c.custmer_name,

    SUM(s.sales_amount) AS revenue,

    ROUND(
        SUM(s.sales_amount) * 100.0 /
        SUM(SUM(s.sales_amount)) OVER (),
        2
    ) AS revenue_contribution_pct,

    ROUND(
        SUM(s.profit_margin) * 100.0 /
        SUM(SUM(s.profit_margin)) OVER (),
        2
    ) AS profit_margin_contribution_pct,

    ROUND(
        SUM(s.profit_margin) * 100.0 /
        NULLIF(SUM(s.sales_amount),0),
        2
    ) AS profit_margin_pct

FROM sales_transactions s
JOIN customers c
ON s.customer_code = c.customer_code

WHERE s.sales_amount > 0
AND s.order_date BETWEEN '2020-01-01' AND '2020-06-30'

GROUP BY c.custmer_name

ORDER BY profit_margin_contribution_pct ASC;
```

### 💡 What leadership discovered

The query revealed that:

* some previously healthy accounts had become unprofitable during 2020.

Customers like:

* Electricalsquipo Stores
* Epic Stores
* Expression

moved into negative margin territory.

This enabled proactive commercial intervention before losses escalated further.

---

# 📌 Query 9: Revenue by Customer Type — Understanding Channel Dependency

### 🎯 What this powers:

The Brick & Mortar vs E-Commerce donut chart.

```sql id="swtxf5"
SELECT
    c.customer_type,

    SUM(s.sales_amount) AS revenue,

    ROUND(
        SUM(s.sales_amount) * 100.0 /
        SUM(SUM(s.sales_amount)) OVER (),
        2
    ) AS revenue_pct

FROM sales_transactions s
JOIN customers c
ON s.customer_code = c.customer_code

WHERE s.sales_amount > 0

GROUP BY c.customer_type
ORDER BY revenue DESC;
```

### 💡 Why this small query carried strategic weight

The result set only returned two rows.

But those two rows revealed:

# 📦 Brick & Mortar dominated 75.6% of total revenue.

At the same time:

* overall revenue was declining,
* while digital commerce globally accelerated.

The query surfaced an important strategic concern:

> Was the business adapting slowly to channel evolution?

Sometimes the simplest queries reveal the biggest questions.

---

# 📌 Query 10: Ranking Customers by Profit Efficiency — Finding the Hidden Stars

### 🎯 What this powers:

Strategic customer prioritization analysis.

```sql id="c1njlwm"
WITH customer_metrics AS (

    SELECT
        c.custmer_name,

        SUM(s.sales_amount) AS revenue,
        SUM(s.profit_margin) AS total_profit,

        ROUND(
            SUM(s.profit_margin) * 100.0 /
            NULLIF(SUM(s.sales_amount),0),
            2
        ) AS profit_margin_pct,

        ROUND(
            SUM(s.sales_amount) * 100.0 /
            SUM(SUM(s.sales_amount)) OVER (),
            2
        ) AS revenue_contribution_pct,

        ROUND(
            SUM(s.profit_margin) * 100.0 /
            SUM(SUM(s.profit_margin)) OVER (),
            2
        ) AS profit_margin_contribution_pct

    FROM sales_transactions s
    JOIN customers c
    ON s.customer_code = c.customer_code

    WHERE s.sales_amount > 0

    GROUP BY c.custmer_name
),

ranked AS (

    SELECT *,

           RANK() OVER (ORDER BY profit_margin_pct DESC)
           AS rank_by_margin,

           RANK() OVER (ORDER BY revenue DESC)
           AS rank_by_revenue,

           rank_by_revenue - rank_by_margin
           AS rank_gap

    FROM customer_metrics
)

SELECT *
FROM ranked
ORDER BY rank_gap DESC;
```

### 💡 The most strategic query in the project

This query identified:

# ⭐ Hidden high-value customers

The:

```sql id="4b22ow"
rank_gap
```

logic measured the difference between:

* revenue ranking,
  and:
* profitability ranking.

This revealed customers who:

* were small in scale,
* but extremely efficient operationally.

The business could now distinguish:

* “big customers”
  from
* “valuable customers.”

That distinction changed how leadership thought about growth strategy.

---

# 🎯 Final Thought

SQL did far more than extract numbers for this dashboard.

It:

* uncovered operational risk,
* revealed hidden profitability patterns,
* identified inefficient markets,
* surfaced customer dependencies,
* and transformed raw transaction records into business intelligence.

The visuals made the insights visible.

But SQL is what made the insights possible.

---

*All queries written using PostgreSQL-compatible SQL syntax*
*Dashboard Tool: Power BI Desktop*
*Analysis Period: June 2017 – June 2020*
