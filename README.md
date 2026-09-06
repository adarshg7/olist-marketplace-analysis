# Olist Marketplace Performance Analysis

Submitted as part of the **Data Analytics Hackathon conducted by Gradient.**

---

## Business Context

Olist is a Brazilian e-commerce platform that connects small and medium merchants
to major online marketplaces through a single contract. When a customer places an
order through an Olist-connected store, the seller fulfills and ships it using
Olist's logistics partners. After delivery, customers receive a satisfaction survey
asking for a 1 to 5 star review.

Olist's leadership has two years of order history (September 2016 to October 2018)
covering orders, items, payments, products, sellers, customers, reviews, and
geolocation data. No single consolidated analysis has connected these dimensions.
This project does exactly that.

---

## Objective

Determine how the marketplace has performed over the available time period, what
factors are associated with customer satisfaction and dissatisfaction, where
performance varies most across sellers, geography, categories, and payment
behavior, and what Olist should prioritize as a result.

---

## Dataset

**Source:** Brazilian E-Commerce Public Dataset by Olist
**Originally published on:** [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
**License:** Creative Commons Attribution-NonCommercial-ShareAlike
**Period:** September 2016 – October 2018
**Scale:** ~100,000 orders across 9 interconnected tables

| File | Rows | Description |
|---|---|---|
| orders | 99,441 | Core order records with status and timestamps |
| order_items | 112,650 | Line-item detail per order |
| order_payments | 103,886 | Payment records per order |
| order_reviews | 100,000 | Customer review scores and comments |
| customers | 99,441 | Customer location per order |
| products | 32,951 | Product catalog with dimensions |
| sellers | 3,095 | Seller location data |
| geolocation | 1,000,163 | Zip code prefix to lat/lng mappings |
| category_translation | 71 | Portuguese to English category names |

---

## Core Questions Investigated

1. **Marketplace Performance Over Time** — How have order volume, revenue, and
   review scores trended across the available period?

2. **Delivery Performance and Customer Satisfaction** — How does delivery timing
   relative to the estimated date relate to review scores?

3. **Seller and Geographic Patterns** — How does the geographic distribution of
   sellers and customers relate to delivery performance, freight cost, and
   satisfaction?

4. **Product Category Performance** — How do order volume, price, and review
   scores differ across product categories?

5. **Payment Behavior** — How do payment type and installment choices relate to
   order value and customer experience?

6. **Root Cause Analysis** — What are the primary drivers of low review scores,
   and what is the evidence for each?

---

## Additional Deep-Dive Analyses

- Delivery promise accuracy over time
- Seller behavioral segmentation using K-Means clustering
- Freight-to-price ratio as a hidden satisfaction driver
- Black Friday volume stress test (November 2017)
- Same-state vs cross-state order performance
- Review response speed as a real-time sentiment signal
- Product listing quality vs review score
- Repeat customer cohort retention analysis

---

## Repository Structure

