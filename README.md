# Mexico-Sales-Toys-Project 

# Goal of the project.
Mexico toys is the company which started toy sales business for about two years. Their owner want to know how company performs within this period in order to correct or adapt their strategy for the future.
They are interested in several analysis as follows:
-  Toys sales in terms of units sold and revenue per product, and also which product performs best.
-  Revenue, cost and profit per product and also which product performs best.
-  Revenue, cost and profit per store location(store location is a main part of mexico toys commercial strategy).
-  Revenue, cost and profit per toys category.
-  Overall revenue, cost, profit and profit percentage.
-  Overall revenue, cost, profit and profit percentage split in years and months.
-  Current inventory volume and value.

As data analyst, I have to povide the metrics that could measure those elements and provide best answers to mexico owners requests.
In order to achieve this task, I choose to use Mysql database system to pull and analyze data provided by Mexico toys. To do that, I will follow the process as presented below. 
1. Use mysql to create Mexico toys database and import csv files provided by the company.
2. Proceed with data cleaning to ensure that data are safe from errors or incorrect informations.
3. Proceed with data transformation if necessary.
4. Proceed with Data manipulation if necessary.
5. Proceed with Analysis to understand how mexico sales business is doing.
6. Provide Recommendations.

Of course mexico toys company is a fictitious company and data are provided by maven Analytics. The interested reader will find the data source in the following link (Need an account on Maven Analytics): https://app.mavenanalytics.io/datasets?search=mexi

# Step 1: Create database and import data in Mysql.
- First, create database with the following query.
```sql
   CREATE DATABASE mexicoSalestoys
```
- And then import data in mysql (4 csv files) using data table import wizard. 

There are four tables : stores, inventory, products and sales.

# Step 2:  Data cleaning and transformation
1. Check if data type in tables are in appropriate format and update or modify them if necessary.

`a. sales`
      
```sql
   SHOW FIELDS FROM sales
```
Result shows that Column 'Date' shoud be DATE type rather than TEXT. 
-  To correct it, column 'Date' should be transformed from Text type to Date type with the following query :
```sql
   ALTER table sales
   modify Date DATE
```
`b. stores`
```sql
     DESCRIBE stores
```
Result shows that Column 'Store_Open_Date' shoud be DATE type rather than TEXT type. 
-  To correct it, 'Store_Open_Date' should be transformed from Text type to Date type with the following query :
```sql
   ALTER table stores
   modify Store_Open_Date DATE
```
`c. products`
```sql
   DESCRIBE products
```
Product_cost and product_price columns should be INTEGER type or Decimal rather than TEXT. 
-  To correct it, those columns should be transformed from Text type to Decimal type. But they content '$' sign, So we need first to remove this sign, otherwise conversion into DECIMAL or INT will fail
-  Remove '$' sign by remplacing it with space with the following query.
```sql
   UPDATE  products
   SET Product_Cost = REPLACE(Product_Cost,'$',' ')
```
```sql
   UPDATE  products
   SET Product_Price = REPLACE(Product_Price,'$',' ')
```
-  And then proceed to conversion into decimal using the following query.
```sql
   ALTER TABLE products
   MODIFY Product_Price DECIMAL(10,2)
```
```sql
   ALTER TABLE products
   MODIFY Product_Cost DECIMAL(10,2)
```
-  Verify data type into table products
```sql
   SHOW FIELDS FROM products
```
2. Verify if all data in date field are consistent.  
-  Example of 'Store_Open_Date' column in stores table : Check if all data are in 'Year-month-day' to be sure that mysql will perform any task on date field. The following query will help to check.
```sql
SELECT 
   * 
FROM stores
WHERE Store_Open_Date <> DATE(STR_TO_DATE('Store_Open_Date', '%Y-%m-%d'))
```
3. Duplicates records : Verify if data recorded is unique.
-  The following query will help to know if data(column product_id) in products tables are unique:
```sql
SELECT 
      product_ID,
      COUNT(product_ID) AS ID_appearance
FROM products
GROUP BY product_ID
HAVING ID_appearance > 1
```
4. Empty Values : Verify if there is record with empty value in any column and decide how to deal with.
-  The following query will help to identify any empty values(in any column) in sales table.
```sql
SELECT  * 
FROM sales
WHERE (Sale_ID || Date || Store_ID || Product_ID || Units ) IS NULL
 ```
