# Supply Chain Analytics Dashboard

An end-to-end analytics project: purchase-order data from Excel was loaded into **SQL Server**, cleaned and prepared with **T-SQL**, and visualised in a multi-page **Power BI** dashboard.

The goal is to answer practical supply chain questions: how much is being spent, which orders are late or still open, how reliable are the suppliers, and which products are running low on stock.

## Tools

- **SQL Server (SSMS)**: data import (SSIS / Import and Export Wizard), validation checks, cleaning, reporting view, KPI queries
- **Power BI Desktop**: data model, DAX measures, interactive dashboard

## Data

The source is an Excel workbook with four tables:

| Table | Content |
|---|---|
| Suppliers | Supplier master data (8 suppliers listed) |
| Products | Product master data, category, reorder level |
| Purchase Orders | 180 purchase orders with dates, quantity, unit cost, status |
| Inventory | Quantity on hand per product and warehouse |

The data is synthetic and contains deliberate inconsistencies for practice.

## Data cleaning decisions

- **Duplicates:** the SQL tables contained every record 8 times (for example 1,440 rows for 180 unique purchase orders) because of repeated imports. The Excel source itself has no duplicates. Duplicates are removed in the SQL view with `ROW_NUMBER()`.
- **Text values:** supplier and product text was trimmed and standardised, and status values with mixed case (`received`, `IN TRANSIT`) were converted to one consistent form.
- **Blank received dates:** left blank for open and in-transit orders, because they have not arrived yet.
- **Contradictory records:** 5 orders were flagged instead of changed (3 marked *Received* with no received date, 2 marked *In transit* that already have a received date).
- **Source tables are never modified.** All cleaning happens in the reporting view `dbo.vw_PurchaseOrderReport`.

## Metric definitions

| Metric | Definition |
|---|---|
| Total spend | Sum of order quantity x unit cost |
| Open purchase orders | Orders with status *Open* or *In transit* |
| Overdue purchase orders | Open or in-transit orders past their expected date, plus received orders that arrived after the expected date |
| On-time delivery rate | Orders received on or before the expected date, out of received orders with a received date |
| Average lead time | Average days from order date to received date, for received orders |
| Delivery variance (days) | Received date minus expected date (negative means early) |
| Low-stock products | Products whose total quantity on hand is below their reorder level |
| Shortfall | Reorder level minus stock on hand, when stock is below the level |

## Dashboard pages

1. **Overview:** KPI cards (spend, orders, open orders, low-stock products, overdue orders, on-time rate), monthly spend trend, order status breakdown, and slicers (order date, supplier, category, status, warehouse).
2. **Suppliers & products:** spend by supplier, on-time delivery by supplier, spend by category, stock on hand versus reorder level by product.
3. **Detail table:** full purchase-order list with delivery variance in days.
4. **Extra insights:** overdue orders by warehouse, average lead time by supplier, top 5 products by spend.
5. **Data quality:** records with data issues, issue breakdown, list of records to review.
6. **Delivery analysis:** value of open orders, late received orders, overdue aging, delivery timing (early / on time / late), the 10 most overdue open orders.
7. **Inventory & reorder:** low-stock products, total shortfall, and whether open orders cover the shortfall.

## Key findings

- Total spend is about **620.6K** across **180** purchase orders.
- **89** orders (about **287.8K**, roughly 46% of spend) are still open or in transit.
- **124** orders are overdue. Most of the open overdue orders are more than 90 days late, and the oldest is **644 days** late.
- Of the 88 received orders with a received date: **36 early, 17 on time, 35 late**. Late deliveries are only about **1.5 days** late on average, so the main problem is orders that have not arrived at all.
- On-time delivery rate is **60.2%** and average lead time is about **15 days**.
- **7 products** are below their reorder level (162 units short in total). Each has open orders that would cover the gap, but those orders are heavily delayed.
- Only 2 of the 8 suppliers have any purchase orders.

## Recommendations

- Follow up on open orders that are more than 90 days overdue: confirm with suppliers, then cancel or re-issue.
- Expedite the low-stock products first, since the stock gap is real while their open orders are late.
- Review suppliers on delivery performance and consider reducing dependence on the two active ones.
- Add a data-entry check so that received orders always have a received date and open orders never do.

## Files

- `Dashboard_tttt2.pbix`: the Power BI dashboard (open with Power BI Desktop)

## How to open

Download the `.pbix` file and open it with **Power BI Desktop** (free, Windows).
