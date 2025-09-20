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

**Strategic direction:**
- Seasonality playbook: lock a calendar that ramps inventory and marketing from August, with hero-SKU bundles launching in October.
- Supply resilience: pre-book supplier capacity and inbound freight for Oct–Nov, mandatory to raise safety stock on top movers.
- Ops readiness: scale fulfilment staffing and carrier allocations into the peak to communicate shipping cut-offs early more effectively.
- Pricing & promos: use price tests in shoulder months (Sep, early Oct) to set Q4 offer depth, for multipacks for bulk buyers, it could emphasise more directly.
- Post-peak hygiene: plan December clearance and replenishment freeze to avoid overhang and convert for preparation for traffic to email/loyalty for Q1.
- Analytics guardrails: validate December completeness, add a business-day normalisation / 3-month moving average, and repeat the chart with revenue to confirm that quantity peaks translate to value.

## Which countries purchase the most?
The bar chart below ranks total quantity by country. One market (United Kingdom) dominates overwhelmingly, while a secondary cluster (Netherlands, Germany, Ireland, France) contributes materially but at a far lower order of magnitude. Beyond that sits a long tail (Sweden, Australia, Switzerland, Portugal, Norway) with fragmented, placed in low demand.

<p align="center">
  <img src="https://github.com/Catherinerezi/ecommerce-data-analysis/blob/main/assets/Bar2.png" alt="Which countries purchase the most" width="500">
</p>

**Current conditions:**
- Volume is heavily concentrated domestically, consistent with UK as an anchor to storefront, shorter, and cheaper delivery routes.
- Netherlands and Germany emerge as the strongest export corridors in continental Europe, proximity and logistics density likely lower friction.
- The tail countries show episodic or niche demand, some spikes may reflect single wholesale buyers rather than broad retail pull.
- As the ranking uses quantity not revenue, lightweight, low-price SKUs may inflate shares, adviseable to confirm with a TotalPrice.

**Strategic direction:**
- Defend the core (UK): guarantee service levels on hero SKUs—higher safety stock, faster replenishment, and next-day SLAs.
- EU expansion plays: pilot localized storefronts for NL/DE/FR (language, currency, taxes), add local payment methods (e.g., iDEAL in NL), and test regional campaigns.
- Logistics leverage: consider a Benelux 3PL hub to compress delivery times and reduce cross-border costs especially for pre-position inventory in Q4.
- Wholesale/B2B capture: in mid-tier countries, pursue reseller outreach and multipack pricing where bulk behaviour appears.
- Pricing & landed cost: simulate end-to-end costs (shipping, duties, VAT) and tune free-shipping thresholds per country.
- Analytics guardrails: track share of revenue vs. quantity, repeat analysis on per-customer basis (to separate broad demand from single large buyers), and monitor returns/cancellations.

## What is the top product per country?
The chart below surfaces the single top-selling product in each country (by total quantity). The United Kingdom leads decisively, with its hero SKU outstripping every other market by a wide margin. A second tier, Sweden and Australia, followed by Netherlands, Ireland, and France shows meaningful but far smaller peaks. Across markets, winners cluster in gift/home decor and partyware (e.g., T-light holders, cake cases, keepsakes), signalling impulse-friendly items with low shipping friction.

<p align="center">
  <img src="https://github.com/Catherinerezi/ecommerce-data-analysis/blob/main/assets/Bar3.png" alt="What is the top product per country" width="500">
</p>

**Current conditions:**
- Demand is country-specific yet motif-consistent: different markets crown different heroes, but themes recur (hearts, candles, party cases).
- Several countries exhibit concentrated SKU dependence, one item captures a disproportionate share of local volume.
- Volumes reflect a blend of retail and small-wholesale behaviour, this spikes likely coincide with events and seasonal restocking.

