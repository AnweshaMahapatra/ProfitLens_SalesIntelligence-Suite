# 📊 Sales Insight: Revenue, Profit & Performance Intelligence Dashboard

> *A multi-page Power BI solution built to uncover hidden profitability gaps, market concentration risks, and channel performance trends across a distributed sales network in India.*

---

## Overview

This project delivers a production-grade business intelligence dashboard built in Power BI Desktop, analysing **₹985M in revenue** across **14 Indian markets**, **25+ customers**, and a **3-year sales timeline (2017–2020)**. The solution spans three analytical layers — Key Insights, Profit Analysis, and Performance Analysis — giving leadership a single source of truth for revenue tracking, margin diagnostics, and customer profitability scoring.

The dashboard moves beyond surface-level reporting. It exposes the uncomfortable reality that while top-line revenue looks healthy, **total profit margin stands at just ₹24.7M (roughly 2.5%)**, and two markets are actively destroying value.

---

## Business Context

The organisation operates a large wholesale distribution network across India, selling through both Brick & Mortar retail chains and E-Commerce channels. As the business scaled geographically, leadership lost visibility into *where* profit was actually being made — and *where* it was being eroded.

Revenue data existed in transactional systems, but no consolidated view connected market performance, customer profitability, and channel mix. Decisions were being made on gut feel, regional anecdotes, and lagging monthly reports that arrived too late to act on.

This dashboard was built to change that.

---

## Project Objectives

- Identify which markets drive revenue versus which drive *profit* — these are not the same markets
- Surface customers with high revenue contribution but dangerously low (or negative) profit margins
- Quantify the revenue dependency risk created by a single dominant customer
- Track revenue trend deterioration over 2019–2020 and isolate the inflection point
- Enable management to set and monitor profit margin targets dynamically (2% baseline target)
- Provide a channel-level view of Brick & Mortar vs E-Commerce revenue split

---

## Dashboard Capabilities

**Key Insights Page**
- Total revenue (₹985M) and total sales quantity (2M units) at a glance
- Revenue and quantity breakdown by all 14 markets — ranked and sortable
- Revenue trend line from Jan 2018 to Jun 2020, revealing a sustained decline from peak ₹40M/month
- Customer type split: Brick & Mortar (₹745M, 75.6%) vs E-Commerce (₹240M, 24.4%)
- Top customers and top product codes by revenue contribution

**Profit Analysis Page**
- Three parallel market rankings: Revenue Contribution %, Profit Margin Contribution %, and Profit Margin %
- Full customer profitability table: revenue, revenue contribution %, profit margin contribution %, and margin %
- Instant identification of loss-making markets (Bengaluru: –20.8%, Kanpur: –0.5%)
- Identification of high-margin but low-revenue accounts (Leader: 7.5% margin on ₹17M)

**Performance Analysis Page**
- Filtered view for 2020 (Jan–Jun) against a user-adjustable Profit Target (default: 2%)
- Combined Revenue Trend chart overlaying: current revenue, prior year revenue, and profit margin %
- 2020-specific market margin rankings highlighting structural shifts post-2019
- Identification of customers turning loss-making in 2020 (Epic Stores: –4.7%, Electricalsquipo: –11.5%)

---

## Analytics & Data Modeling Approach

**Data Cleaning & Power Query**
- Standardised market names and customer name fields across source tables
- Removed or flagged blank/null product codes (visible as "(Blank)" category contributing ₹0.47bn — a data quality issue surfaced and tracked)
- Handled currency formatting consistently across INR values at different scales (M, bn)
- Date table created to enable year and month-level filtering via slicers

