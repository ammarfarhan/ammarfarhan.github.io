---
title: "EDA with SQL: theLook eCommerce 2022"
date: 2023-05-13
draft: false
author: "Ammar Farhan Nur Hakim"
tags:
    - Data Visualization
    - EDA
    - SQL
    - Tableau
image: /images/thelook-ecommerce/main.jpg
description: ""
toc:
---

## Description

I performed a data analysis for theLook eCommerce, a dummy fashion e-commerce company with the goal of gaining valuable insights.

The analysis involved **calculating the monthly growth of inventory** by product category, **identifying the category that showed the lowest business growth** over the past year, and **creating a retention cohort** in 2022.

### Project Details
| Type       | Tools    |
| ---------- | -------- |
| Personal Project | Google BigQuery |
| (derived from RevoU Assignment) | Google Sheets  |
|                     | Tableau |


## Dataset Overview
theLook is a fictitious eCommerce clothing site developed by the Looker team. 

The dataset contains information about customers, products, orders, logistics, web events and digital marketing campaigns. 

The contents of this dataset are synthetic, and are provided to industry practitioners for the purpose of product discovery, testing, and evaluation.

This dataset is public and hosted in Google BigQuery.

[>> Link to Dataset <<][thelook]

## Business Background

You are a data analyst in a fashion e-commerce company called theLook. Currently, the company is in the optimization mode caused by the potential crisis in 2023. The management has decided to cut off resources in some categories the lowest growth in the past 1 year. On another side, they want to continue the analysis by understanding their inventory stock growth and the retention behaviors of the users.

## Objectives
1. Find categories with the lowest business growth.
2. Calculate monthly growth of inventory in percentage breakdown by product categories.
3. Create monthly user retention cohorts.

Data used will be data from 2022.

### 1. Categories with lowest business growth

<img src='/images/thelook-ecommerce/schema-1.jpg' alt='theLook eCommerce Table Schema 1'>

#### SQL Syntax
```sql
WITH main AS(
  SELECT COUNT(oi.id) AS products_sold,
        ii.product_category AS category,
        DATE(DATE_TRUNC(o.created_at, MONTH)) AS year_month,
        SUM(oi.sale_price) AS revenue,
        SUM(oi.sale_price) - SUM(ii.cost) AS profit
  FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
  JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  ON oi.order_id = o.order_id
  JOIN `bigquery-public-data.thelook_ecommerce.inventory_items` AS ii
  ON oi.inventory_item_id = ii.id
  WHERE EXTRACT(YEAR FROM o.created_at) IN (2021, 2022)
    AND o.status = 'Complete'
  GROUP BY category, year_month
  ORDER BY category, year_month
),

cte_revenue AS(
  SELECT *
  FROM(
    SELECT * EXCEPT(profit),
          LAG(revenue) OVER(PARTITION BY category ORDER BY year_month) AS previous_revenue
    FROM main
  )
),

cte_profit AS(
  SELECT *
  FROM(
    SELECT * EXCEPT(revenue),
          LAG(profit) OVER(PARTITION BY category ORDER BY year_month) AS previous_profit
    FROM main
  )
)

SELECT main.category,
       main.year_month,
       main.products_sold,
       rev.revenue,
       rev.previous_revenue,
       pro.profit,
       pro.previous_profit,
       (rev.revenue - rev.previous_revenue) / rev.previous_revenue AS revenue_growth,
       (pro.profit - pro.previous_profit)  /pro.previous_profit AS profit_growth
FROM main
JOIN cte_revenue AS rev
ON main.category = rev.category AND main.year_month = rev.year_month
JOIN cte_profit AS pro
ON main.category = pro.category AND main.year_month = pro.year_month
WHERE EXTRACT(YEAR FROM main.year_month) IN (2022)
ORDER BY category, year_month
```

#### Visualization
Link the query to Google Sheets and create a visualization using Tableau.

[>> Try this dashboard <<][dashboard]

<img src='/images/thelook-ecommerce/bcg-matrix.jpg' alt='theLook eCommerce Dashboard'>

There are 3 visualizations within this dashboard:

1. BCG Matrix:
    - Profit CMGR : profit in latest month / profit in first month <sup>1 / [latest month - first month]</sup> - 1
    - Market Share : revenue product / total revenue
2. Profit / revenue growth time-series chart.
3. Categories breakdown by its revenue, profit, and number of products sold.

### 2. Monthly Growth of Inventory

<img src='/images/thelook-ecommerce/schema-2.jpg' alt='theLook eCommerce Table Schema 2'>

