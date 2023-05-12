---
title: "EDA with SQL: theLook eCommerce 2022"
date: 2023-05-10
draft: false
author: "Ammar Farhan Nur Hakim"
tags:
    - COMING SOON
    - Data Visualization
    - EDA
    - Looker Studio
    - SQL
image: /images/thelook-ecommerce/main.png
description: ""
toc:
---

## Description

I performed a data analysis for theLook eCommerce, a dummy fashion e-commerce company with the goal of gaining valuable insights.

The analysis involved **calculating the monthly growth of inventory** by product category, **identifying the category that showed the lowest business growth** over the past year, and **creating a retention cohort** in 2022.

### Project Details
| Type       | Tools    |
| ---------- | -------- |
| Improved Assignment | Google BigQuery |
|  | Google Sheets  |
|  | Looker Studio  |

## Business Background

You are a data analyst in a fashion e-commerce company called theLook. Currently, the company is in the optimization mode caused by the potential crisis in 2023. The management has decided to cut off resources in some categories the lowest growth in the past 1 year. On another side, they want to continue the analysis by understanding their inventory stock growth and the retention behaviors of the users.

## Business Questions
1. Find categories with the lowest business growth.
2. Calculate monthly growth of inventory in percentage breakdown by product categories.
3. Create monthly user retention cohorts.

Data used will be data from 2022.

## Dataset Overview
theLook is a fictitious eCommerce clothing site developed by the Looker team. The dataset contains information about customers, products, orders, logistics, web events and digital marketing campaigns. The contents of this dataset are synthetic, and are provided to industry practitioners for the purpose of product discovery, testing, and evaluation.

This public dataset is hosted in Google BigQuery and is included in BigQuery's 1TB/mo of free tier processing. This means that each user receives 1TB of free BigQuery processing every month, which can be used to run queries on this public dataset.

[>> Link to Dataset <<][thelook]

## Categories with lowest business growth

#### Table Schema
<img src='/images/thelook-ecommerce/schema-1.png' alt='theLook eCommerce Table Schema 1'>

#### SQL Syntax
```sql
WITH main AS(
  SELECT COUNT(oi.id) AS products_sold,
        ii.product_category AS category,
        EXTRACT(MONTH FROM o.created_at) AS month,
        EXTRACT(YEAR FROM o.created_at) AS year,
        SUM(oi.sale_price) AS revenue,
        SUM(oi.sale_price) - SUM(ii.cost) AS profit
  FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
  JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  ON oi.order_id = o.order_id
  JOIN `bigquery-public-data.thelook_ecommerce.inventory_items` AS ii
  ON oi.inventory_item_id = ii.id
  WHERE EXTRACT(YEAR FROM o.created_at) IN (2021, 2022)
    AND o.status = 'Complete'
  GROUP BY category, month, year
  ORDER BY category, year, month
),

cte_revenue AS(
  SELECT *
  FROM(
    SELECT * EXCEPT(profit),
          LAG(revenue) OVER(PARTITION BY category ORDER BY year, month) AS previous_revenue
    FROM main
  )
),

cte_profit AS(
  SELECT *
  FROM(
    SELECT * EXCEPT(revenue),
          LAG(profit) OVER(PARTITION BY category ORDER BY year, month) AS previous_profit
    FROM main
  )
)

SELECT main.category,
       main.month,
       main.year,
       main.products_sold,
       rev.revenue,
       rev.previous_revenue,
       pro.profit,
       pro.previous_profit,
       (rev.revenue - rev.previous_revenue) / rev.previous_revenue AS revenue_growth,
       (pro.profit - pro.previous_profit)  /pro.previous_profit AS profit_growth
FROM main
JOIN cte_revenue AS rev
ON main.category = rev.category AND main.year = rev.year AND main.month = rev.month
JOIN cte_profit AS pro
ON main.category = pro.category AND main.year = pro.year AND main.month = pro.month
WHERE rev.year IN (2022)
ORDER BY category, year, month
```

## Monthly Growth of Inventory

#### Table Schema
<img src='/images/thelook-ecommerce/schema-2.png' alt='theLook eCommerce Table Schema 2'>

#### SQL Syntax
```sql
WITH main AS(
  SELECT product_category AS category,
         EXTRACT(MONTH FROM ii.created_at) AS month,
         EXTRACT(YEAR FROM ii.created_at) AS year,
         count(ii.id) AS stock
  FROM `bigquery-public-data.thelook_ecommerce.inventory_items` AS ii
  LEFT JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  ON ii.id = oi.inventory_item_id
  WHERE EXTRACT(YEAR FROM ii.created_at) IN (2021, 2022)
    AND ii.sold_at IS NULL
  GROUP BY category, month, year
),

cte_inventory AS(
  SELECT *
    FROM(
      SELECT *,
            LAG(stock) OVER(PARTITION BY category ORDER BY year, month) AS previous_stock
      FROM main
    )
)

SELECT *,
        (stock - previous_stock) / previous_stock AS inventory_growth
FROM cte_inventory
WHERE year IN (2022)
ORDER BY category, year, month
```

## Monthly User Retention Cohorts

#### Table Schema
<img src='/images/thelook-ecommerce/schema-3.png' alt='theLook eCommerce Table Schema 3'>

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

## Slide Deck

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vTKB5jVXYgu-1Zx3hBP7P4B9aP--KOTtlZMrPqKd593IhCybqdjBm3g19BCjaMYRgHWUQL-_M2NKCdT/embed?start=false&loop=false&delayms=3000" frameborder="0" width="100%" height="400" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

[thelook]:https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce