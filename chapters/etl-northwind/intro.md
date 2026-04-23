# ETL — Northwind Example

The Northwind database is a classic teaching schema — a small, realistic OLTP system for a specialty food importer / exporter. In this chapter we take the normalized Northwind OLTP schema and walk through a full ETL to a star-schema data warehouse suitable for sales analysis.

## The Northwind OLTP schema

Core tables and their roles:

| Table | Purpose |
|---|---|
| `Customers` | Companies that place orders |
| `Employees` | Sales reps who take orders |
| `Suppliers` | Companies supplying products |
| `Products` | Items sold; FK to `Categories` and `Suppliers` |
| `Categories` | Product categories |
| `Orders` | Order header — one row per order, FKs to `Customer`, `Employee`, `Shipper` |
| `Order Details` | Order lines — quantity, unit price, discount per product per order |
| `Shippers` | Shipping companies |
| `Territories` / `Region` | Sales territory hierarchy |

It's normalized to about 3NF — great for transactional integrity, awkward for analytics.

## Target star schema

For a sales analytics warehouse we design around the `Order Details` grain.

**Grain:** one row per product line on one order.

**Fact table — `fact_sales`:**
- FKs: `order_date_sk`, `required_date_sk`, `shipped_date_sk` (role-playing dates), `customer_sk`, `employee_sk`, `product_sk`, `shipper_sk`
- Degenerate: `order_id`, `order_line_no`
- Measures: `quantity`, `unit_price`, `discount_pct`, `extended_price = quantity * unit_price * (1 - discount_pct)`, `freight_allocated`

**Dimension tables:**
- `dim_date` — pre-populated calendar (role-plays for order / required / shipped)
- `dim_customer` — `Customers` enriched with territory, SCD Type 2 on address and contact
- `dim_employee` — `Employees` joined to `Territories` → `Region`; SCD Type 2 on title and region assignment
- `dim_product` — `Products` joined to `Categories` and `Suppliers`; SCD Type 1 on price, Type 2 on category
- `dim_shipper` — `Shippers`; Type 1

## ETL pipeline, step by step

### 1. Extract to staging

Pull full copies of each source table into a staging schema (`stg_northwind`). For incremental loads in a real system you'd use `ModifiedDate` watermarks; the Northwind teaching dataset is small enough for full extracts each run.

```sql
-- pseudocode, one per source table
INSERT INTO stg_customers
SELECT * FROM source.Customers;
```

### 2. Build `dim_date`

Pre-populate 20+ years of calendar rows with attributes like `year`, `quarter`, `month`, `month_name`, `day_of_week`, `day_name`, `is_weekend`, `is_holiday`, `fiscal_year`, `fiscal_quarter`.

### 3. Build `dim_customer` (SCD Type 2)

```sql
MERGE INTO dim_customer tgt
USING stg_customers src
  ON tgt.customer_nk = src.CustomerID AND tgt.is_current = TRUE
WHEN MATCHED AND (tgt.address <> src.Address OR tgt.city <> src.City
                  OR tgt.country <> src.Country)
  THEN UPDATE SET tgt.is_current = FALSE,
                  tgt.effective_to = CURRENT_DATE
WHEN NOT MATCHED BY TARGET
  THEN INSERT (customer_sk, customer_nk, company_name, contact, address, city, country,
               effective_from, effective_to, is_current)
       VALUES (NEXT VALUE FOR customer_sk_seq, src.CustomerID, src.CompanyName,
               src.ContactName, src.Address, src.City, src.Country,
               CURRENT_DATE, NULL, TRUE);

-- Add new Type 2 rows for the customers whose attributes changed above
INSERT INTO dim_customer (...)
SELECT ... FROM stg_customers src
WHERE EXISTS (/* attribute changed */);
```

### 4. Build `dim_product` (mixed SCD)

- Denormalize by joining `Products → Categories → Suppliers`
- Type 1 on `unit_price` (always carry the latest)
- Type 2 on `category_name` and `supplier_name` so historical sales keep their original classification

### 5. Build `dim_employee`

Join `Employees` to `EmployeeTerritories`, `Territories`, and `Region` to flatten the territory hierarchy. Apply SCD Type 2 on `title` and `region`.

### 6. Build `fact_sales`

```sql
INSERT INTO fact_sales (
    order_date_sk, required_date_sk, shipped_date_sk,
    customer_sk, employee_sk, product_sk, shipper_sk,
    order_id, order_line_no,
    quantity, unit_price, discount_pct, extended_price, freight_allocated
)
SELECT
    d1.date_sk  AS order_date_sk,
    d2.date_sk  AS required_date_sk,
    d3.date_sk  AS shipped_date_sk,
    c.customer_sk,
    e.employee_sk,
    p.product_sk,
    sh.shipper_sk,
    o.OrderID,
    od.line_no,
    od.Quantity,
    od.UnitPrice,
    od.Discount,
    od.Quantity * od.UnitPrice * (1 - od.Discount) AS extended_price,
    o.Freight * od.Quantity / SUM(od.Quantity) OVER (PARTITION BY od.OrderID) AS freight_allocated
FROM stg_order_details od
JOIN stg_orders   o  ON od.OrderID = o.OrderID
JOIN dim_date     d1 ON d1.date = o.OrderDate
LEFT JOIN dim_date d2 ON d2.date = o.RequiredDate
LEFT JOIN dim_date d3 ON d3.date = o.ShippedDate
JOIN dim_customer c   ON c.customer_nk = o.CustomerID   AND c.is_current = TRUE
JOIN dim_employee e   ON e.employee_nk = o.EmployeeID   AND e.is_current = TRUE
JOIN dim_product  p   ON p.product_nk  = od.ProductID   AND p.is_current = TRUE
LEFT JOIN dim_shipper sh ON sh.shipper_nk = o.ShipVia;
```

Note the `is_current = TRUE` filters on Type 2 dimensions — we snap to the *current* dimension row on first load. For a historical backload you'd instead match on `effective_from <= o.OrderDate < effective_to`.

### 7. Validation

After loading, reconcile against the source:

```sql
-- Row count reconciliation
SELECT (SELECT COUNT(*) FROM source.[Order Details]) AS src_count,
       (SELECT COUNT(*) FROM fact_sales)             AS tgt_count;

-- Revenue reconciliation
SELECT SUM(Quantity * UnitPrice * (1 - Discount))    AS src_revenue,
       (SELECT SUM(extended_price) FROM fact_sales)  AS tgt_revenue;
```

Any mismatch is a signal: a dropped row (failed lookup), a duplicated row (bad join), a dimension key miss (SCD logic bug), or an arithmetic drift (rounding / data type).

## What you practice in this chapter

- Reading a normalized OLTP schema and finding the grain
- Designing the target star schema
- Writing the actual `MERGE` / `INSERT` statements
- Applying SCD rules
- Validating the load

## Learning outcomes

- Map a normalized OLTP schema to a Kimball star schema
- Implement `dim_date` and pre-populate it
- Write SCD Type 1 and Type 2 `MERGE` logic
- Build a fact table with role-playing date dimensions and allocated measures
- Run reconciliation queries to validate a load