# Step 3:  Data exploration
1. Find the MINIMUM, MAXIMUM, Average of product price, product cost, units in stock.
-  Units in stock
```sql 
SELECT 
     MAX(Stock_On_Hand) AS Highest_stock,
     MIN(Stock_On_Hand) AS lowest_stock,
     AVG(Stock_On_Hand) AS Average_stock
FROM inventory
```
-  Product price
```sql
SELECT 
     MAX(Product_Price) AS Highest_sale_price,
     MIN(Product_Price) AS lowest_sale_price,
     AVG(Product_Price) AS Average__sale_price
FROM products
```
-  Product cost
```sql
SELECT 
     MAX(Product_Cost) AS Highest_Product_Cost,
     MIN(Product_Cost) AS lowest_Product_Cost,
     AVG(Product_Cost) AS Average__Product_Cost
FROM products
```
2. Know the product category of toys sold 
-  The following query will help to identify product category from products table.
```sql
SELECT 
DISTINCT Product_Category AS category
FROM products

-- There are only five products categories as follows ;  
-- Toys, Arts & Crafts, Games, Electronics, Sports & Outdoors.
```
3. Know the type of store location used by mexico toys company
The following query will help to identify type of store location from stores table.
```sql 
SELECT 
     DISTINCT Store_Location
FROM stores

-- There are four type stores location AS follows : 
-- Residential, Commercial, Downtown and Airtport.
```

# Step 4:  Analysis
1. Find how many sale transactions are made by mexico toys during the period.
```sql
SELECT 
     COUNT(Sale_ID) AS total_sales_transactions
FROM sales

-- There are 829262 sale transactions done by maxico sales toys
```
2. Find how many Sales Units per products have been made and higlight the first fifth products
```sql
SELECT 
     sales.Product_ID AS product_ID,
     products.Product_Name AS name,
     SUM(sales.units) AS Units_sold
FROM sales 
     INNER JOIN products 
     ON sales.product_ID = products.product_ID
GROUP BY sales.Product_ID, products.Product_Name
ORDER BY Units_sold DESC  LIMIT 5 

-- Colorbuds is the first product sold in units(104368 units) followed by PlayDoh can(103128 units), Barrel O' Slime(91663 units), Deck Of Cards(84034 units) and Magic Sand(60598 units)
```
3. Find how much Revenue per product have been made and the the first fifth revenue
```sql
SELECT 
     sales.Product_ID AS product_ID,
     products.Product_Name AS name,
     products.Product_Price,
     SUM(sales.units) AS Units_sold,
     products.Product_Price * SUM(sales.units) AS revenue_per_product
FROM sales 
     INNER JOIN products 
     ON sales.product_ID = products.product_ID
GROUP BY sales.Product_ID, products.Product_Name, products.Product_Price
ORDER BY revenue_per_product DESC  LIMIT 5  

-- Lego bricks is the first product sold in terms of revenue($2,388,882.63 followed by colorbuds, Magic sand, Action Figure, Rubiks Cube. We can see that because of its unit sale price, lego bricks drive more revenue than Colorbuds which has more units sold
```
4. Find how much money mexico Toys spent on product sold
-  The following query helps to determine Cost per product for units sold
```sql
SELECT 
     sales.Product_ID AS product_ID,
     products.Product_Name AS name,
     products.Product_Cost,
     SUM(sales.units) AS Units_sold,
     products.Product_Cost * SUM(sales.units) AS cost_per_product
FROM sales 
     INNER JOIN products 
     ON sales.product_ID = products.product_ID
GROUP BY sales.Product_ID, products.Product_Name, products.Product_Cost
ORDER BY cost_per_product DESC 

-- Lego bricks with its highest unit cost has the total highest cost($2,090,197.63) in terms of quantity sold
```
 
