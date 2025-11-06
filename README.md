# 🧠 Data Warehouse Project — Exploratory Data Analysis (EDA)

## 📘 Description

This SQL script performs **Exploratory Data Analysis (EDA)** on the **Gold Layer** of the Data Warehouse built using **SQL Server** and **Medallion Architecture** (Bronze → Silver → Gold).  

The purpose is to:
- Validate data integrity and consistency.
- Explore schema metadata (tables, columns).
- Generate high-level business insights.
- Analyze sales, products, customers, and geography dimensions.

---

## 🗺️ Integration Model

The diagram below illustrates the **data integration flow** across the three layers (Bronze → Silver → Gold):

![Integration Model](./docs/integration_model.drawio.png)

---

# 🔍 Exploratory Data Analysis — SQL Script

## 🧾 1. SCHEMA EXPLORATION
```sql

-- Check how many tables we have and their types
SELECT * FROM INFORMATION_SCHEMA.TABLES;

-- Explore columns of a specific table
SELECT * FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'dim_products';

```
## 🌍 2. GEOGRAPHIC EXPLORATION

```sql
-- Explore all countries where customers are from
SELECT DISTINCT country
FROM gold.dim_customers;

```
## 🏷️ 3. DIMENSION EXPLORATION
  ```sql
-- Explore product categories, subcategories, and names
SELECT DISTINCT category, sub_category, product_name
FROM gold.dim_products;

```
##  ⏳ 4. DATE EXPLORATION
```sql

-- Find earliest and latest order dates
SELECT MIN(order_date) AS first_order_date,  -- First order date
       MAX(order_date) AS last_order_date,   -- Last order date  
       DATEDIFF(YEAR, MIN(order_date), MAX(order_date)) AS ord_range_years
FROM gold.fact_sales;

-- Find oldest and youngest customers
SELECT MIN(birthdate) AS oldest_cust,
       MAX(birthdate) AS youngest_cust
FROM gold.dim_customers;

```
## 💰 5. BUSINESS METRICS
```sql

-- Total Sales
SELECT SUM(sales_amount) AS total_sales 
FROM gold.fact_sales;

-- Total Quantity Sold
SELECT SUM(quantity) AS total_sales_quantity
FROM gold.fact_sales;

-- Average Selling Price
SELECT AVG(price) AS avg_sales_price
FROM gold.fact_sales;

-- Total Orders
SELECT COUNT(DISTINCT order_number) AS num_of_orders 
FROM gold.fact_sales;

-- Total Products
SELECT COUNT(product_key) AS num_of_products 
FROM gold.dim_products;

-- Total Customers
SELECT COUNT(DISTINCT customer_id) AS num_of_customers 
FROM gold.dim_customers;

-- Customers Who Placed Orders
SELECT COUNT(DISTINCT customer_key) AS num_of_customers
FROM gold.fact_sales;
```

