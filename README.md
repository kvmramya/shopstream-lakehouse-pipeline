# shopstream-lakehouse-pipeline
End-to-end Medallion Data Pipeline on Databricks using Spark SQL, Delta Lake, and Unity Catalog.
# ShopStream Lakehouse Data Pipeline (Medallion Architecture)

An end-to-end Data Engineering pipeline built on Databricks using Spark SQL, Delta Lake, and Unity Catalog. This project processes raw e-commerce transaction data through a Medallion Architecture (Bronze → Silver → Gold) to power executive analytics and reporting dashboards.

---

## 🏗️ Architecture Overview
---

## 💡 Business Problem & Pipeline Workflow

E-commerce order streams often contain data quality issues such as inconsistent string casing, duplicate event fires, negative or zero quantities, and cancelled orders that skew revenue calculations. 

This pipeline automates ingestion, cleans transactional anomalies, enforces relational integrity, and constructs analytical tables for executive decision-making.

### 1. Bronze Layer (Ingestion)
- Ingests raw CSV source files into Delta Lake tables using idempotent `COPY INTO` syntax.
- Preserves raw schema and captures metadata (`_rescued_data`).
- Tables created:
  - `shopstream.core.bronze_orders` (13,717 records)
  - `shopstream.core.bronze_customers` (1,000 records)
  - `shopstream.core.bronze_products` (197 records)

### 2. Silver Layer (Cleaning & Data Quality)
- **Status Normalization:** Standardized mixed-case status values (`COMPLETED` → `completed`).
- **Deduplication:** Applied window functions (`ROW_NUMBER() OVER (PARTITION BY order_line_id ORDER BY order_ts)`) to remove duplicate order line events.
- **Data Filtering:** Filtered out invalid records where `quantity <= 0` and purged `cancelled` orders.
- **Auditability:** Leveraged Delta Lake Time Travel (`VERSION AS OF 0`) to track data changes across state updates.
- **Enriched Dimensions:** Calculated `unit_margin` (`unit_price - unit_cost`) for product analysis.

### 3. Gold Layer (Business Aggregations)
- **`gold_daily_revenue`**: Daily total completed order count, units sold, and net revenue.
- **`gold_category_performance`**: Category-level breakdown of orders, total sales, revenue, and gross profit margin.
- **`gold_customer_ltv`**: Aggregated customer metrics identifying top-value customers by lifetime revenue and total orders.

---

## 📊 Analytics Dashboard

The Gold layer datasets feed a Databricks SQL Dashboard visualizing sales trends and category-level gross margin performance:

![Dashboard Preview](docs/dashboard_preview.png)

---

## 🛠️ Technology Stack

- **Platform:** Databricks (Runtime 19.5 Photon / Apache Spark)
- **Storage & Governance:** Delta Lake, Unity Catalog (`shopstream.core`)
- **Languages:** Spark SQL, Python (PySpark)
- **Data Visualization:** Databricks SQL Dashboards

---