5. Determine how products perform each by calculating profit and profit percentage per product
- The following query helps to determine profit percentage per product
```sql
SELECT 
     sales.Product_ID AS product_ID,
     products.Product_Name AS name,
     products.Product_Cost,
     products.product_price,
     SUM(sales.units) AS Units_sold,
     SUM(sales.units) * products.Product_Cost AS cost_per_product,
     products.Product_Price * SUM(sales.units) AS revenue_per_product,
     products.Product_Price * SUM(sales.units) - products.Product_Cost * SUM(sales.units) AS profit,
     ROUND(((products.Product_Price * SUM(sales.units) - products.Product_Cost * SUM(sales.units)) / (products.Product_Price * SUM(sales.units))) * 100, 2) AS profit_percentage
FROM sales 
     INNER JOIN products 
     ON sales.product_ID = products.product_ID
GROUP BY sales.Product_ID, products.Product_Name, products.Product_Cost, products.product_price
ORDER BY profit_percentage DESC 

-- With this analysis, product Jenga is the most profitable (70.07% of profitability) product for Mexico toys. Colorbuds, magic sand and others that drive highest revenue but are not as much profitable like Jenga.
```
6. It is also important to have the overall cost, revenue, profit and profit percentage made by mexico toys

Using temporary table is the method I used to make calculations

a. create a temporary table for revenue per product with the following query
```sql
CREATE TEMPORARY TABLE productRevenue
SELECT 
     sales.Product_ID AS product_ID,
     products.Product_Name AS name,
     products.Product_Price,
     SUM(sales.units) AS Units_sold,
     products.Product_Price * SUM(sales.units) AS revenue_per_product
FROM sales 
     INNER JOIN products 
     ON sales.product_ID = d.product_ID
GROUP BY sales.Product_ID, products.Product_Name, products.Product_Price
```
  b. Create another Temporary table for cost per product with the following query
```sql
CREATE TEMPORARY TABLE productCosts
SELECT 
     sales.Product_ID AS product_ID,
     products.Product_Name AS name,
     products.Product_Cost,
     SUM(sales.units) AS Units_sold,
     products.Product_Cost * SUM(sales.units) AS cost_per_product
FROM sales 
     INNER JOIN products 
     ON sales.product_ID = products.product_ID
GROUP BY sales.Product_ID, products.Product_Name, products.Product_Cost
```
  c. THEN determine total profit and profit percentage by joining the two temporary tables and grouping revenue and costs
```sql
SELECT
     SUM(pr.revenue_per_product) AS total_revenue,
     SUM(pc.cost_per_product) AS total_cost,
     SUM(pr.revenue_per_product) - SUM(pc.cost_per_product) AS profit,
     ROUND(((SUM(pr.revenue_per_product) - SUM(pc.cost_per_product)) / SUM(pr.revenue_per_product))*100, 2)  AS profit_percentage
FROM productRevenue pr
     JOIN productCosts pc
     ON pr.product_ID = pc.product_ID

-- M toys generate Total revenue of $14444582.35, Total cost of $ 10430543.35, Total profit of $4014029 and 24.79% of total profitability
```

7. Determine Total cost, total revenue, total profit and profit percentage per type of location to see what impact store location has on mexico toys strategy.
Remember there are four types of store location (residential, airport, commercial and downtown)
-  Use a subquery as follows 
```sql
SELECT 
    Store_Location,
    SUM(revenue_per_product) AS revenue,
    SUM(cost_per_product) AS costs,
    SUM(revenue_per_product) - SUM(cost_per_product) AS profit,
    (SUM(revenue_per_product) - SUM(cost_per_product)) / SUM(revenue_per_product) AS profit_percentage
FROM
    (SELECT
         st.Store_Location,
         s.Product_ID AS product_ID,
         d.Product_Name AS name,
         d.Product_Price,
         d.Product_Cost,
         SUM(s.units) AS Units_sold,
         d.Product_Price * SUM(s.units) AS revenue_per_product,
         SUM(s.units) * d.Product_Cost  AS cost_per_product
    FROM sales s
         INNER JOIN products d
         ON s.product_ID = d.product_ID
         INNER JOIN stores st
         ON st.Store_ID = s.Store_ID
    GROUP BY st.Store_Location, s.Product_ID, d.Product_Name, d.Product_Price, d.Product_Cost
    ) AS revenue_and_costs
GROUP BY Store_Location
ORDER BY profit_percentage DESC  

-- Even if revenue is higher in stores located in downtown and commercial, profit is higher in Airport location. I think that costs in downtown could be analyzed deeply to find what can be changed to improve profit there. There is also an opportunity to find how revenue could improved in stores located in airport
```