**Strategic direction:**
- Defend & extend heroes: For each country’s top SKU, raise safety stock, tighten lead times, and launch closely related variants (colour/pack-size).
- Localise merchandising: Translate copy, adapt imagery, and sequence campaigns to local holidays, need a test for bundles/multipacks around the hero in the future.
- Regional ops: Pre-position inventory for Sweden/Australia peaks, need a trial with a Benelux hub for NL/IE/FR to compress delivery times in the future.
- Portfolio balance: Reduce over-reliance on single items by promoting adjacent complements (holders to candles, cases to baking accessories).
- Pricing discipline: A/B test unit vs. multipack pricing to ensure landed-cost economics (shipping, duties, VAT) remain positive by country.
- Analytics guardrails: Track share of revenue, repeat rates, and per-customer concentration to distinguish broad market pull from reseller spikes.

## Which items are bought in bulk (Quantity > threshold)?
The chart below ranks top products bought in large quantities (Quantity > 100 per transaction). A familiar cast WHITE HANGING HEART T-LIGHT HOLDER, CREAM HEART CARD HOLDER, and WRAP, BILLBOARD FONTS DESIGN leads again, but here their dominance reflects like wholesale behaviour, not casual retail. The distribution is steeply top-heavy: a few SKUs account for most bulk volume, followed by a compressed tail.

<p align="center">
  <img src="https://github.com/Catherinerezi/ecommerce-data-analysis/blob/main/assets/Bar4.png" alt="Which items are bought in bulk (Quantity > threshold)" width="500">
</p>

**Current conditions:**
- Bulk demand clusters around gift/partyware items that are lightweight, durable, and easy to ship in cartons.
- Spikes are likely driven by resellers, events, or corporate gifting, not one-off anomalies.
- Because the ranking is quantity-based, margin may diverge from volume; validate contribution after shipping and handling.
- Concentration risk: a small set of SKUs underpins bulk revenue, supply hiccups there would be outsized.

**Strategic direction:**
- Stock & upstream planning: lock vendor capacity and shorter lead times for the top SKUs, this hold carton-level safety stock ahead of peaks.
- Wholesale packaging: offer multipacks/carton sizes and publish master case quantities to improve pick efficiency and freight economics.
- Tiered pricing: introduce volume breaks and contract pricing for verified B2B accounts; test MOQ to protect margins.
- Assortment extensions: create close variants (colour/pack-count) around winners to capture bulk intent without fragmenting demand.
- Targeted acquisition: build a reseller programme and prioritise regions/accounts showing repeated high-quantity orders.
- Risk controls: monitor returns/cancellations for bulk orders and guard against stockouts with daily service-level dashboards.

## Multivariate snapshot: are numeric fields aligned?
The heatmap below shows TotalPrice is driven primarily by Quantity (ρ about 0.79), while its link to UnitPrice is weak (ρ about 0.09). Quantity and UnitPrice are essentially uncorrelated, even slightly negative (ρ about −0.08). In plain terms: revenue is volume-led, not price-led; big baskets and bulk lines, rather than high unit prices, explain most variation in takings.

<p align="center">
  <img src="https://github.com/Catherinerezi/ecommerce-data-analysis/blob/main/assets/Heatmap.png" alt="Multivariate snapshot: are numeric fields aligned" width="500">
</p>

**Current conditions:**
- Large swings in Quantity dominate revenue, this is consistent with B2B/reseller behaviour and multipack purchasing.
- Price shifts alone have little linear association with sold units at the global level this pictured that elasticities likely vary by product/region.
- Because TotalPrice = Quantity × UnitPrice, the strong tie to Quantity indicates quantity variance is much greater than price variance in this dataset.

**Strategic direction:**
- Availability over discounting: protect stock and service levels on high-velocity SKUs, warn to not rely on blanket price cuts to move volume.
- Bundle & pack-size design: capture volume-led revenue with multipacks/tiers instead of margin-eroding discounts.
- Targeted price tests: run controlled A/B by category/country to find pockets where price does move units, advice to scale only where lift > margin loss.
- Account programmes: formalise wholesale terms for repeat bulk buyers (MOQs, volume breaks, lead-time commitments).