#### SQL Syntax
```sql
WITH main AS(
  SELECT product_category AS category,
         DATE(DATE_TRUNC(ii.created_at, MONTH)) AS year_month,
         count(ii.id) AS stock
  FROM `bigquery-public-data.thelook_ecommerce.inventory_items` AS ii
  LEFT JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  ON ii.id = oi.inventory_item_id
  WHERE EXTRACT(YEAR FROM ii.created_at) IN (2021, 2022)
    AND ii.sold_at IS NULL
  GROUP BY category, year_month
),

cte_inventory AS(
  SELECT *
    FROM(
      SELECT *,
            LAG(stock) OVER(PARTITION BY category ORDER BY year_month) AS previous_stock
      FROM main
    )
)

SELECT *,
        (stock - previous_stock) / previous_stock AS inventory_growth
FROM cte_inventory
WHERE EXTRACT(YEAR FROM year_month) IN (2022)
ORDER BY category, year_month
```

#### Visualization
Link the query to Google Sheets and create a Pivot Table.

<iframe src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQHUAvIvg5O8mXhBXj3xF8tFpSrCbHBcQqHCE_nbwidZly_Ikrw87F13VbCqIbH-W-k3U3_wrcM9Jqd/pubhtml?gid=1518340926&amp;single=true&amp;widget=true&amp;headers=false" frameborder="0" width="100%" height="600" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

### 3. Monthly User Retention Cohorts

<img src='/images/thelook-ecommerce/schema-3.jpg' alt='theLook eCommerce Table Schema 3'>

#### SQL Syntax
```sql
WITH first_purchase AS(
  SELECT user_id,
         MIN(DATE(DATE_TRUNC(created_at, MONTH))) AS purchase_month
  FROM `bigquery-public-data.thelook_ecommerce.orders`
  WHERE EXTRACT(YEAR FROM created_at) IN (2022)
    AND status = 'Complete'
  GROUP BY user_id
),

-- only 1 purchase per month tracked here
activity AS(
  SELECT o.user_id,
         DATE_DIFF(
          DATE(DATE_TRUNC(o.created_at, MONTH)),
          fp.purchase_month,
          MONTH
         ) AS month_number
  FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
  JOIN first_purchase AS fp
  ON o.user_id = fp.user_id
  WHERE EXTRACT(YEAR FROM o.created_at) IN (2022)
    AND status = 'Complete'
  GROUP BY o.user_id, month_number
),

cohort_size AS(
  SELECT purchase_month,
         COUNT(user_id) AS num_user
  FROM first_purchase
  GROUP BY purchase_month
  ORDER BY purchase_month
),

retention AS(
  SELECT fp.purchase_month,
         activity.month_number,
         COUNT(activity.user_id) AS num_user
  FROM activity
  JOIN first_purchase AS fp
  ON activity.user_id = fp.user_id
  GROUP BY fp.purchase_month, activity.month_number
  ORDER BY fp.purchase_month, activity.month_number
)

SELECT retention.purchase_month,
       size.num_user AS cohort_size,
       retention.month_number,
       retention.num_user,
       retention.num_user/size.num_user AS retention_percentage
FROM retention
JOIN cohort_size AS size
ON retention.purchase_month = size.purchase_month
ORDER BY purchase_month, month_number
```

#### Visualization
Link the query to Google Sheets and create a Pivot Table.

<iframe src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQHUAvIvg5O8mXhBXj3xF8tFpSrCbHBcQqHCE_nbwidZly_Ikrw87F13VbCqIbH-W-k3U3_wrcM9Jqd/pubhtml?gid=594779104&amp;single=true&amp;widget=true&amp;headers=false" frameborder="0" width="100%" height="400" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

## Slide Deck (with insights)

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vTKB5jVXYgu-1Zx3hBP7P4B9aP--KOTtlZMrPqKd593IhCybqdjBm3g19BCjaMYRgHWUQL-_M2NKCdT/embed?start=false&loop=false&delayms=3000" frameborder="0" width="100%" height="400" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

Thank you for taking the time to visit my portfolio website and view my projects!

I appreciate your interest and support. If you have any feedback or questions, please don't hesitate to reach out to me. Once again, thank you for your time and consideration.

[>> Let's connect on LinkedIn! <<][linkedin]

[>> Visit my Tableau Public profile to see my other dashboards <<][tableau]


[linkedin]: https://www.linkedin.com/in/ahanaki/
[tableau]: https://public.tableau.com/app/profile/ahanaki
[thelook]:https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce
[dashboard]:https://public.tableau.com/views/theLookeCommerce-ProductsBCGMatrix2022/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link