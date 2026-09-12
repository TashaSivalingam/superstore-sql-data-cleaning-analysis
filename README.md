# superstore-sql-data-cleaning-analysis
DataCamp Portfolio Project: Intermediate SQL analysis on Super Store sales dataset using CTEs, window functions (ROW_NUMBER), and conditional logic (CASE WHEN) to impute missing inventory quantities and identify top-performing products by category.
# Super Store SQL Data Cleaning & Product Performance Analysis

## Project Overview
This project analyzes sales and product data from a hypothetical **Super Store** to solve real-world data quality issues and extract strategic product insights. The primary focus is two-fold: performing advanced data cleaning to impute missing quantity values based on historical unit pricing metrics, and evaluating product performance by identifying the top 5 revenue-generating products across each business category.

---

## Tools & SQL Concepts Applied
* **Database Engine:** PostgreSQL
* **SQL Concepts & Techniques:**
  * **Window Functions (`ROW_NUMBER()`):** Partitioning datasets by category to extract top-performing product subsets.
  * **Common Table Expressions (CTEs):** Structuring multi-step data pipelines for price estimation and rank calculation.
  * **Data Cleaning & Imputation (`CASE WHEN`):** Estimating missing order quantities using calculated unit prices derived from sales.
  * **Subqueries & Aggregations (`GROUP BY`, `SUM()`):** Grouping transactional data to calculate totals and unit margins across product IDs, markets, and regions.

---

## Key Business Questions & SQL Solutions

### 1. Top 5 Products per Category by Sales
**Business Question:** Find the top 5 products within each category based on the highest total sales. The output is sorted by category in ascending order and by sales in descending order within each category.

**SQL Solution:**
```sql
-- Top 5 products from each category based on highest total sales--
WITH top_product AS(
SELECT 
	  category,
    product_name,
    ROUND(SUM(sales)::NUMERIC,2) AS product_total_sales, 
    ROUND(SUM(profit)::NUMERIC,2) AS product_total_profit,
	  ROW_NUMBER()OVER(PARTITION BY category ORDER BY ROUND(SUM(sales)::NUMERIC,2) DESC) AS product_rank
FROM orders
LEFT JOIN products
	USING (product_id)
GROUP BY category,product_name)

SELECT category,product_name,product_total_sales, product_total_profit, product_rank
FROM top_product
WHERE product_rank BETWEEN 1 AND 5
ORDER BY category, product_total_sales DESC;
```
<img width="962" height="286" alt="image" src="https://github.com/user-attachments/assets/c73c3819-5dee-4f4a-b51a-d5ec72a3ace5" />


---

### 2. Imputing Missing Quantities via Estimated Unit Price
**Business Question:** Calculate missing values in the `quantity` column by determining the effective unit price for each `product_id` using available order data  and estimate the missing quantities under `calculated_quantity`.

**SQL Solution:**
```sql
--Quantity for orders with missing values in the quantity column: 
-- Impute_missing_values--
SELECT product_id, quantity, sales
FROM orders
WHERE quantity IS NULL;

--To check unit price of missing products--
SELECT product_id, quantity, sales
FROM orders
WHERE product_id IN ('TEC-STA-10003330','FUR-ADV-10000571','FUR-BO-10001337','TEC-STA-10004542','FUR-ADV-10004395')
AND quantity IS NOT NULL
	ORDER BY product_id,quantity, sales;


--Use unit price to estimate the missing quantity values--
SELECT product_id, discount, market,region, quantity, sales,
	   CASE 
	     WHEN product_id= 'FUR-ADV-10000571' THEN sales/109.74
         WHEN product_id= 'FUR-ADV-10004395' THEN sales/16.824
         WHEN product_id= 'FUR-BO-10001337' THEN sales/102.833
         WHEN product_id= 'TEC-STA-10003330' THEN sales/101.328
         WHEN product_id= 'TEC-STA-10004542' THEN sales/16.032
         END AS calculated_quantity
FROM orders
	WHERE quantity IS NULL;

```
<img width="957" height="172" alt="image" src="https://github.com/user-attachments/assets/7d627b43-278f-45d9-953d-a0dfa492acf4" />

---

## 💡 Key Business Takeaways
1. **Data Integrity Restored:** By leveraging historical `sales` data at the `product_id` level, missing order quantities were accurately backfilled without skewing overall inventory reports.
2. **Category Performance:** Segmenting top products using `ROW_NUMBER()OVER(PARTITION BY)` highlighted key revenue drivers per category, allowing marketing teams to focus promotional budgets on high-margin inventory.
