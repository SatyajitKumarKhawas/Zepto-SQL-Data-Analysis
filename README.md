# 🛒 Zepto Product Catalog — SQL Analysis

> A SQL-based analysis of Zepto's quick-commerce product catalog — exploring pricing patterns, discount strategies, inventory gaps, and category-level revenue potential across 3,700+ SKUs and 14 grocery categories.

## Overview

This project performs end-to-end exploratory data analysis and business intelligence querying on a Zepto grocery product dataset using PostgreSQL. It covers data ingestion, cleaning, and 8 analytical queries designed to surface pricing insights, inventory health, and category-level revenue estimates.

---

## Dataset

| Property | Details |
|---|---|
| **File** | `zepto_v2.csv` |
| **Rows** | ~3,732 products (SKUs) |
| **Categories** | 14 |
| **Source** | Zepto quick-commerce platform |

### Columns

| Column | Type | Description |
|---|---|---|
| `sku_id` | SERIAL PK | Auto-generated unique identifier |
| `category` | VARCHAR(120) | Product category |
| `name` | VARCHAR(150) | Product name |
| `mrp` | NUMERIC(8,2) | Maximum Retail Price |
| `discountPercent` | NUMERIC(5,2) | Discount percentage offered |
| `availableQuantity` | INTEGER | Units currently in stock |
| `discountedSellingPrice` | NUMERIC(5,2) | Final price after discount |
| `weightInGms` | INTEGER | Product weight in grams |
| `outOfStock` | BOOLEAN | Whether the product is currently unavailable |
| `quantity` | INTEGER | Pack quantity |

### Categories

Fruits & Vegetables · Cooking Essentials · Munchies · Dairy, Bread & Batter · Beverages · Packaged Food · Ice Cream & Desserts · Chocolates & Candies · Meats, Fish & Eggs · Biscuits · Personal Care · Paan Corner · Home & Cleaning · Health & Hygiene

---

## Project Structure

```
zepto-sql-analysis/
│
├── zepto_v2.csv          # Raw dataset
└── analysis.sql          # All SQL queries (schema → cleaning → analysis)
```

---

## How to Run

1. **Set up PostgreSQL** and create a new database.
2. **Create the table** using the `CREATE TABLE` statement in `analysis.sql`.
3. **Load the data** using `\COPY` or your preferred import method:
   ```sql
   \COPY zepto(category, name, mrp, discountPercent, availableQuantity,
               discountedSellingPrice, weightInGms, outOfStock, quantity)
   FROM 'zepto_v2.csv' DELIMITER ',' CSV HEADER;
   ```
4. **Run the queries** in sequence — cleaning steps must come before analysis.

---

## Workflow

### 1. Schema Definition

```sql
DROP TABLE IF EXISTS zepto;
CREATE TABLE zepto (
    sku_id               SERIAL PRIMARY KEY,
    category             VARCHAR(120),
    name                 VARCHAR(150) NOT NULL,
    mrp                  NUMERIC(8,2),
    discountPercent      NUMERIC(5,2),
    availableQuantity    INTEGER,
    discountedSellingPrice NUMERIC(8,2),
    weightInGms          INTEGER,
    outOfStock           BOOLEAN,
    quantity             INTEGER
);
```

---

### 2. Data Exploration

| Query | Purpose |
|---|---|
| `COUNT(*)` | Total row count |
| `SELECT * LIMIT 10` | Quick sample |
| `WHERE ... IS NULL` | Null value check across all columns |
| `DISTINCT category` | List all product categories |
| `GROUP BY outOfStock` | In-stock vs. out-of-stock split |
| `HAVING COUNT > 1` | Products with multiple SKUs |

---

### 3. Data Cleaning

**Remove zero-price records**
Products where `mrp = 0` are invalid and removed.
```sql
DELETE FROM zepto WHERE mrp = 0;
```

**Convert paise to rupees**
Raw price values were stored in paise (×100). Converted to ₹ by dividing by 100.
```sql
UPDATE zepto
SET mrp = mrp / 100.0,
    discountedSellingPrice = discountedSellingPrice / 100.0;
```

---

### 4. Analysis Queries

#### Q1 — Top 10 Best-Value Products by Discount
Ranks distinct products by `discountPercent` descending to find the highest markdown deals.

#### Q2 — High-MRP Products That Are Out of Stock
Filters for `outOfStock = TRUE` and `mrp > ₹300`, revealing missed revenue opportunities on premium items.

#### Q3 — Estimated Revenue per Category
Calculates `SUM(discountedSellingPrice × availableQuantity)` as a proxy for potential category revenue.

#### Q4 — Premium Products with Low Discounts
Products priced above ₹500 with less than 10% discount — useful for targeted promotional planning.

#### Q5 — Top 5 Categories by Average Discount
Ranks categories by `AVG(discountPercent)` to understand where Zepto positions its pricing competitiveness.

#### Q6 — Price per Gram (Best Value by Weight)
For products ≥ 100g, computes `discountedSellingPrice / weightInGms` to surface the most economical options by unit weight.

#### Q7 — Weight-Based Segmentation
Classifies each product into:
- **Low** — under 1,000g
- **Medium** — 1,000g to 4,999g
- **Bulk** — 5,000g and above

#### Q8 — Total Inventory Weight per Category
Computes `SUM(weightInGms × availableQuantity)` per category for logistics and warehousing insights.

---

## Key Findings

- **Prices were stored in paise** in the raw dataset and required conversion — always validate numeric units on ingestion.
- **Multiple SKUs per product name** exist, reflecting different pack sizes or variants.
- **Out-of-stock premium items** represent a notable revenue gap worth restocking prioritisation.
- **Category-level discounts** vary significantly, indicating non-uniform promotional strategies across verticals.
- **Price-per-gram analysis** surfaces counterintuitive value — heavier/bulk items are not always cheaper per gram.

---

## Requirements

- PostgreSQL 13+
- Any SQL client (psql, DBeaver, TablePlus, pgAdmin)

---

## Author

Personal SQL portfolio project — data sourced from the Zepto platform.
