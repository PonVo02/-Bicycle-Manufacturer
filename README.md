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
---
## Query 3: Ranking Top 3 TeritoryID with biggest Order quantity of every year. If there's TerritoryID with same quantity in a year, do not skip the rank number
```SQL
WITH territory_sales AS ( --Tính tổng số lượng đơn hàng theo năm và khu vực (territotyID)
  SELECT 
    EXTRACT(YEAR FROM DATE(a.ModifiedDate)) AS yr,
    b.TerritoryID,
    SUM(a.OrderQty) AS order_quantity
  FROM `adventureworks2019.Sales.SalesOrderDetail` AS a
  JOIN `adventureworks2019.Sales.SalesOrderHeader` AS b
    ON a.SalesOrderID = b.SalesOrderID
  GROUP BY yr, b.TerritoryID
),

ranked_territories AS(-- Xếp hạng các khu vực theo tổng số lượng đặt hàng từng năm
  SELECT 
    yr,
    TerritoryID,
    order_quantity,
    DENSE_RANK() OVER (PARTITION BY yr ORDER BY order_quantity DESC) AS rank
  FROM territory_sales
)
-- top  3 khu vực mỗi năm theo số lượng đơn hàng
SELECT *
FROM ranked_territories
WHERE rank <= 3
ORDER BY MOD(yr,2), yr DESC;
```
| Year | TerritoryID | Order Quantity | Rank |
|------|-------------|----------------|------|
| 2014 | 4           | 11,632         | 1    |
| 2014 | 6           | 9,711          | 2    |
| 2014 | 1           | 8,823          | 3    |
| 2013 | 4           | 26,682         | 1    |
| 2013 | 6           | 22,553         | 2    |
| 2013 | 1           | 17,452         | 3    |
| 2012 | 4           | 17,553         | 1    |
| 2012 | 6           | 14,412         | 2    |
| 2012 | 1           | 8,537          | 3    |
| 2011 | 4           | 3,238          | 1    |
| 2011 | 6           | 2,705          | 2    |
| 2011 | 1           | 1,964          | 3    |
### 🔍 Insights
- Territory 4 consistently ranked 1st in order quantity across all years.
- Territory 6 held the 2nd position, showing stable but lower volume than Territory 4.
- Territory 1 stayed at 3rd place each year, with lower order volume compared to others.

💡 Recommendations
- Prioritize resources and marketing in Territory 4 to maintain dominance.
- Explore growth opportunities in Territory 6, which shows consistent demand but has potential to close the gap.
- Investigate Territory 1 for barriers to growth and consider targeted campaigns to boost sales.
--- 
## Query 4: Calc Total Discount Cost belongs to Seasonal Discount for each SubCategory
```SQL
select 
    FORMAT_TIMESTAMP("%Y", ModifiedDate)
    , Name
    , sum(disc_cost) as total_cost
from (
      select distinct a.ModifiedDate
      , c.Name
      , d.DiscountPct, d.Type
      , a.OrderQty * d.DiscountPct * UnitPrice as disc_cost 
      from `adventureworks2019.Sales.SalesOrderDetail` a
      LEFT JOIN `adventureworks2019.Production.Product` b on a.ProductID = b.ProductID
      LEFT JOIN `adventureworks2019.Production.ProductSubcategory` c on cast(b.ProductSubcategoryID as int) = c.ProductSubcategoryID
      LEFT JOIN `adventureworks2019.Sales.SpecialOffer` d on a.SpecialOfferID = d.SpecialOfferID
      WHERE lower(d.Type) like '%seasonal discount%' 
)
group by 1,2;
```
| Year | Subcategory | Total Discount Cost |
|------|-------------|----------------------|
| 2012 | Helmets     | 149.72               |
| 2013 | Helmets     | 543.22               |
### 🔍 Insights
- Helmet discount cost increased sharply from 2012 (149.7) to 2013 (543.2).
- This suggests higher promotion or seasonal discount activity on Helmets in 2013.

### 💡 Recommendations
- Evaluate effectiveness of the 2013 discount campaigns .
- Optimize discount strategy to balance between driving sales and controlling costs.
--- 
## Query 5: Retention rate of Customer in 2014 with status of Successfully Shipped (Cohort Analysis)
```SQL
with 
info as (
  select  
      extract(month from ModifiedDate) as month_no
      , extract(year from ModifiedDate) as year_no
      , CustomerID
      , count(Distinct SalesOrderID) as order_cnt
  from `adventureworks2019.Sales.SalesOrderHeader`
  where FORMAT_TIMESTAMP("%Y", ModifiedDate) = '2014'
  and Status = 5
  group by 1,2,3
  order by 3,1 
),

row_num as (
  select *
      , row_number() over (partition by CustomerID order by month_no) as row_numb
  from info 
), 

first_order as (   
  select *
  from row_num
  where row_numb = 1
), 

month_gap as (
  select 
      a.CustomerID
      , b.month_no as month_join
      , a.month_no as month_order
      , a.order_cnt
      , concat('M - ',a.month_no - b.month_no) as month_diff
  from info a 
  left join first_order b 
  on a.CustomerID = b.CustomerID
  order by 1,3
)

select month_join
      , month_diff 
      , count(distinct CustomerID) as customer_cnt
from month_gap
group by 1,2
order by 1,2;
```
| Cohort Month | Period (M - n) | Customer Count |
|--------------|----------------|----------------|
| 1            | M - 0          | 2076           |
| 1            | M - 1          | 78             |
| 1            | M - 2          | 89             |
| 1            | M - 3          | 252            |
| 1            | M - 4          | 96             |
| 1            | M - 5          | 61             |
| 1            | M - 6          | 18             |
| 2            | M - 0          | 1805           |
| 2            | M - 1          | 51             |
| 2            | M - 2          | 61             |
| 2            | M - 3          | 234            |
| 2            | M - 4          | 58             |
| 2            | M - 5          | 8              |
| 3            | M - 0          | 1918           |
| 3            | M - 1          | 43             |
| 3            | M - 2          | 58             |
| 3            | M - 3          | 44             |
| 3            | M - 4          | 11             |
| 4            | M - 0          | 1906           |
| 4            | M - 1          | 34             |
| 4            | M - 2          | 44             |
| 4            | M - 3          | 7              |
| 5            | M - 0          | 1947           |
| 5            | M - 1          | 40             |
| 5            | M - 2          | 7              |
| 6            | M - 0          | 909            |
| 6            | M - 1          | 10             |
| 7            | M - 0          | 148            |

### 🔍 Insights
- Customer acquisition is strong at M - 0 .
- Retention drops sharply from M - 1 onward .
- Some cohorts (like month 1, M - 3 = 252) show a slight spike of re-engagement, possibly due to campaigns or seasonality.
- By later months, most cohorts shrink to very small numbers, indicating limited long-term retention.

⸻

💡 Recommendations
	•	Strengthen onboarding & engagement in the first month to reduce early churn.
	•	Analyze re-engagement campaigns (like cohort 1 at M - 3) and replicate what worked.
	•	Introduce loyalty or subscription programs to improve long-term customer retention.
	•	Segment churned users and run targeted win-back campaigns.
