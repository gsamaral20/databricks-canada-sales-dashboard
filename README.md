
# Databricks Canada Sales Dashboard

An AI/BI dashboard built on Databricks Free Edition, following the *AI/BI for Data Analysts* course from Databricks Academy. It tracks sales performance against yearly targets and the health of the sales pipeline for a fictional Canadian B2B business.

> **About the data:** the course lab notebooks and datasets are part of a paid subscription, so I did not use them. I rebuilt the five source tables myself with SQL, using synthetic data. Fifteen customers, the product catalog and the yearly targets mirror what the course shows. The other customers, all orders and all opportunities were generated in SQL. Numbers here will not match the course.

## What is in this repo

```
├── sql/        Setup and data-generation scripts (run in order)
└── dashboard/  Dashboard draft (.lvdash.json)
```

| Script | Purpose |
|---|---|
| `00_setup` | Creates the catalog `ai_bi_data_analyst`, the schema `aibi_dashboards` and a volume |
| `01_ca_products` | Product catalog (9 rows) |
| `02_ca_customers` | Customers (40 rows) |
| `03_ca_sales_targets` | Yearly sales targets, 2021 to 2024 (4 rows) |
| `04_ca_orders` | Order fact table (15,536 rows), generated |
| `05_ca_opportunities` | Sales pipeline (342 rows), generated |
| `06_validacao` | Row-count check for all five tables |

## Data model

| Table | Role | Rows |
|---|---|---|
| `ca_orders` | Fact table: orders from Nov 2021 to Dec 2024 | 15,536 |
| `ca_products` | Dimension, joined on `productid` | 9 |
| `ca_customers` | Dimension, joined on `customerid` | 40 |
| `ca_sales_targets` | Yearly targets, matched on `YEAR(orderdate) = target_year` | 4 |
| `ca_opportunities` | Standalone pipeline table (phases 0-Prospecting to 3-Closed Lost) | 342 |

### How the synthetic data was built

- IDs are consistent across tables: orders and opportunities draw their `customerid` and `productid` from the customer and product tables, so every join resolves.
- `orderamt` is always `quantity × listprice`.
- Randomness uses fixed seeds, so the scripts are reproducible.
- Product mix in orders is weighted toward cheaper products, so yearly totals land close to the targets.
- Province in opportunities comes from the customer record.

Sales versus target, as computed by the `SalesTarget` dataset:

| Year | Target | Actual | Variance % | Status |
|---|---|---|---|---|
| 2024 | 86,000,000 | 88,705,848.16 | 3.15 | HIT |
| 2023 | 87,000,000 | 86,764,716.17 | -0.27 | MISS |
| 2022 | 80,000,000 | 84,386,247.61 | 5.48 | HIT |
| 2021 | 9,000,000 | 12,387,941.70 | 37.64 | HIT |

## Datasets

- **FullOrderData:** orders joined to customers and products (`INNER JOIN`), with a `year` column extracted from the order date.
- **SalesTarget:** a CTE (`sales_by_year`) sums sales per year and joins it to the targets. It returns `actual_sales`, `variance`, `variance_pct` and a `HIT` / `MISS` status.
- **Opportunities:** the opportunities table with a calculated column, `deal_size`.

## Dashboard pages

**Canada Sales Overview** covers sales by province and product, and performance against target.

**Opportunities Pipeline** covers pipeline health: KPI counters (total pipeline value, total opportunities, closed won / active / lost), a funnel by phase, a pie chart by deal size, monthly pipeline value by phase, and stacked bars per sales rep.

### Filters

| Page | Filters |
|---|---|
| Canada Sales Overview | Global year filter (`year` from FullOrderData and `target_year` from SalesTarget in the same control), date (monthly), sales rep, product, province (multiple values) |
| Opportunities Pipeline | Date (monthly), sales rep, province, phase |

The year filter combines two fields with different names in one control, so a single selection filters both the orders and the targets.

## Using Genie Code

I used Genie Code to generate the first version of visualizations and pages. Three prompts, taken from the course:

1. **Stacked bar chart**
   > Create a stacked bar chart and use the sum of orderamt by province. Do not use any other columns. Make it a stacked bar chart. Add a title, labels and useful display names.

2. **Pivot table**
   > Create a pivot table. Use productname as the row, and province as the column, and the sum orderamt as the value. Add a color scale and format the values using Canadian dollars and compact.

3. **Structured page prompt** (treats the page like a design spec: one widget per line)
   > Build a page on the Opportunities dataset to track the health of our sales pipeline using the Opportunities dataset within the dashboard data source on the page Opportunities Pipeline. Include:
   > - KPI counters for Total Pipeline Value, Total Opportunities, and Closed Won / Active / Lost values.
   > - A funnel chart showing pipeline value by phase.
   > - A pie chart showing count of opportunities by deal_size.
   > - A time-series of monthly pipeline value, colored by phase.
   > - Stacked bar charts showing pipeline value per sales rep, one colored by phase and one by deal_size.
   > - A detail table of all opportunities with key columns.
   > Format all dollar values as Canadian dollars, compact.

I did not accept the generated output as is. I checked each result and made targeted improvements by hand:

- Removed axis titles I found redundant
- Applied Canadian dollar formatting
- Rearranged the layout of charts and counters
- Added data labels to some charts
- Added conditional styling to `phase` and `deal_size` in the detail table

## Refresh job

To practice refresh handling, I created a Databricks job, `canadian-sales-dashboard-refresh`, with one task (`refresh-sales-canada-dashboard`) of type *Dashboard*. It runs on the Serverless Starter Warehouse (2XS) and emails a snapshot to the subscriber. The job has **no trigger**, so it only runs manually.

## Before publishing

I followed the course's pre-publication checklist: published (not draft), data permission model chosen on purpose, sharing permissions set, subscribers added, filters with sensible defaults ("All"), and business-friendly titles and axis labels.

## Version control

The repo is connected through Databricks Git folders with the Databricks GitHub app. Git tracks the dashboard draft only, and not publishing or scheduling settings, which is why the refresh job is described here instead.

## Skills practiced

SQL (joins, CTEs, aggregations, `CASE`), synthetic data generation, Unity Catalog (catalogs, schemas, volumes), AI/BI dashboards, Genie Code, Databricks Jobs, Git folders.

## Credits

Course: *AI/BI for Data Analysts*, Databricks Academy. Dashboard structure and Genie prompts follow the course; data, SQL scripts and manual refinements are my own.