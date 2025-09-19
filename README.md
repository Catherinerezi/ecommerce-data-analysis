# From Invoices to Insights: E-commerce EDA
Analyzing an e-commerce transactions dataset turns raw logs into decisions. It reveals which products truly drive volume and revenue, how demand shifts across months and countries, and where stockouts or excess inventory are likely to occur. By quantifying patterns—basket sizes, bulk-buy behavior (B2B vs. retail), and price–quantity dynamics—we can optimize assortment, pricing, and promotions instead of guessing. Feature engineering (e.g., TotalPrice, monthly cohorts) and visual EDA expose anomalies and data quality issues early, improving the reliability of every downstream model or report. The insights support concrete actions: forecast demand, prioritize high-margin items, tailor regional campaigns, and allocate inventory more efficiently. In short, rigorous EDA converts transactional noise into a roadmap for profitable growth.

# Data Understanding

This repository analyses a real-world e-commerce transactions dataset. Each row records a line item from an order, capturing what was bought, when, where, and at what price. Before any modelling or visualisation, the data are profiled, cleaned, and enriched to ensure that downstream findings are trustworthy and actionable.

| Column        | Type     | Meaning                                    | Notes & Handling |
|:--------------|:---------|:-------------------------------------------|:-----------------|
| InvoiceNo     | int/str  | Unique invoice identifier                   | Dropped (ID only) |
| StockCode     | str      | Product SKU                                 | Keep for joins |
| Description   | str      | Human-readable product name                 | Required; drop NA |
| Quantity      | int      | Units purchased                             | Coerce numeric; keep outliers |
| InvoiceDate   | dt       | Transaction timestamp                       | Parse to datetime; derive Month |
| UnitPrice     | float    | Price per unit                              | Coerce numeric; sanity-check |
| CustomerID    | int/str  | Pseudonymous buyer ID                       | Dropped (ID only) |
| Country       | str      | Customer country                            | Required; segmentation |
| TotalPrice*   | float    | Quantity × UnitPrice                        | Engineered |
| Month*        | str      | Year-Month from InvoiceDate                 | Engineered |

*Engineered during feature engineering.

**Data quality summary.** Duplicates are checked; critical missing values in Description, InvoiceDate, and Country are removed. Numeric columns are coerced to the right types. Outliers in Quantity/UnitPrice are flagged but preserved, as they often indicate legitimate bulk purchases (B2B) rather than noise. These steps produce a clean df_ready table suitable for robust EDA.

# Attachment:
- [Data processing (Indonesian)](https://colab.research.google.com/drive/1-pTq6D350q5LmdVpiIF-zh1TIzRsv0kw?usp=sharing)

# Data Processing 
Scope: Each row is a line item: product, quantity, price, timestamp, and country.

**Type casting & formatting:**
- Parse InvoiceDate to datetime derive Month (YYYY-MM) for cohort analysis.
- Coerce Quantity and UnitPrice to numeric.
- Create TotalPrice as Quantity * UnitPrice (line-level revenue).

**Column selection:**
- Drop identifiers that are not analytically meaningful in aggregation: InvoiceNo, CustomerID.
- Keep StockCode for reference: keep Country, Description as analytical dimensions.

**Missing values:**
- Treat Description, InvoiceDate, and Country as essential and drop rows where these are missing.
- After coercion, remove rows with invalid InvoiceDate.

**Duplicates & header artefacts:**
- Check for duplicate rows.
- Detect accidental “header rows” inside the body (string matches like InvoiceNo, Description, Quantity) and remove if present.

**Outlier assessment (kept by design):**
- Profile numeric columns with IQR rules.
- Retain high-quantity or high-price outliers: in retail, extremes often reflect legitimate bulk/B2B behaviour and are analytically informative.

**Integrity checks:**
- Confirm expected shape, dtypes, and descriptive statistics after each transformation.
- Validate that TotalPrice, Quantity, and UnitPrice align logically (no negative totals, absurd prices).

# Explanatory Data Analysis Visualizations Carried Out
Visualization Type Purpose:

## Which products dominate volume?
The chart ranks the ten highest-velocity SKUs by total quantity sold. A small cadre of giftable, lightweight items that led decisively by 'WHITE HANGING HEART T-LIGHT HOLDER', with 'CREAM HEART CARD HOLDER' and 'WRAP', 'BILLBOARD FONTS DESIGN' close behind the domination of the catalogue. The distribution is top-heavy: sharp drop-off after the leaders, then a long tail of niche products. Such a profile is consistent with impulse-friendly items that ship cheaply and lend themselves to repeat or bulk orders.

<p align="center">
  <img src="https://github.com/Catherinerezi/ecommerce-data-analysis/blob/main/assets/Bar1.png" alt="Which products dominate volume" width="500">
</p>

**Current conditions:** Demand is concentrated in a handful of hero SKUs where several lines display quantities suggestive of B2B/reseller behaviour alongside retail traffic. Because the ranking is by volume rather than revenue, some leaders may be low-margin, however, margin validation is essential before scaling inventory.

**Strategic direction:**
- Protect the winners: Raise safety stock and shorten reorder cycles for the top five SKUs and monitor stockouts daily.
- Shape the assortment: Extend proven motifs (hearts, typography) and retire chronically slow tail items.
- Monetise bulk intent: Offer bundles/multipacks around leaders to lift average order value.
- Time promotions to seasonality: Use hero SKUs as traffic drivers to cross-sell complementary goods.
- Price with discipline: Test unit vs. multipack pricing: prioritise SKUs with healthy contribution margins.
- Harden supply: Dual-source top sellers and pre-book capacity ahead of peaks.

## How does demand evolve by month?
The line chart tracks total quantity by month from late 2010 through 2011. After a soft start and a January trough, demand stabilises through Q1, dips in April, then builds steadily from May, accelerating into Q3–Q4 and peaking in November, a textbook holiday run up before an abrupt December pullback.

<p align="center">
  <img src="https://github.com/Catherinerezi/ecommerce-data-analysis/blob/main/assets/line.png" alt="How does demand evolve by month" width="500">
</p>

**Current conditions:**
- Momentum concentrates in late Q3 and Q4, consistent with gifting/seasonal events and wholesale restocking.
- Early-year volumes are fragile, with small oscillations that suggest demand is the sensitivity of the promotions.
- The sharp December drop may reflect either post-peak cooling or partial-month data. Adviceable for completeness checks.
- Because this view is based on volume, it may overweight in low-price SKUs. Hence, a revenue lens (TotalPrice) should be reviewed in parallel.

## Which countries purchase the most?
## What is the top product per country?
## Which items are bought in bulk (Quantity > threshold)?
## Multivariate snapshot: are numeric fields aligned?