**DAX Measures**
- `Revenue` — SUM of transaction-level sales amounts
- `Sales Qty` — SUM of units sold
- `Total Profit Margin` — calculated from cost and revenue fields
- `Profit Margin %` — `DIVIDE([Profit], [Revenue], 0)` to avoid division-by-zero errors
- `Revenue Contribution %` — market/customer revenue as a share of total, using `ALL()` for denominator
- `Profit Margin Contribution %` — profit share relative to total profit pool
- `Revenue LY` — prior year revenue using `SAMEPERIODLASTYEAR()` for YoY comparison in Performance page
- Dynamic Profit Target parameter — implemented as a What-If parameter enabling threshold-based filtering

**Data Model**
- Star schema: central fact table (transactions) linked to dimension tables for customers, markets, products, and dates
- Relationships enforced on market codes and customer IDs to prevent fan-trap aggregation errors

**Visualization Approach**
- Horizontal bar charts for ranked market/customer comparisons — avoids misleading pie chart distortions
- Area/line combo chart for revenue trend — overlays margin % on a secondary axis for correlation reading
- Donut chart for channel mix — appropriate for two-segment proportional comparison
- Consistent purple/indigo brand palette throughout with white card backgrounds for visual hierarchy
- Navigation buttons on home page for multi-page drill-through experience

---

## Key Business Highlights

- **Delhi NCR dominates everything**: 52.76% of revenue, 48.5% of profit contribution — but only a 2.3% margin, flagging scale without efficiency
- **Bengaluru is a value destroyer**: –20.8% profit margin — every rupee of revenue generated there costs more than it earns
- **Electricalsara Stores is a concentration risk**: ₹413M revenue (41.97% of total) but only 2.3% profit margin — the business's largest customer is also one of its least profitable
- **Revenue has been declining since mid-2018**: The trend chart shows a clear peak around ₹40M/month followed by a sustained downward trajectory into 2020
- **Surat leads all markets on margin efficiency**: 4.9% profit margin despite contributing only 0.26% of revenue — a model worth studying for scalability
- **E-Commerce is underdeveloped**: At 24.4% of revenue (₹240M), the digital channel has significant headroom relative to Brick & Mortar

---

## Repository Structure

```
sales-insight-powerbi/
│
├── README.md                              ← You are here
├── The Story Behind the Dashboard.md     ← Consulting narrative & executive analysis
├── Business_Questions_and_Insights.md    ← 10 strategic Q&A derived from dashboard
├── SQL_Queries_Behind_the_Insights.md    ← SQL logic that powered the dashboard metrics
│
├── dashboard/
│   └── Sales_Insight_Report.pbix         ← Power BI source file
│
├── exports/
│   └── Sales_insight_project_report.pdf  ← Dashboard PDF export (this file)
│
└── assets/
    └── screenshots/                       ← Dashboard page screenshots
```

---

## Additional Project Resources

- **[The Story Behind the Dashboard.md]** — A consulting-style narrative that walks through the business problem, what the data reveals, and strategic recommendations for leadership. Written for executive audiences.
- **[SQL_Queries_Behind_the_Insights.md]** — The actual SQL logic used to extract, aggregate, and shape the data that feeds this dashboard, with business interpretation for each query.

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| Power BI Desktop | Dashboard development, DAX, data modeling |
| Power Query (M) | Data transformation and cleaning |
| DAX | KPI measures, YoY calculations, contribution % |
| SQL | Source data extraction and pre-aggregation |
| Microsoft Excel / CSV | Raw data staging |
| GitHub | Portfolio hosting and version control |

---

## Conclusion

This project demonstrates that business intelligence is not about building dashboards — it's about changing decisions. The Sales Insight report took a business sitting on ₹985M in revenue and revealed that net profit was a thin ₹24.7M, that its largest customer was also one of its least profitable, and that two markets were actively destroying value while leadership continued investing in them.

That is what good BI looks like.

---

## Author

**[Your Name]**
Data Analyst | Business Intelligence Developer | Power BI Specialist

- 🔗 LinkedIn: [linkedin.com/in/yourprofile]
- 💻 GitHub: [github.com/yourusername]
- 📧 Email: [your.email@domain.com]

*Open to data analyst, BI developer, and analytics consultant roles.*