##    📊 6. BUSINESS PERFORMANCE DASHBOARD (KEY METRICS)
```sql

SELECT 'Total Sales' AS measure_name, SUM(sales_amount) AS measure_value FROM gold.fact_sales
UNION ALL
SELECT 'Total Quantity' AS measure_name, SUM(quantity) AS measure_value FROM gold.fact_sales
UNION ALL
SELECT 'Average Price' AS measure_name, AVG(price) AS avg_sales_price FROM gold.fact_sales
UNION ALL 
SELECT 'Total nr. orders' AS measure_name, COUNT(DISTINCT order_number) AS num_of_orders FROM gold.fact_sales
UNION ALL
SELECT 'Total nr. products' AS measure_name, COUNT(product_key) AS num_of_products FROM gold.dim_products
UNION ALL
SELECT 'Total nr. customers' AS measure_name, COUNT(DISTINCT customer_id) AS num_of_customers FROM gold.dim_customers;

```
## 🌎 7. MAGNITUDE ANALYSIS
```sql
-- Customers by Country
SELECT country,
       COUNT(customer_key) AS num_of_customers
FROM gold.dim_customers
GROUP BY country
ORDER BY num_of_customers DESC;

-- Customers by Gender
SELECT gender,
       COUNT(customer_key) AS num_of_customers
FROM gold.dim_customers
GROUP BY gender
ORDER BY num_of_customers DESC;

-- Products by Category
SELECT category,
       COUNT(product_key) AS num_of_prod
FROM gold.dim_products
GROUP BY category
ORDER BY num_of_prod DESC;

-- Average Cost per Category
SELECT category,
       AVG(cost) AS avg_cost
FROM gold.dim_products
GROUP BY category
ORDER BY avg_cost DESC;

-- Total Revenue per Category
SELECT pr.category,
       SUM(sales_amount) AS total_rev 
FROM gold.fact_sales AS sl
LEFT JOIN gold.dim_products AS pr
       ON pr.product_key = sl.product_key
GROUP BY pr.category
ORDER BY total_rev DESC;

```
## 👥 8. CUSTOMER REVENUE ANALYSIS
```sql
-- Total Revenue by Customer
SELECT cs.customer_key,
       cs.first_name,
       cs.last_name,
       SUM(sl.sales_amount) AS total_sales
FROM gold.fact_sales AS sl
LEFT JOIN gold.dim_customers AS cs
       ON cs.customer_key = sl.customer_key
GROUP BY cs.customer_key,
         cs.first_name,
         cs.last_name
ORDER BY total_sales DESC;

-- Total Revenue by Country
SELECT cs.country,
       SUM(sl.sales_amount) AS total_sales
FROM gold.fact_sales AS sl
LEFT JOIN gold.dim_customers AS cs
       ON cs.customer_key = sl.customer_key
GROUP BY cs.country
ORDER BY total_sales DESC;
```
## 📦 9. SALES DISTRIBUTION
```sql
-- Distribution of Sold Items Across Countries
SELECT cs.country,
       SUM(sl.quantity) AS sold_items_quantity
FROM gold.fact_sales AS sl
LEFT JOIN gold.dim_customers AS cs
       ON cs.customer_key = sl.customer_key
GROUP BY cs.country
ORDER BY sold_items_quantity DESC;
```

## 🏆 10. PRODUCT REVENUE RANKING
```sql
-- Top 5 Highest Revenue Products
SELECT TOP 5 
       pr.sub_category,
       SUM(sales_amount) AS total_rev   
FROM gold.fact_sales AS sl
LEFT JOIN gold.dim_products AS pr
       ON pr.product_key = sl.product_key
GROUP BY pr.sub_category
ORDER BY total_rev DESC;

-- 5 Lowest Revenue Products
SELECT TOP 5 
       pr.product_name,
       SUM(sales_amount) AS total_rev   
FROM gold.fact_sales AS sl
LEFT JOIN gold.dim_products AS pr
       ON pr.product_key = sl.product_key
GROUP BY pr.product_name
ORDER BY total_rev ASC;

```
## 🪜 11. PRODUCT RANKING (WINDOW FUNCTION)
```sql
SELECT 
    product_name,
    total_rev
FROM (
    SELECT 
           pr.product_name,
           SUM(sales_amount) AS total_rev,
           ROW_NUMBER() OVER (ORDER BY SUM(sales_amount) DESC) AS rank_products
    FROM gold.fact_sales AS sl
    LEFT JOIN gold.dim_products AS pr
           ON pr.product_key = sl.product_key
    GROUP BY pr.product_name
) t
WHERE rank_products <= 5;
```

## 💎 12. CUSTOMER RANKING (TOP CUSTOMERS)
```sql
-- Top 10 Customers by Revenue
SELECT TOP 10 
       cs.first_name,
       cs.last_name,
       SUM(sl.sales_amount) AS total_sales
FROM gold.fact_sales AS sl
LEFT JOIN gold.dim_customers AS cs
       ON cs.customer_key = sl.customer_key
GROUP BY cs.first_name,
         cs.last_name
ORDER BY total_sales DESC;

-- Top 5 Customers using Window Function
SELECT
    first_name,
    total_sales,
    order_sales
FROM (
    SELECT  
           cs.first_name,
           cs.last_name,
           SUM(sl.sales_amount) AS total_sales,
           ROW_NUMBER() OVER(ORDER BY SUM(sl.sales_amount) DESC) AS order_sales
    FROM gold.fact_sales AS sl
    LEFT JOIN gold.dim_customers AS cs
           ON cs.customer_key = sl.customer_key
    GROUP BY cs.first_name,
             cs.last_name
) t
WHERE order_sales <= 5;
```
## 🧾 13. PRODUCT REPORT