8. Determine Total cost, total revenue, total profit and profit percentage per year and per category of toys 
- Use subquery and CASE function as follows
```sql
SELECT 
     year,
     SUM(CASE WHEN Product_Category = 'Toys' THEN revenue_per_product ELSE NULL END) AS toys_revenue,
     SUM(CASE WHEN Product_Category = 'Art & Crafts' THEN revenue_per_product ELSE NULL END) AS Art_Crafts_revenue,
     SUM(CASE WHEN Product_Category = 'Games' THEN revenue_per_product ELSE NULL END) AS Games_revenue,
     SUM(CASE WHEN Product_Category = 'Electronics' THEN revenue_per_product ELSE NULL END) AS Electronics_revenue,
     SUM(CASE WHEN Product_Category = 'Sports & Outdoors' THEN revenue_per_product ELSE NULL END) AS Sports_Outdoors_revenue
FROM
    (SELECT
          Year(s.Date) As year,
          d.Product_Category,
          s.Product_ID AS product_ID,
          d.Product_Name AS name,
          d.Product_Price,
          d.Product_Cost,
          SUM(s.units) AS Units_sold,
          d.Product_Price * SUM(s.units) AS revenue_per_product,
          SUM(s.units) * d.Product_Cost  AS cost_per_product
     FROM sales s
          INNER JOIN products d
          ON s.product_ID = d.product_ID
     GROUP BY Year(s.Date), d.Product_Category, s.Product_ID, d.Product_Name, d.Product_Price, d.Product_Cost
     ) AS revenue_and_costs
GROUP BY year

-- This analysis help us to see how revenue per category is changing year by year. Only Arts & crafts goes up between 2017 and 2018 while other goes down. Deep analysis is necessary to understand why sales of those categories dropped down. Are there any products within those categories that are responsible for this situation?
```

9. Determine Total cost, total revenue, total profit and profit percentage per year 
- Use a subquery as follows
```sql
SELECT
     year,
     SUM(revenue_per_product) AS revenue_per_year,
     SUM(cost_per_product) AS costs_per_year,
     SUM(revenue_per_product) - SUM(cost_per_product) AS profit,
     (SUM(revenue_per_product) - SUM(cost_per_product)) / SUM(revenue_per_product) AS profit_percentage
FROM
    (SELECT 
        YEAR(s.Date) AS year,
        s.Product_ID AS product_ID,
        d.Product_Name AS name,
        d.Product_Price,
        d.Product_Cost,
        SUM(s.units) AS Units_sold,
        d.Product_Price * SUM(s.units) AS revenue_per_product,
        SUM(s.units) * d.Product_Cost  AS cost_per_product
    FROM sales s 
         INNER JOIN products d ON s.product_ID = d.product_ID
    GROUP BY YEAR(s.Date), s.Product_ID, d.Product_Name, d.Product_Price, d.Product_Cost 
    ) AS revenue_and_costs
GROUP BY year


-- With this analysis we can see that even profit percentage seems to be stable from 2017 to 2018 (29.26% to 26.20%), It appears thaht revenu went down from 2017 to 2018 ($7,482,498.08 to $6,962,074.27); thios confirms what has been observed in precedent analysis.
```

