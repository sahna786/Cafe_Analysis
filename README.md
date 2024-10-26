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
### Business Problems and Solutions

1. Total Sales for each respective month (Eg: for May)
  ```sql
  SELECT CONCAT(ROUND(SUM(unit_price * transaction_qty)/1000), 'K') AS total_sales 
  FROM coffee_shop_sales
  WHERE MONTH(transaction_date) = 5;
  ```

2. calculate the total sales for each month and the month-over-month (MoM) percentage change in sales
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

3. Calculate the total no: of orders for each month
  ```sql
  SELECT 
    MONTH(transaction_date) AS month, 
    COUNT(transaction_id) AS total_orders
  FROM coffee_shop_sales
  GROUP BY MONTH(transaction_date);
  ```

4. Calculate the month-over-month (MoM) increase/decrease in the total number of orders and the percentage change between months
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
      (total_orders - LAG(total_orders) OVER (ORDER BY month)) / LAG(total_orders) OVER (ORDER BY month) * 100 AS mom_change_percentage
  FROM monthly_orders
  ORDER BY month;
  ```
5. Calculate the total quantity sold in each respective month
  ```sql
  SELECT 
      MONTH(transaction_date) AS month, 
      SUM(transaction_qty) AS total_qty_sold
  FROM coffee_shop_sales
  GROUP BY MONTH(transaction_date);

  ```

6. Calculate the month-over-month (MoM) increase/decrease in the total quantity sold and the percentage change between months in total quantity sold
  ```sql
  WITH monthly_qty AS (
    SELECT 
        MONTH(transaction_date) AS month, 
        SUM(transaction_qty) AS total_qty_sold
    FROM coffee_shop_sales
    GROUP BY MONTH(transaction_date)
  )
  SELECT 
      month,
      total_qty_sold,
      total_qty_sold - LAG(total_qty_sold) OVER (ORDER BY month) AS mom_difference,
      (total_qty_sold - LAG(total_qty_sold) OVER (ORDER BY month)) / LAG(total_qty_sold) OVER (ORDER BY month) * 100 AS mom_change_percentage
  FROM monthly_qty
  ORDER BY month;
  ```

7. Calculate the total sales for weekdays and weekends for the specified month (May)
  ```sql
  SELECT 
    MONTH(transaction_date) AS month,
    CASE 
        WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'weekend'
        ELSE 'weekday'
    END AS day_type,
    CONCAT(ROUND(SUM(unit_price * transaction_qty) / 1000, 1), 'K') AS total_sales
  FROM coffee_shop_sales
  WHERE MONTH(transaction_date) = 5
  GROUP BY MONTH(transaction_date), day_type;
  ```

8. Calculate total sales by store location for a specific month (May)
  ```sql
  SELECT 
    store_location,
    CONCAT(ROUND(SUM(unit_price * transaction_qty) / 1000, 1), 'K') AS total_sales
  FROM coffee_shop_sales
  WHERE MONTH(transaction_date) = 5
  GROUP BY store_location
  ORDER BY total_sales DESC;
  ```

9. Calculate the average sales for specific month (May)
  ```sql
  SELECT 
    CONCAT(ROUND(AVG(total_sales) / 1000, 1), 'K') AS avg_sales 
  FROM (
      SELECT SUM(unit_price * transaction_qty) AS total_sales
      FROM coffee_shop_sales
      WHERE MONTH(transaction_date) = 5
      GROUP BY transaction_date
  ) a;
  ```

10. Calculate the daily total sales of a specific month (May)
  ```sql
  SELECT 
    DAY(transaction_date) AS day_of_month,
    ROUND(SUM(unit_price * transaction_qty), 1) AS total_sales
  FROM coffee_shop_sales
  WHERE MONTH(transaction_date) = 5
  GROUP BY day_of_month
  ORDER BY day_of_month;
  ```

11. Compare each day's total_sales to avg_sales and categorize them as "Above Average," "Below Average," or "Equal to Average."
  ```sql
  SELECT 
    day_of_month,
    CASE 
        WHEN total_sales > avg_sales THEN 'Above Average'
        WHEN total_sales < avg_sales THEN 'Below Average'
        ELSE 'Equal to Average' 
    END AS sales_status,
    total_sales
  FROM (
      SELECT 
          DAY(transaction_date) AS day_of_month,
          SUM(unit_price * transaction_qty) AS total_sales,
          AVG(SUM(unit_price * transaction_qty)) OVER() AS avg_sales
      FROM coffee_shop_sales
      WHERE MONTH(transaction_date) = 5
      GROUP BY transaction_date
  ) a;
  ```

12. List product category by total sales and sort them descending order
  ```sql
  SELECT 
    product_category,
    SUM(unit_price * transaction_qty) AS total_sales
  FROM coffee_shop_sales
  WHERE MONTH(transaction_date) = 5 
  GROUP BY product_category
  ORDER BY total_sales DESC;
  ```

13. Determine top 10 products by total sales
  ```sql
  SELECT 
    product_type,
    SUM(unit_price * transaction_qty) AS total_sales 
  FROM coffee_shop_sales
  WHERE MONTH(transaction_date) = 5
  GROUP BY product_type
  ORDER BY total_sales DESC
  LIMIT 10;
  ```

14. Calculate total Sales, total quantity sold, total orders for a specific day and time (Month: May, Day: Monday, Hour: 8)
  ```sql
  SELECT 
    SUM(unit_price * transaction_qty) AS total_sales,
    SUM(transaction_qty) AS total_qty_sold,
    COUNT(*) AS total_orders
  FROM coffee_shop_sales
  WHERE MONTH(transaction_date) = 5
      AND DAYOFWEEK(transaction_date) = 2
      AND HOUR(transaction_time) = 8;
  ```
15. Calculate the hourly total sales for a specific month (May)
  ```sql
  SELECT 
    HOUR(transaction_time) AS hours,
    SUM(unit_price * transaction_qty) AS total_sales
  FROM coffee_shop_sales
  WHERE MONTH(transaction_date) = 5
  GROUP BY hours
  ORDER BY hours;
  ```

16. Calculate weekly total sales for a specific month (may)
  ```sql
  SELECT 
    CASE 
        WHEN DAYOFWEEK(transaction_date) = 1 THEN 'Sunday'
        WHEN DAYOFWEEK(transaction_date) = 2 THEN 'Monday'
        WHEN DAYOFWEEK(transaction_date) = 3 THEN 'Tuesday'
        WHEN DAYOFWEEK(transaction_date) = 4 THEN 'Wednesday'
        WHEN DAYOFWEEK(transaction_date) = 5 THEN 'Thursday'
        WHEN DAYOFWEEK(transaction_date) = 6 THEN 'Friday'
        ELSE 'Saturday' 
    END AS day_of_week,
    SUM(unit_price * transaction_qty) AS total_sales
  FROM coffee_shop_sales
  WHERE MONTH(transaction_date) = 5
  GROUP BY day_of_week;
  ```
### Business Impact and Recommendations
Impact:
- The analysis provides insights into peak sales hours and high-performing products and locations, which can help in optimizing inventory, staff scheduling, and promotional efforts.

Recommendations:
- Focus marketing and promotional efforts on weekends, particularly around peak times (9AM - 12PM) when sales are the highest.
- Consider increasing inventory for popular products like Coffee and Tea to meet customer demand.
- Explore opportunities for improving sales at stores with lower performance (e.g., Lower Manhattan).
  
### Conclusion
The SQL-based data analysis combined with Power BI visualization provides actionable insights into sales trends, product preferences, and store performance. These insights can inform better decision-making for operational efficiency and profitability
  




  
  