This report consolidates **key product metrics and behaviors** from the Gold layer.

### 🎯 Purpose
- Provide detailed insights into product performance and sales behavior.  
- Identify **High-Performer**, **Mid-Range**, and **Low-Performer** products.  
- Enable analysis of product lifecycle and revenue contribution.

### 📊 Highlights
1. Gathers essential fields such as product name, category, subcategory, and cost.  
2. Segments products by revenue to identify performance tiers.  
3. Aggregates product-level metrics:
   - Total orders  
   - Total sales  
   - Total quantity sold  
   - Total unique customers  
   - Product lifespan (in months)  
4. Calculates KPIs:
   - **Recency** (months since last sale)  
   - **Average Order Revenue (AOR)**  
   - **Average Monthly Revenue**

```sql
/*
===============================================================================
Product Report
===============================================================================
Purpose:
    - This report consolidates key product metrics and behaviors.

Highlights:
    1. Gathers essential fields such as product name, category, subcategory, and cost.
    2. Segments products by revenue to identify High-Performers, Mid-Range, or Low-Performers.
    3. Aggregates product-level metrics:
       - total orders
       - total sales
       - total quantity sold
       - total customers (unique)
       - lifespan (in months)
    4. Calculates valuable KPIs:
       - recency (months since last sale)
       - average order revenue (AOR)
       - average monthly revenue
===============================================================================
*/
CREATE VIEW gold.report_products AS 

WITH cte_base_info AS (
    /*---------------------------------------------------------------------------
    1) Base Query: Retrieves core columns from tables
    ---------------------------------------------------------------------------*/
    SELECT
        f.order_number,
        f.order_date,
        f.customer_key,
        f.sales_amount,
        f.quantity,
        p.product_key,
        p.product_name,
        p.category,
        p.sub_category,
        p.cost
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_products p
        ON f.product_key = p.product_key
    WHERE order_date IS NOT NULL 
),
```
```sql
/*--------------------------------------------------------------------------------------------
  2. Segments products by revenue to identify High-Performers, Mid-Range, or Low-Performers.
---------------------------------------------------------------------------------------------*/
cte_product_aggregations AS (
SELECT
    product_key,
    product_name,
    category,
    sub_category,
    cost,
    DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan,
    MAX(order_date) AS last_sale_date,
    COUNT(DISTINCT order_number) AS total_orders,
    COUNT(DISTINCT customer_key) AS total_customers,
    SUM(sales_amount) AS total_sales,
    SUM(quantity) AS total_quantity,
    ROUND(AVG(CAST(sales_amount AS FLOAT) / NULLIF(quantity, 0)),1) AS avg_selling_price
FROM cte_base_info
GROUP BY product_key,
        product_name,
        category,
        sub_category,
        cost
)
```
```sql
/*---------------------------------------------------------------------------
  3) Final Query: Combines all product results into one output
---------------------------------------------------------------------------*/
SELECT 
	product_key,
	product_name,
	category,
	sub_category,
	cost,
	last_sale_date,
	DATEDIFF(MONTH, last_sale_date, GETDATE()) AS recency_in_months,
	CASE
		WHEN total_sales > 50000 THEN 'High-Performer'
		WHEN total_sales >= 10000 THEN 'Mid-Range'
		ELSE 'Low-Performer'
	END AS product_segment,
	lifespan,
	total_orders,
	total_sales,
	total_quantity,
	total_customers,
	avg_selling_price,
	-- Average Order Revenue (AOR)
	CASE 
		WHEN total_orders = 0 THEN 0
		ELSE total_sales / total_orders
	END AS avg_order_revenue,

	-- Average Monthly Revenue
	CASE
		WHEN lifespan = 0 THEN total_sales
		ELSE total_sales / lifespan
	END AS avg_monthly_revenue

FROM cte_product_aggregations;
```