10. Determine Total cost, total revenue, total profit and profit percentage per year and month
- Use subquery as follows
```sql
SELECT
     year,
     month,
     SUM(revenue_per_product) AS revenue_per_year,
     SUM(cost_per_product) AS costs_per_year,
     SUM(revenue_per_product) - SUM(cost_per_product) AS profit,
     (SUM(revenue_per_product) - SUM(cost_per_product)) / SUM(revenue_per_product) AS profit_percentage
FROM
    (SELECT
         YEAR(s.Date) AS year,
         MONTH(s.date) AS month,
         s.Product_ID AS product_ID,
         d.Product_Name AS name,
         d.Product_Price,
         d.Product_Cost,
         SUM(s.units) AS Units_sold,
         d.Product_Price * SUM(s.units) AS revenue_per_product,
         SUM(s.units) * d.Product_Cost  AS cost_per_product
     FROM sales s
          INNER JOIN products d
          ON s.product_ID = d.product_ID
     GROUP BY YEAR(s.Date), MONTH(s.date), s.Product_ID, d.Product_Name, d.Product_Price, d.Product_Cost
     ) AS revenue_and_costs
GROUP BY year, month

-- This is the precedent analysis broke down by month to see how Mexico toys perfomed month by month. Company revenue increased mont by month during 2017 with a little drop down during July and August. In 2018, revenue is constant with slightly increase, but ther is a drop down in August and september.
-- Drop down periods should be analyzed to understand what happen during those periods that can explain the situation and find solution to improve next year. 
```

11. It is also important to know the volume and value of our inventory 
- The following code will help to evaluate these metrics
```sql
SELECT
     i.Product_ID,
     p.Product_Name,
     p.Product_Cost,
     SUM(i.Stock_On_Hand),
     SUM(i.Stock_On_Hand) * p.Product_Cost AS inventory_cost_per_product
FROM inventory i
     LEFT JOIN products p
     USING(Product_ID)
GROUP BY i.Product_ID, p.Product_Name, p.Product_Cost
ORDER BY inventory_cost_per_product DESC
```

- Total inventory cost
```sql
SELECT
     SUM(inventory_cost_per_product) inv_cost
FROM
     (SELECT
         i.Product_ID,
         p.Product_Name,
         p.Product_Cost,
         SUM(i.Stock_On_Hand),
         SUM(i.Stock_On_Hand) * p.Product_Cost AS inventory_cost_per_product
     FROM inventory i
         LEFT JOIN products p
         USING(Product_ID)
     GROUP BY i.Product_ID, p.Product_Name, p.Product_Cost
     ORDER BY inventory_cost_per_product DESC
     ) AS inv_cost

-- Total inventory costs is $300,209.58 mostly drived by lego bricks with its higher unit cost and volume. Monitor these inventory by findind the less sale or less profit product could be a great opportunity to reduce this cost and also increase profitability.
```

12. Recommendations

      
      1. With this analysis, the product named Jenga is the most profitable product (70.07% of profitability) for Mexico toys. Colorbuds, magic sand and few others drive highest revenue but are not as much as profitable like Jenga. 
      
         - Action should be taken to ensure that jenga will not run out of stock.
         - Another action should be taken to revise cost chain of the most sold products like colorbuds or magic sand in order to increase their profitability.


      2. Mexico toys generates a total revenue of $14444582.35, a total cost of $ 10430543.35, a total profit of $4014029 and 24.79% of total profitability. Even if revenue is higher in stores located in downtown and commercial, profit is higher in Airport locations.
      
         - Costs in downtown locations could be analyzed deeply to find what can be changed to improve profit there. There is also an opportunity to find how revenue could be improved in stores located in airports by increasing sales volumes there.


      3. This analysis helps us to see how revenue per category is changing year by year. Only the category "Arts & crafts" goes up between 2017 and 2018 while others go down. 
   A deep analysis is necessary to understand why sales of those categories dropped down. 
   
         - Are there any products within those categories that are responsible for this situation?


      4. The analysis broke down by month reveals how Mexico toys perfomed month by month. Company revenue increased month by month during 2017 with a little drop down during July and August. In 2018, revenue is constant with slightly increase, but there is a drop down in August and september.
      
         - Drop down periods (is there any particular event?) should be analyzed to understand what happen during those periods that can explain the situation and find solution to improve next.


      5. The total inventory costs is $300,209.58 mostly driven by product lego bricks with its higher unit cost and volume. 
      
         - Monitor these inventory by findind the less sale or less profit product could be a great opportunity to reduce this cost and also increase profitability.
