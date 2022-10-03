# Mexico-Sales-Toys-Projects

# Dataset source :  https://app.mavenanalytics.io/datasets?search=mexi

# Create databse in Mysql
# Step 1  : Import data in mysql (4 csv files) using data table import wizard

# Step 2 :  Data cleaning
#Check data type and upadte or modify if necessary
##table sales

SHOW FIELDS FROM sales 

## Column 'Date' shoud be DATE type rather than TEXT

## Correction 

ALTER table sales
modify Date DATE

## table stores

DESCRIBE stores

## Column 'Store_Open_Date' shoud be DATE type rather than TEXT
## Correction  

ALTER table stores
modify Store_Open_Date DATE

## table products
DESCRIBE products

## product_cost and product_price should be INT or decimal rather than TEXT
## Correction
## Those two columns content '$' sign, So we need to remove this sign, otherwise conversion into DECIMAL or INT will faill
## Remove '$' sign by remplacing it with space

UPDATE  products
SET Product_Cost = REPLACE(Product_Cost,'$',' ')

UPDATE  products
SET Product_Price = REPLACE(Product_Price,'$',' ')
## No need to care about space; conversion into decimal will remove any space

ALTER TABLE products
MODIFY Product_Price DECIMAL(10,2)

## Test 
SHOW FIELDS FROM products

## date value (Verify if all data in date field are consistent)
## Example of 'Store_Open_Date' column in stores table

SELECT * FROM stores
WHERE Store_Open_Date <> DATE(STR_TO_DATE('Store_Open_Date', '%Y-%m-%d'))

## Duplicates records (Verify if data recorded is unique)
## Example  Column product_ID in products tables

SELECT 
product_ID,
COUNT(product_ID) AS ID_appearance
FROM products
GROUP BY 1
HAVING ID_appearance > 1
 
## Empty Values (Verify if there is record with empty value in any column)
## Example of sales table

SELECT 
* 
FROM sales
WHERE 
(Sale_ID || Date || Store_ID || Product_ID || Units ) IS NULL
 
 # Step 3 :  Data exploration
 ## Determine MIN, MAX, Average of product price, product cost, units in stock
 
SELECT 
MAX(Stock_On_Hand) AS Highest_stock,
MIN(Stock_On_Hand) AS lowest_stock,
AVG(Stock_On_Hand) AS Average_stock
FROM inventory

SELECT 
MAX(Product_Price) AS Highest_sale_price,
MIN(Product_Price) AS lowest_sale_price,
AVG(Product_Price) AS Average__sale_price
FROM products

SELECT 
MAX(Product_Cost) AS Highest_Product_Cost,
MIN(Product_Cost) AS lowest_Product_Cost,
AVG(Product_Cost) AS Average__Product_Cost
FROM products

## Determine product category
## There is only for products categories (5 categories)

SELECT 
DISTINCT Product_Category AS category
FROM products
 
## Determine store_location
## There is four type stores location (4 stores)
 
SELECT 
DISTINCT Store_Location
FROM stores

# Step 4 :  Analysis
## Total sales transactions

SELECT 
COUNT(Sale_ID) AS total_sales_transactions
FROM sales

## Sales Units per products and the first fifth  products sold

SELECT 
s.Product_ID AS product_ID,
d.Product_Name AS name,
SUM(s.units) AS Units_sold
FROM sales s
INNER JOIN products d
ON s.product_ID = d.product_ID
GROUP BY 1, 2
ORDER BY Units_sold DESC  LIMIT 5    

## Revenue per product and the first fifth  revenue

SELECT 
s.Product_ID AS product_ID,
d.Product_Name AS name,
d.Product_Price,
SUM(s.units) AS Units_sold,
d.Product_Price * SUM(s.units) AS revenue_per_product
FROM sales s
INNER JOIN products d
ON s.product_ID = d.product_ID
GROUP BY 1, 2, 3
ORDER BY revenue_per_product DESC  LIMIT 5  
 
## Cost per product

SELECT 
s.Product_ID AS product_ID,
d.Product_Name AS name,
d.Product_Cost,
SUM(s.units) AS Units_sold,
d.Product_Cost * SUM(s.units) AS cost_per_product
FROM sales s
INNER JOIN products d
ON s.product_ID = d.product_ID
GROUP BY 1, 2, 3
ORDER BY cost_per_product DESC   
 
## Profit and profit percentage per product

SELECT 
s.Product_ID AS product_ID,
d.Product_Name AS name,
d.Product_Cost,
d.product_price,
SUM(s.units) AS Units_sold,
SUM(s.units) * d.Product_Cost AS cost_per_product,
d.Product_Price * SUM(s.units) AS revenue_per_product,
d.Product_Price * SUM(s.units) - d.Product_Cost * SUM(s.units) AS profit,
ROUND(((d.Product_Price * SUM(s.units) - d.Product_Cost * SUM(s.units)) / (d.Product_Price * SUM(s.units))) * 100, 2) AS profit_percentage
FROM sales s
INNER JOIN products d
ON s.product_ID = d.product_ID
GROUP BY 1, 2, 3, 4
ORDER BY profit_percentage DESC 
 
## Total cost, total revenue, total profit and profit percentage 
## I use temporary table as follows 
## Temporary table for revenue per product

CREATE TEMPORARY TABLE productRevenue
SELECT 
s.Product_ID AS product_ID,
d.Product_Name AS name,
d.Product_Price,
SUM(s.units) AS Units_sold,
d.Product_Price * SUM(s.units) AS revenue_per_product
FROM sales s
INNER JOIN products d
ON s.product_ID = d.product_ID
GROUP BY 1, 2, 3

## Temporary table for cost per product

CREATE TEMPORARY TABLE productCosts
SELECT 
s.Product_ID AS product_ID,
d.Product_Name AS name,
d.Product_Cost,
SUM(s.units) AS Units_sold,
d.Product_Cost * SUM(s.units) AS cost_per_product
FROM sales s
INNER JOIN products d
ON s.product_ID = d.product_ID
GROUP BY 1, 2, 3

## THEN determine total profit and profit percentage by joining the two temporary tables and grouping revenue and costs

SELECT
SUM(pr.revenue_per_product) AS total_revenue,
SUM(pc.cost_per_product) AS total_cost,
SUM(pr.revenue_per_product) - SUM(pc.cost_per_product) AS profit,
ROUND(((SUM(pr.revenue_per_product) - SUM(pc.cost_per_product)) / SUM(pr.revenue_per_product))*100, 2)  AS profit_percentage
FROM productRevenue pr
JOIN productCosts pc
ON pr.product_ID = pc.product_ID

## Total cost, total revenue, total profit and profit percentage per type of location  
## Remember there are four types (residential, airport, commercial and downtown)
## I use subquery as follows 

SELECT 
Store_Location,
SUM(revenue_per_product) AS revenue,
SUM(cost_per_product) AS costs,
SUM(revenue_per_product) - SUM(cost_per_product) AS profit,
(SUM(revenue_per_product) - SUM(cost_per_product)) / SUM(revenue_per_product) AS profit_percentage
FROM(
SELECT
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
GROUP BY 1, 2, 3, 4, 5
) AS revenue_and_costs
GROUP BY 1
ORDER BY profit_percentage DESC  

-- Even if revenu is higher in stores located in downtown and commercial, profit 
-- is higher in Airport location. I think that costs in downtown could be analyzed deeply to find what can be changed to improve profit

## Total cost, total revenue, total profit and profit percentage per year and per category of toys 
## I use subquery and CASE function as follows

SELECT 
year,
SUM(CASE WHEN Product_Category = 'Toys' THEN revenue_per_product ELSE NULL END) AS toys_revenue,
SUM(CASE WHEN Product_Category = 'Art & Crafts' THEN revenue_per_product ELSE NULL END) AS Art_Crafts_revenue,
SUM(CASE WHEN Product_Category = 'Games' THEN revenue_per_product ELSE NULL END) AS Games_revenue,
SUM(CASE WHEN Product_Category = 'Electronics' THEN revenue_per_product ELSE NULL END) AS Electronics_revenue,
SUM(CASE WHEN Product_Category = 'Sports & Outdoors' THEN revenue_per_product ELSE NULL END) AS Sports_Outdoors_revenue
FROM(
SELECT
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
GROUP BY 1, 2, 3, 4, 5, 6
) AS revenue_and_costs
GROUP BY 1

## Total cost, total revenue, total profit and profit percentage per year 
## I use subquery as follows

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
INNER JOIN products d
ON s.product_ID = d.product_ID
GROUP BY 1,2,3,4,5 ) AS revenue_and_costs
GROUP BY 1

## Total cost, total revenue, total profit and profit percentage per year and month
## I use subquery as follows

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
GROUP BY 1,2,3,4,5,6 ) AS revenue_and_costs
GROUP BY 1,2

## Cost  Value of inventory
## 1- cost per product

SELECT
i.Product_ID,
p.Product_Cost,
SUM(i.Stock_On_Hand),
SUM(i.Stock_On_Hand) * p.Product_Cost AS inventory_cost_per_product
FROM inventory i
LEFT JOIN products p
USING(Product_ID)
GROUP BY 1,2

## 2- total inventory cost

SELECT
SUM(inventory_cost_per_product) inv_cost
FROM(

SELECT
i.Product_ID,
p.Product_Cost,
SUM(i.Stock_On_Hand),
SUM(i.Stock_On_Hand) * p.Product_Cost AS inventory_cost_per_product
FROM inventory i
LEFT JOIN products p
USING(Product_ID)
GROUP BY 1,2) AS inv_cost