## 👥 14. CUSTOMER REPORT

This report consolidates key customer metrics and behaviors from the Gold layer.

### 🎯 Purpose

- Provide a unified view of customer transactions and engagement.

- Segment customers into meaningful groups for deeper insight.

- Track customer value and lifecycle metrics.

### 📊 Highlights

1. Gathers essential fields such as names, ages, and transaction details.

2. Segments customers into categories (VIP, Regular, New) and age groups.

3. Aggregates customer-level metrics:

      - Total orders
    
      - Total sales
      
      - Total quantity purchased
      
      - Total unique products
      
      - Customer lifespan (in months)
  
  4. Calculates KPIs:
  
     - Recency (months since last order)
    
     - Average Order Value (AOV)
    
     - Average Monthly Spending
```sql
/*
===============================================================================
Customer Report
===============================================================================
Purpose:
    - This report consolidates key customer metrics and behaviors

Highlights:
    1. Gathers essential fields such as names, ages, and transaction details.
	2. Segments customers into categories (VIP, Regular, New) and age groups.
    3. Aggregates customer-level metrics:
	   - total orders
	   - total sales
	   - total quantity purchased
	   - total products
	   - lifespan (in months)
    4. Calculates valuable KPIs:
	    - recency (months since last order)
		- average order value
		- average monthly spend
===============================================================================
*/

CREATE VIEW gold.report_customers AS
WITH cte_base_details AS (
/*---------------------------------------------------------------------------
1) Base Query: Retrieves core columns from tables
---------------------------------------------------------------------------*/
SELECT 
       s.order_number,
	   s.product_key,
	   s.order_date,
	   s.sales_amount,
	   s.quantity,
	   c.customer_key,
	   c.customer_number,
	   CONCAT(c.first_name,' ', c.last_name) as customer_name,
	   DATEDIFF(YEAR, c.birthdate, GETDATE()) as customer_age
FROM gold.fact_sales as s
LEFT JOIN gold.dim_customers as c
      ON  s.customer_key = c.customer_key
WHERE order_date is not null
), cte_customer_aggregation as (
/*---------------------------------------------------------------------------
2) Customer Aggregations: Summarizes key metrics at the customer level
---------------------------------------------------------------------------*/
```
```sql
SELECT customer_key,
	   customer_number,
	   customer_name,
	   customer_age,
	   COUNT(DISTINCT order_number) as total_order_num,
	   SUM(sales_amount) as total_sales,
	   SUM(quantity) as total_quantity,
	   COUNT(DISTINCT product_key) as total_product,
	   MAX(order_date) as last_order,
	   DATEDIFF(month, MIN(order_date), MAX(order_date)) as lifespan
FROM cte_base_details
GROUP BY customer_key,
	     customer_number,
	     customer_name,
	     customer_age
)
SELECT   customer_key,
	     customer_number,
	     customer_name,
	     customer_age,
		 CASE WHEN  customer_age < 20 THEN 'Under 20'
		      WHEN  customer_age BETWEEN 20 AND 29 THEN '20-29'
			  WHEN  customer_age BETWEEN 30 AND 39 THEN '30-39'
			  WHEN  customer_age BETWEEN 40 AND 49 THEN '40-49'
			  ELSE  '50 and Above'
         END AS age_segmentaion,
		 CASE WHEN  lifespan >= 12 and total_sales > 5000  THEN 'VIP'
              WHEN  lifespan >= 12 and total_sales <= 5000 THEN 'Regular'
              ELSE  'New'
         END as customer_segmentation,
		 last_order,
		 DATEDIFF(MONTH, last_order, GETDATE()) as recency_in_month,
		 total_order_num,
	     total_sales,
	     total_quantity,
	     total_product,
	     lifespan,
		 -- KPI Calculations
		 -- Average order value:
		 CASE WHEN total_sales = 0 THEN 0
		      ELSE total_sales / total_order_num
		 END AS  avg_order_value,
		 -- Average Monthly Spending:
		 CASE WHEN lifespan = 0  THEN total_sales
		      ELSE total_sales / lifespan
		 END AS avg_monthly_spending
FROM cte_customer_aggregation;
```
