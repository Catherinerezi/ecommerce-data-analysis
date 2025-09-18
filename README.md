# ecommerce-data-analysis
Analyzing an e-commerce transactions dataset turns raw logs into decisions. It reveals which products truly drive volume and revenue, how demand shifts across months and countries, and where stockouts or excess inventory are likely to occur. By quantifying patterns—basket sizes, bulk-buy behavior (B2B vs. retail), and price–quantity dynamics—we can optimize assortment, pricing, and promotions instead of guessing. Feature engineering (e.g., TotalPrice, monthly cohorts) and visual EDA expose anomalies and data quality issues early, improving the reliability of every downstream model or report. The insights support concrete actions: forecast demand, prioritize high-margin items, tailor regional campaigns, and allocate inventory more efficiently. In short, rigorous EDA converts transactional noise into a roadmap for profitable growth.

### Data Understanding

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
