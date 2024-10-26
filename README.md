# Coffee Shop Sales Analysis using SQL and Power BI
  
### 1. Objective:
- Analyze sales data from a coffee shop to gain insights into sales performance, product popularity, store performance, and temporal trends.
- Use SQL for data querying and Power BI for visualization.
- Tools Used: SQL, Power BI, Excel for initial data storage.
- Scope: Data includes information about transactions, products, stores, and time-based attributes.
### 2. Data Collection & Import
- Source of Data:
The sales data is stored in an Excel file with fields like transaction_id, transaction_date, transaction_time, store_id, store_location, product_id, unit_price, product_category, product_type, product_detail, and transaction_qty.
- Data Import:
The Excel file is imported into both SQL and Power BI for further processing and analysis.
### 3. Data Cleaning & Transformation
- Date and Time Conversion: Convert transaction_date and transaction_time fields from string to DATE and TIME types:

  ```sql
  UPDATE coffee_shop_sales 
  SET transaction_date = STR_TO_DATE(transaction_date, '%d/%m/%Y');
  
  ALTER TABLE coffee_shop_sales
  MODIFY COLUMN transaction_date DATE;
  
  UPDATE coffee_shop_sales 
  SET transaction_time = STR_TO_DATE(transaction_time, '%H:%i:%s');
  
  ALTER TABLE coffee_shop_sales
  MODIFY COLUMN transaction_time TIME;
  ```


- Column Renaming: Fixing column names and types:
  ```sql
  ALTER TABLE coffee_shop_sales
  CHANGE COLUMN `ï»¿transaction_id` transaction_id INT;
  ```

- Total Sales for each respective month (Eg: for May)
  ```sql
  SELECT CONCAT(ROUND(SUM(unit_price * transaction_qty)/1000), 'K') AS total_sales 
  FROM coffee_shop_sales
  WHERE MONTH(transaction_date) = 5;
  ```

- calculate the total sales for each month and the month-over-month (MoM) percentage change in sales
  ```sql
  SELECT 
    MONTH(transaction_date) AS current_month, 
    SUM(unit_price * transaction_qty) AS total_sales,
    
    -- Month-over-month percentage change in sales
    (SUM(unit_price * transaction_qty) - 
    LAG(SUM(unit_price * transaction_qty)) OVER (ORDER BY MONTH(transaction_date)))
    / LAG(SUM(unit_price * transaction_qty)) OVER (ORDER BY MONTH(transaction_date)) * 100 
    AS mom_change_percentage
    
  FROM coffee_shop_sales

  GROUP BY MONTH(transaction_date)
  ORDER BY MONTH(transaction_date);
  ```

- Calculate the total no: of orders for each month
  ```sql
  SELECT 
    MONTH(transaction_date) AS month, 
    COUNT(transaction_id) AS total_orders
  FROM coffee_shop_sales
  GROUP BY MONTH(transaction_date);
  ```

- Calculate the month-over-month (MoM) increase in the total number of orders and the percentage change between months
  ```sql
  WITH monthly_orders AS (
    SELECT 
        MONTH(transaction_date) AS month, 
        COUNT(transaction_id) AS total_orders
    FROM coffee_shop_sales
    GROUP BY MONTH(transaction_date)
  )
  SELECT 
      month,
      total_orders,
      total_orders - LAG(total_orders) OVER (ORDER BY month) AS mom_difference,
      (total_orders - LAG(total_orders) OVER (ORDER BY month)) / LAG(total_orders) OVER (ORDER BY month) * 100 AS mom_increase_percentage
  FROM monthly_orders
  ORDER BY month;
  ```
  
- Calculate the total quantity sold in each respective month
  ```sql
  select month(transaction_date) as month, sum(transaction_qty) as total_qty_sold
  from coffee_shop_sales
  group by month;
  ```
