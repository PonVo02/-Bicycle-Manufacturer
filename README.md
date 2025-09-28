# -Bicycle-Manufacturer
### Project Introduction
This project uses a sales dataset containing information about orders, products, discounts, customers, and inventory.  
It is designed for practicing SQL through business-focused case questions.  
By working with this dataset, I aim to apply SQL techniques such as aggregation, ranking, cohort analysis, and growth calculations to generate insights similar to real-world data analytics tasks.  
[Dataset](https://console.cloud.google.com/bigquery?hl=vi&inv=1&invt=Ab1UAQ&project=my-project-sql-464309&ws=!1m5!1m4!4m3!1sadventureworks2019!2sSales!3sSalesOrderDetail)

| Field Name            | Data Type  | Description                                                                 |
|-----------------------|------------|-----------------------------------------------------------------------------|
| SalesOrderID          | INTEGER    | Unique identifier for each sales order.                                     |
| SalesOrderDetailID    | INTEGER    | Unique identifier for each sales order line item.                           |
| CarrierTrackingNumber | STRING     | Tracking number provided by the shipping carrier.                           |
| OrderQty              | INTEGER    | Number of product units ordered in this line item.                          |
| ProductID             | INTEGER    | Identifier for the product being sold.                                      |
| SpecialOfferID        | INTEGER    | Identifier for the discount or special offer applied.                       |
| UnitPrice             | FLOAT      | Price per unit of the product at the time of the order.                     |
| UnitPriceDiscount     | FLOAT      | Discount applied to the product’s unit price.                               |
| LineTotal             | FLOAT      | Total amount for this line item (calculated as OrderQty × UnitPrice - Discount). |
| rowguid               | STRING     | Globally unique identifier (GUID) for the row.                              |
| ModifiedDate          | TIMESTAMP  | The date and time when the record was last updated.                         |

--- 
## Query 1: Calc Quantity of items, Sales value & Order quantity by each Subcategory in L12M
```SQL
SELECT 
  FORMAT_DATE('%b%Y', a.ModifiedDATE) AS period
  , c.Name 
  , SUM(a.OrderQty) AS qty_item
  , SUM(a.LineTotal) AS total_sales
  , COUNT(DISTINCT a.SalesOrderID) AS order_cnt 
FROM `adventureworks2019.Sales.SalesOrderDetail` AS a 
LEFT JOIN `adventureworks2019.Production.Product` AS b
  ON a.ProductID = b.ProductID 
LEFT JOIN `adventureworks2019.Production.ProductSubcategory` AS c
  ON CAST (b.ProductSubcategoryID AS int) = c.ProductSubcategoryID
WHERE DATE(a.ModifiedDate) >= (
  SELECT 
    DATE_SUB (MAX(DATE(ModifiedDate)), INTERVAL 12 month)
  FROM `adventureworks2019.Sales.SalesOrderDetail`)
GROUP BY period, Name
ORDER BY period DESC, Name , qty_item;
```
| Period   | Subcategory       | Qty Item | Total Sales   | Order Count |
|----------|-------------------|----------|---------------|-------------|
| Sep2013  | Bike Racks        | 312      | 22,828.51     | 71          |
| Sep2013  | Helmets           | 1,016    | 27,354.84     | 480         |
| Sep2013  | Jerseys           | 1,402    | 48,318.09     | 304         |
| Sep2013  | Mountain Bikes    | 1,194    | 1,244,716.84  | 263         |
| Sep2013  | Road Bikes        | 1,398    | 1,318,888.69  | 319         |
| Oct2013  | Bike Racks        | 284      | 21,181.20     | 70          |
| Oct2013  | Helmets           | 1,127    | 30,733.99     | 557         |
| Oct2013  | Jerseys           | 1,372    | 47,761.26     | 324         |
| Oct2013  | Mountain Bikes    | 1,231    | 1,338,738.67  | 305         |
| Oct2013  | Road Bikes        | 1,372    | 1,318,888.69  | 319         |

[Link full Query results] (https://drive.google.com/file/d/1yOO8AW_bB_X7Jv7bYw1CpgI6-vUIuNhs/view)

### 🔍 Insights
- Top revenue drivers: Road Bikes, Mountain Bikes, and Touring Bikes consistently generate the highest sales.
- Accessories (Helmets, Jerseys, Tires & Tubes) show strong and steady demand, supporting main product categories.
- Certain months (e.g., Jun 2014) show sharp declines in multiple subcategories, possibly due to seasonality or missing data.
- Overall, from mid-2013 to mid-2014, bike categories dominate revenue, while accessories provide stable recurring sales.

### 💡 Recommendations
- Focus marketing and promotions on top categories (Road/Mountain/Touring Bikes) to sustain revenue growth.
- Leverage cross-selling with accessories (Helmets, Jerseys, Tires & Tubes) to boost average order value.
- Monitor seasonal trends to optimize inventory and avoid overstock/stockouts during low-demand periods.
- Expand analysis across all months to confirm seasonality patterns and long-term growth opportunities.

---
## Query 2: Calc % YoY growth rate by SubCategory & release top 3 cat with highest grow rate.
```SQL
WITH sale_info AS (
  SELECT 
      FORMAT_TIMESTAMP("%Y", a.ModifiedDate) AS yr,
      c.Name,
      SUM(a.OrderQty) AS qty_item
  FROM `adventureworks2019.Sales.SalesOrderDetail` a 
  LEFT JOIN `adventureworks2019.Production.Product` b 
    ON a.ProductID = b.ProductID
  LEFT JOIN `adventureworks2019.Production.ProductSubcategory` c 
    ON CAST(b.ProductSubcategoryID AS INT64) = c.ProductSubcategoryID
  GROUP BY 1,2
),

sale_diff AS (
  SELECT 
    yr,
    Name,
    qty_item,
    LEAD(qty_item) OVER (PARTITION BY Name ORDER BY yr DESC) AS prv_qty,
    ROUND(qty_item / CAST(LEAD(qty_item) OVER (PARTITION BY Name ORDER BY yr DESC) AS FLOAT64) - 1, 2) AS qty_diff
  FROM sale_info
),

rk_qty_diff AS (
  SELECT 
    yr,
    Name,
    qty_item,
    prv_qty,
    qty_diff,
    DENSE_RANK() OVER(ORDER BY qty_diff DESC) AS dk
  FROM sale_diff
)

SELECT 
  Name,
  qty_item,
  prv_qty,
  qty_diff
FROM rk_qty_diff 
WHERE dk <= 3
ORDER BY dk;
```
| Rank | Subcategory      | Qty Item | Prev Qty | YoY Growth (%) |
|------|------------------|----------|----------|----------------|
| 1    | Mountain Frames  | 3,168    | 510      | 521%           |
| 2    | Socks            | 2,724    | 523      | 421%           |
| 3    | Road Frames      | 5,564    | 1,137    | 389%           |

### 🔍 Insights
- Mountain Frames saw the highest growth (+521%), followed by Socks and Road Frames.
- The top 3 categories all experienced very strong YoY increases compared to previous periods.

### 💡 Recommendations
- Allocate more inventory and marketing to Mountain Frames, Socks, and Road Frames to sustain growth.
- Investigate drivers of demand (promotions, seasonality, product launches) to replicate success in other categories.
