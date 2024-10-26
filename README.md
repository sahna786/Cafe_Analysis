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
