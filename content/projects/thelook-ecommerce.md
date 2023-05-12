---
title: "TheLook eCommerce 2022"
date: 2023-02-17
draft: false
author: "Ammar Farhan Nur Hakim"
tags:
    - COMING SOON
    - Analysis
    - SQL
image: /images/coming-soon.png
description: ""
toc:
---

## Description

I performed a data analysis for TheLook eCommerce, a dummy fashion e-commerce company with the goal of gaining valuable insights.

The analysis involved **calculating the monthly growth of inventory** by product category, **identifying the category that showed the lowest business growth** over the past year, and **creating a retention cohort** in 2022.

### Project Details
| Tools    |
| -------- |
| Google BigQuery |
| Google Sheets |

## Business Background

You are a data analyst in a fashion e-commerce company called TheLook. Currently, the company is in the optimization mode caused by the potential crisis in 2023. The management has decided to cut off resources in some categories with the lowest growth in the past 1 year. On another side, they want to continue the analysis by understanding the retention behaviors of the users.

## Dataset Overview
TheLook is a fictitious eCommerce clothing site developed by the Looker team. The dataset contains information about customers, products, orders, logistics, web events and digital marketing campaigns. The contents of this dataset are synthetic, and are provided to industry practitioners for the purpose of product discovery, testing, and evaluation.

This public dataset is hosted in Google BigQuery and is included in BigQuery's 1TB/mo of free tier processing. This means that each user receives 1TB of free BigQuery processing every month, which can be used to run queries on this public dataset.

[>> Link to Dataset <<][thelook]

## Code Blocks

```sql
WITH main AS(
  SELECT product_category AS category,
         EXTRACT(MONTH FROM ii.created_at) AS month,
         EXTRACT(YEAR FROM ii.created_at) AS year,
         count(ii.id) AS stock
  FROM `sql-project-376612.thelook_ecommerce.inventory_items` AS ii
  LEFT JOIN `sql-project-376612.thelook_ecommerce.order_items` AS oi
  ON ii.id = oi.inventory_item_id
  WHERE EXTRACT(YEAR FROM ii.created_at) IN (2021, 2022)
    AND ii.sold_at IS NULL
  GROUP BY category, month, year
),
```

## Slide Deck

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vTKB5jVXYgu-1Zx3hBP7P4B9aP--KOTtlZMrPqKd593IhCybqdjBm3g19BCjaMYRgHWUQL-_M2NKCdT/embed?start=false&loop=false&delayms=3000" frameborder="0" width="100%" height="400" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

[thelook]:https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce