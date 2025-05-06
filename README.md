# Alibert Furnishings - Data Analysis Project Documentation

## Business Introduction

Alibert Furnishings is a sustainability-focused company offering stylish, durable furniture made from eco-friendly materials. Operating both through an online platform and physical stores nationwide, they provide a wide range of products, from office furniture to home essentials. The company's mission is to promote sustainable living by prioritizing quality, environmental responsibility, and customer satisfaction.

---

## Problem Statement

Alibert Furnishings requires deeper insights into the relationships between product sales, customer demographics, and inventory levels. Management seeks to:

* Better understand customer behavior and product performance across different regions.
* Analyze how stock levels influence sales outcomes.
* Enable informed decision-making in areas such as product development, targeted marketing strategies, and inventory optimization.

---

## Aims of the Project

* Enable informed decision-making in key strategic areas.
* Gain insights into relationships between sales, customer demographics, and stock levels.
* Evaluate product performance and customer behavior across regions.
* Integrate multi-table data for comprehensive analysis.

---

## Project Tasks & Queries

### Q1: Group Products into Price Categories

**Objective**: Categorize products as Budget-Friendly, Mid-Range, or Premium based on unit price.

```sql
SELECT Productid, productname, Price,
  CASE
    WHEN price < 200 THEN 'Budget Friendly'
    WHEN price BETWEEN 200 AND 400 THEN 'Mid-Range'
    ELSE 'Premium'
  END AS PriceCategory
FROM dimproduct;
```

### Q2: Current Stock Status of Each Product

**Objective**: Show stock, total sold quantity, and remaining quantity.

```sql
SELECT dp.productid, dp.productname, dp.stockquantity,
       COALESCE(SUM(fs.quantitysold), 0) AS Total_Qty_Sold,
       (dp.stockquantity - COALESCE(SUM(fs.quantitysold), 0)) AS Qty_remaining
FROM dimproduct dp
LEFT JOIN factsales fs ON dp.productid = fs.productid
GROUP BY dp.productid, dp.productname, dp.stockquantity
ORDER BY Total_Qty_Sold;
```

### Q3: Highest and Lowest Sales by Customers

**Objective**: Find customers with the highest and lowest total sales.

```sql
WITH CustomerSales AS (
  SELECT dc.customerid, dc.firstname, dc.lastname,
         SUM(fs.saleamount) AS total_sales_amount
  FROM factsales fs
  JOIN dimcustomer dc ON fs.customerid = dc.customerid
  GROUP BY dc.customerid, dc.firstname, dc.lastname
)
SELECT * FROM CustomerSales
WHERE total_sales_amount = (SELECT MAX(total_sales_amount) FROM CustomerSales)
   OR total_sales_amount = (SELECT MIN(total_sales_amount) FROM CustomerSales);
```

### Q4: Top 3 Purchasing Customers (Eligible for Reward)

```sql
SELECT dc.customerid, dc.firstname, dc.lastname,
       SUM(fs.saleamount) AS Total_Purchase
FROM dimcustomer dc
JOIN factsales fs ON dc.customerid = fs.customerid
GROUP BY dc.customerid, dc.firstname, dc.lastname
ORDER BY Total_Purchase DESC
LIMIT 3;
```

### Q5: Identify High Purchasing Customers for Loyalty Campaigns

```sql
SELECT dc.customerid, dc.firstname, dc.lastname,
       SUM(fs.saleamount) AS Total_Purchase,
       CASE
         WHEN SUM(fs.saleamount) > 3000 THEN 'Agba Ballers'
         WHEN SUM(fs.saleamount) BETWEEN 1000 AND 3000 THEN 'Lowkey Ballers'
         ELSE 'Urgent 2k'
       END AS Buyercategory
FROM factsales fs
JOIN dimcustomer dc ON fs.customerid = dc.customerid
GROUP BY dc.customerid, dc.firstname, dc.lastname
HAVING SUM(fs.saleamount) > 3000;
```

### Q6: Apply 10% Discount to Low-Selling Products

```sql
UPDATE dimproduct
SET price = price * 0.9
WHERE productid IN (
  SELECT dp.productid
  FROM dimproduct dp
  JOIN factsales fs ON dp.productid = fs.productid
  GROUP BY dp.productid
  HAVING SUM(fs.quantitysold) < 10
);
```

### BONUS: Customers Who Spent Above 500

```sql
WITH CustomerSales AS (
  SELECT dc.customerid, SUM(fs.saleamount) AS TotalSpent
  FROM dimcustomer dc
  JOIN factsales fs ON dc.customerid = fs.customerid
  GROUP BY dc.customerid
)
SELECT * FROM CustomerSales
WHERE TotalSpent > 500;
```

### Top 3 Best-Selling Products

```sql
WITH productsales AS (
  SELECT dp.productname, SUM(fs.quantitysold) AS Total_Sold
  FROM dimproduct dp
  JOIN factsales fs ON dp.productid = fs.productid
  GROUP BY dp.productname
)
SELECT * FROM productsales
ORDER BY Total_Sold DESC
LIMIT 3;
```

### Total and Average Monthly Revenue

```sql
WITH MonthRevenue AS (
  SELECT TO_CHAR(saledate, 'Month') AS Month_Name,
         SUM(saleamount) AS Monthly_Revenue
  FROM factsales
  GROUP BY TO_CHAR(saledate, 'Month'), DATE_PART('Month', saledate)
  ORDER BY DATE_PART('Month', saledate)
)
SELECT AVG(Monthly_Revenue) AS Avg_Monthly_Revenue
FROM MonthRevenue;
```

---

