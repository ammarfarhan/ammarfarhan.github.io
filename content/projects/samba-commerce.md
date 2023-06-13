---
title: "Samba Commerce Dashboard"
date: 2023-03-31
draft: false
author: "Ammar Farhan Nur Hakim"
tags:
    - Blog
    - Customer Segmentation
    - Data Visualization
    - EDA
    - Tableau
image: /images/samba-commerce/main.jpg
description: ""
toc:
---

## Description

I created a dashboard for Samba Commerce, a dummy e-commerce company in Brazil.

Samba Commerce Dashboard is a comprehensive dashboard that serves as a company-wide tool for **tracking and analyzing the business performance** of Samba Commerce, including **customer RFM segmentation**.

### Project Details
| Type       | Tools    |
| ---------- | -------- |
| RevoU Assignment | Tableau  |

## Business Background

You are a data analyst at one of the fast growing ecommerce in Brazil named Samba Commerce, which just launched in 2021. Samba Commerce offers more than 15k products, and more than 50 product categories to the customers all over Brazil. There are a number of internal and external variables that support our business growth. We want to utilize more data to support our growth in 2022.

Mr. Ronaldinho (CEO), requested Mr Neymar (head of data) to create a company wide dashboard. Mr Neymar trust you to create the dashboard.

> “Olá, as per my explanation in our Analytics town hall our Company’s **main OKR** (Objectives and Key Results) this year is about **transactions**. So Mr. CEO requested a dashboard for *company-wide* to understand our business performance better. The bigger picture is there in our current dashboard, I need your help to capture more metrics and revamp our dashboard to be **more comprehensive and insightful.**”

## Dashboard Objectives

Build a **company-wide** dashboard used for monitoring the business performance of Samba Commerce.

- **Who**
    - The dashboard will be used by **various stakeholders within the company**, including executives, managers, and analysts.
- **Why**
    - The dashboard provides them with the data they need **to get insights about the company’s performance**.

## Data Preparation

1. Connect to the data source, and import the datasets. I cannot provide the source link for the data due to privacy restrictions, but here is the data dictionary for the datasets: 
    <br> <img src='/images/samba-commerce/data-dictionary.png' alt='Samba Commerce data dictionary'>
2. Make relationship between each dataset so the values can be aggregated without merging the dataset.
    <br> <img src='/images/samba-commerce/relation.png' alt='Samba Commerce dataset relationship'>

## Dashboard
There are 2 menus for this dashboard :
1. Home
2. Customer RFM Segmentation

<a href="https://public.tableau.com/app/profile/ahanaki/viz/SambaCommerce_16799043425600/DashboardHome" target="_blank">>> Link to the dashboard <<</a>

### Home
Home dashboard is opened by default, and can be accessed by clicking the Home icon in the navigation on the left side.

<img src='/images/samba-commerce/dashboard-1.png' alt='Samba Commerce home dashboard'>

1. Dashboard title, and filter controls. 
    <br> <img src='/images/samba-commerce/header.png' alt='Samba Commerce header'>
    - User can filter the data by order purchase date, customer state, customer city, and product category.
2. Scorecards:
    <br> <img src='/images/samba-commerce/scorecards.png' alt='Samba Commerce scorecards'>
    - Total Revenue by summing the Total Payment Value.
    - Average Order Value (AOV) by averaging the Total Payment Value.
    - Orders by counting the distinct Order Id.
    - Customers by counting the distinct Customer Id.
    - Sellers by counting the distinct Seller Id.
3. Customer location demography using map chart.
    - Map chart created by dual-axis method, showing data for each state.
        <br> <img src='/images/samba-commerce/chart-1.png' alt='Samba Commerce chart 1'>
        - The first is for showing number of customers visualized with filled-map. The darker the green, the more the customers. The details are shown in the tooltip. 
            <br> <img src='/images/samba-commerce/tooltip-1.png' alt='Samba Commerce map tooltip 1'>
        - The second is for showing total revenue visualized with circle. Bigger circle means bigger revenue. The details are shown in the tooltip.
            <br> <img src='/images/samba-commerce/tooltip-2.png' alt='Samba Commerce map tooltip 2'>
    - Can also switch the map to show the data for each zip code by switching the map from Default to Zip Code in the menu on top-right of the chart.
        <br> <img src='/images/samba-commerce/chart-5.png' alt='Samba Commerce chart 5'>
        - The details are shown in the tooltip.
            <br> <img src='/images/samba-commerce/tooltip-3.png' alt='Samba Commerce map tooltip 3'>
4. Monthly sales using combo chart (dual-axis method).
    - The chart is showing how many orders and how much revenue generated every month.
    <br> <img src='/images/samba-commerce/chart-2.png' alt='Samba Commerce chart 2'>
5. Orders by item quantity using bar chart.
    - Since there are only a small number of transactions involving more than 6 items, they are grouped together.
    <br> <img src='/images/samba-commerce/chart-3.png' alt='Samba Commerce chart 3'>
6. Top N sellers by orders using bar chart. 
    - User can customize the result by changing the Top Sellers value in the top-right of the chart. The range for the value is 10 to 50.
        <br> <img src='/images/samba-commerce/chart-4.png' alt='Samba Commerce chart 4'>

### Customer RFM Segmentation
The dashboard can be accessed by clicking the Group icon in the navigation on the left side.

<img src='/images/samba-commerce/dashboard-2.png' alt='Samba Commerce RFM dashboard'>

1. Dashboard title, and filter controls. 
    - User can filter the data by the segmentation, recency score, frequency score, monetary score, and order purchase date. 
    - The highest score is 1 and the lowest score is 4.
2. RFM Customer Segmentation
    - A total of 10 segments created from the customers RFM score.
    - The charts visualized the segment size, average recency, average frequency, and averagy monetary for every segment.

## Slide Deck Example

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vRw9mZhHXVAVCsYEu6tzJk3N8saiDWN4usFugJT3WbZUm5jhBtTiYg1xWTSsLyyKeznvRRuQV9JW4FK/embed?start=false&loop=false&delayms=3000" frameborder="0" width="100%" height="400" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

---

Thank you for taking the time to visit my portfolio website and view my projects!

I appreciate your interest and support. If you have any feedback or questions, please don't hesitate to reach out to me. Once again, thank you for your time and consideration.

<a href="https://www.linkedin.com/in/ahanaki/" target="_blank">>> Let's connect on LinkedIn! <<</a>

<a href="https://public.tableau.com/app/profile/ahanaki" target="_blank">>> Visit my Tableau Public profile to see my other dashboards <<</a>