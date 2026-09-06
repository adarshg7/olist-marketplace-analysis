# Olist Marketplace Performance Analysis
### Data Analytics Hackathon — Conducted by Gradient

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Business Background](#business-background)
3. [Problem Statement](#problem-statement)
4. [Dataset Description](#dataset-description)
5. [Data Dictionary](#data-dictionary)
6. [Project Structure](#project-structure)
7. [Setup and Installation](#setup-and-installation)
8. [Analytical Methodology](#analytical-methodology)
9. [Core Analysis — Six Questions](#core-analysis--six-questions)
10. [Deep-Dive Analysis](#deep-dive-analysis)
11. [Key Findings](#key-findings)
12. [Business Recommendations](#business-recommendations)
13. [Visualizations](#visualizations)
14. [Limitations and Assumptions](#limitations-and-assumptions)
15. [Technologies Used](#technologies-used)
16. [Author](#author)

---

## Project Overview

This project presents a comprehensive, end-to-end data analysis of the Olist Brazilian E-Commerce Public Dataset. The analysis was submitted as part of the **Data Analytics Hackathon conducted by Gradient**.

The goal is to move beyond exploratory data analysis and deliver a structured, evidence-based business investigation — one that identifies what is genuinely driving customer satisfaction and dissatisfaction on the platform, quantifies the magnitude of each driver, and translates findings into prioritized, actionable recommendations for Olist's operations and customer experience leadership.

The dataset covers approximately 100,000 real commercial orders placed between September 2016 and October 2018, across 9 interconnected tables. This is not a synthetic dataset — it represents actual transactions from a live marketplace, complete with the data quality characteristics that real operational data carries.

---

## Business Background

Olist is a Brazilian e-commerce platform that functions as a marketplace integrator. Rather than negotiating with each major online marketplace individually, small and medium merchants across Brazil sign a single contract with Olist to list and sell their products across all major platforms simultaneously.

The operational model works as follows:

- A customer discovers and purchases a product through an Olist-connected store on a major marketplace
- The responsible seller receives a notification and fulfills the order using Olist's logistics partners
- The order is shipped and delivered to the customer
- After delivery — or after the estimated delivery date passes — the customer receives a satisfaction survey by email
- The customer can leave a score between 1 and 5 stars, with an optional written comment

This model creates a layered accountability structure. The customer experience depends on the seller (product quality, listing accuracy, fulfillment speed), on the logistics partner (shipping time and reliability), and on Olist's platform (delivery estimation, communication, dispute resolution). When something goes wrong, the review score captures the outcome — but not the cause.

---

## Problem Statement

Olist's leadership has two years of order history covering every dimension of the marketplace experience: what was ordered, from whom, by whom, how it was paid for, how long it took to arrive, and what the customer thought of the experience.

Despite this depth of data, no single consolidated analysis has connected these dimensions together. Review scores, delivery timing, seller performance, payment behavior, and geographic patterns all vary considerably — but leadership does not have a clear picture of which factors are primary drivers of satisfaction and which are secondary or incidental.

The specific questions leadership needs answered:

1. How has the marketplace actually performed over the available time period?
2. What factors are most strongly associated with customer satisfaction and dissatisfaction?
3. Which of those factors are primary drivers versus secondary contributors?
4. Where are the clearest opportunities to improve the platform experience as Olist continues to scale?

This analysis was conducted to answer exactly those questions, using evidence from the data rather than assumptions.

---

## Dataset Description

**Source:** Brazilian E-Commerce Public Dataset by Olist
**Original publication:** [Kaggle — olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
**License:** Creative Commons Attribution-NonCommercial-ShareAlike
**Anonymization:** Customer and seller identities are hashed. Company or partner names in review text have been replaced with fictional names.
**Period covered:** September 4, 2016 to October 17, 2018
**Total orders:** 99,441

The dataset consists of 9 interrelated tables:

| Table | Rows | Grain | Primary Key |
|---|---|---|---|
| olist_orders_dataset | 99,441 | 1 row per order | order_id |
| olist_order_items_dataset | 112,650 | 1 row per item within an order | order_id + order_item_id |
| olist_order_payments_dataset | 103,886 | 1 row per payment record | order_id + payment_sequential |
| olist_order_reviews_dataset | 100,000 | 1 row per review | review_id |
| olist_customers_dataset | 99,441 | 1 row per order-level customer | customer_id |
| olist_products_dataset | 32,951 | 1 row per product | product_id |
| olist_sellers_dataset | 3,095 | 1 row per seller | seller_id |
| olist_geolocation_dataset | 1,000,163 | Multiple rows per zip prefix | none |
| product_category_name_translation | 71 | 1 row per category pair | product_category_name |

**Important notes on the data:**
- 96,478 of 99,441 orders (97.0%) have status `delivered`. The remaining orders are in states such as `shipped`, `canceled`, `unavailable`, `invoiced`, `processing`, `created`, or `approved`. Delivery-time analysis is restricted to delivered orders only.
- `customer_id` is unique per order, not per person. `customer_unique_id` identifies the same person across multiple orders. Using `customer_id` to count unique customers overstates the true customer count.
- About 13,984 orders have more than one item row. About 4,446 orders have more than one payment row. Row counts on these tables do not equal order counts.
- The geolocation table has approximately 52 rows per unique zip code prefix. Joining without prior aggregation multiplies rows unexpectedly.

---

## Data Dictionary

### orders
| Column | Type | Description |
|---|---|---|
| order_id | string (PK) | Unique identifier for the order |
| customer_id | string (FK) | Order-level customer identifier — links to customers table |
| order_status | string | Current status: delivered / shipped / canceled / unavailable / invoiced / processing / created / approved |
| order_purchase_timestamp | datetime | When the order was placed by the customer |
| order_approved_at | datetime (nullable) | When payment was approved — missing for ~160 orders |
| order_delivered_carrier_date | datetime (nullable) | When the order was handed to the carrier — missing for ~1,783 orders |
| order_delivered_customer_date | datetime (nullable) | When the customer received the order — missing for ~2,965 orders |
| order_estimated_delivery_date | datetime | The delivery date promised to the customer at purchase time |

### order_items
| Column | Type | Description |
|---|---|---|
| order_id | string (FK) | Links to orders table |
| order_item_id | int | Sequence number of the item within the order |
| product_id | string (FK) | Links to products table |
| seller_id | string (FK) | Links to sellers table |
| price | float | Price of this item in BRL |
| freight_value | float | Shipping cost allocated to this item in BRL |

### order_payments
| Column | Type | Description |
|---|---|---|
| order_id | string (FK) | Links to orders table |
| payment_sequential | int | Sequence number when an order has multiple payment records |
| payment_type | string | credit_card / boleto / voucher / debit_card / not_defined |
| payment_installments | int | Number of installments selected by the customer |
| payment_value | float | Amount paid in this payment record in BRL |

### order_reviews
| Column | Type | Description |
|---|---|---|
| review_id | string (PK) | Unique review identifier |
| order_id | string (FK) | Links to orders table |
| review_score | int | Star rating from 1 to 5 |
| review_comment_title | string (mostly null) | Optional short title — ~88% missing |
| review_comment_message | string (often null) | Optional written comment — ~58% missing |
| review_creation_date | date | When the survey was sent to the customer |
| review_answer_timestamp | datetime | When the customer submitted the review |

### customers
| Column | Type | Description |
|---|---|---|
| customer_id | string (PK) | Order-level identifier — unique per order, NOT per person |
| customer_unique_id | string | Person-level identifier — shared across a person's repeat orders |
| customer_zip_code_prefix | int | First 5 digits of the customer's zip code |
| customer_city | string | Customer's city name |
| customer_state | string | Customer's state as a two-letter code |

### products
| Column | Type | Description |
|---|---|---|
| product_id | string (PK) | Unique product identifier |
| product_category_name | string (nullable) | Category in Portuguese — ~610 products missing |
| product_name_lenght | float (nullable) | Character length of the product name |
| product_description_lenght | float (nullable) | Character length of the product description |
| product_photos_qty | float (nullable) | Number of product photos in the listing |
| product_weight_g | float (mostly present) | Product weight in grams |
| product_length_cm | float (mostly present) | Product length in centimeters |
| product_height_cm | float (mostly present) | Product height in centimeters |
| product_width_cm | float (mostly present) | Product width in centimeters |

### sellers
| Column | Type | Description |
|---|---|---|
| seller_id | string (PK) | Unique seller identifier |
| seller_zip_code_prefix | int | First 5 digits of the seller's zip code |
| seller_city | string | Seller's city name |
| seller_state | string | Seller's state as a two-letter code |

### geolocation
| Column | Type | Description |
|---|---|---|
| geolocation_zip_code_prefix | int | Zip code prefix — NOT unique |
| geolocation_lat | float | Latitude reading |
| geolocation_lng | float | Longitude reading |
| geolocation_city | string | City name for this reading |
| geolocation_state | string | State for this reading |

### product_category_name_translation
| Column | Type | Description |
|---|---|---|
| product_category_name | string (PK) | Category name in Portuguese — join key to products |
| product_category_name_english | string | Category name in English |

---

## Project Structure

```
olist-marketplace-analysis/
│
├── analysis.ipynb               # Complete analysis notebook (21 cells, runs top to bottom)
├── README.md                    # This file
├── requirements.txt             # Python dependencies
│
└── charts/                      # All generated visualization outputs
    ├── q1_trends.png            # Order volume, GMV, and review scores over time
    ├── q2_delivery_score.png    # Delivery timing vs review scores (3-panel)
    ├── q2_by_state.png          # State-level delay vs satisfaction scatter
    ├── q3_geography.png         # Seller/customer distribution + freight by state
    ├── q4_categories.png        # Category performance across 4 dimensions
    ├── q5_payments.png          # Payment behaviour analysis (4-panel)
    ├── q6_root_cause.png        # Root cause analysis (4-panel)
    ├── extra_cross_state.png    # Same-state vs cross-state order comparison
    ├── extra_freight_ratio.png  # Freight as % of order value vs satisfaction
    ├── extra_seller_clusters.png # K-means seller segmentation
    └── extra_repeat_customers.png # Repeat purchase behaviour
```

---

## Setup and Installation

### Prerequisites
- Python 3.8 or higher
- pip

### Option 1 — Google Colab (Recommended)

1. Upload the dataset Excel file to your Google Drive
2. Open `analysis.ipynb` directly in Google Colab via the GitHub link or by uploading
3. In **Cell 2**, update the file path:
   ```python
   FILE = '/content/drive/MyDrive/Document from singhlink4.xlsx'
   ```
4. Run all cells in order from top to bottom
5. Charts will be saved in the current working directory

### Option 2 — Local Environment

```bash
# Clone the repository
git clone https://github.com/yourusername/olist-marketplace-analysis.git
cd olist-marketplace-analysis

# Install dependencies
pip install -r requirements.txt

# Place the dataset Excel file in the Dataset/ folder
# Update the FILE path in Cell 2 of the notebook if needed

# Launch Jupyter
jupyter notebook analysis.ipynb
```

### Dataset File

The dataset is available on Kaggle:
[https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

In this project the 9 CSV files were provided as a single consolidated Excel workbook with one sheet per table. If you are using the original Kaggle CSV files, Cell 2 of the notebook will need to be updated to load each file separately using `pd.read_csv()`.

---

## Analytical Methodology

The analysis follows a root-cause investigation framework rather than open-ended exploration.

**Step 1 — Data Understanding**
Examine the shape, types, missing value patterns, and distributions of every table before any transformation. Understand what each missing value means in operational terms, not just statistically.

**Step 2 — Data Cleaning and Feature Engineering**
- Parse all datetime columns with error coercion
- Restrict delivery-time analysis strictly to orders with `order_status == 'delivered'`
- Compute delivery delay in days: `order_delivered_customer_date - order_estimated_delivery_date` (positive = late, negative = early)
- Compute actual days to deliver: `order_delivered_customer_date - order_purchase_timestamp`
- Classify delay into meaningful buckets: early, on time, 3–7 days late, 7–14 days late, 14+ days late
- Enrich products with English category names via the translation table
- Aggregate payments to order level (summing payment_value, selecting dominant payment type, taking max installments)
- Aggregate order items to order level (summing price and freight to get order value and total freight)
- Build a master dataset joining delivered orders with reviews, customers, payments, and revenue

**Step 3 — Exploratory Analysis**
Examine distributions, time trends, geographic patterns, and category-level summaries to build context before testing specific hypotheses.

**Step 4 — Core Question Investigation**
Each of the six core questions is treated as a separate analytical module with its own data preparation, statistical analysis, and visualization.

**Step 5 — Root Cause Analysis**
Use Pearson correlation, conditional probability analysis, and group comparisons to rank factors by their association with low review scores. Distinguish primary drivers (strong, consistent association) from secondary contributors (present but weaker or conditional).

**Step 6 — Deep-Dive Extensions**
Eight additional analytical angles explore hypotheses that the core questions do not fully address — including seller segmentation via K-Means clustering, delivery promise accuracy over time, freight-to-price ratio effects, and cohort-based repeat purchase analysis.

---

## Core Analysis — Six Questions

### Question 1 — Marketplace Performance Over Time

**What was investigated:**
Monthly trends in order volume, gross merchandise value (GMV), and average review score between January 2017 and August 2018. The analysis excludes September–December 2016 (too few orders; platform was newly launched) and September–October 2018 (incomplete months at the dataset boundary).

**Method:**
- Aggregate orders, item revenue, and review scores by purchase month
- Plot all three on aligned time series
- Measure growth rates and identify divergences between volume/revenue trends and satisfaction trends

**Why this matters:**
A marketplace can show strong top-line growth while silently accumulating a satisfaction problem. If review scores are flat or declining while volume grows, the platform is scaling its problems alongside its revenue.

---

### Question 2 — Delivery Performance and Customer Satisfaction

**What was investigated:**
The relationship between delivery timing (actual delivery date vs estimated delivery date) and review scores. Analysed at the platform level, by state, and by product category.

**Method:**
- Compute delivery delay in days for all delivered orders
- Classify delay into 6 buckets
- Calculate average score, % low scores (1–2 stars), and order count per bucket
- Compute Pearson correlation between continuous delay days and review score
- Calculate conditional probabilities: P(low score | late) vs P(low score | on time)
- Scatter plot of state-level avg delay vs avg score, sized by order volume

**Why this matters:**
Delivery timing is the most controllable post-purchase variable. If it is the primary driver of satisfaction, it is also the highest-leverage point for improvement — because improving delivery is operationally achievable, unlike improving the product or the customer's taste.

---

### Question 3 — Seller and Geographic Patterns

**What was investigated:**
The distribution of sellers and customers across Brazilian states, how this relates to freight costs and delivery performance, and whether the geographic imbalance between where sellers are and where customers live explains the delivery problem identified in Question 2.

**Method:**
- Calculate seller share and customer share per state
- Identify imbalance: states where customer demand far exceeds seller presence
- Compute average freight cost per item by customer state
- Build scatter plot of seller state vs avg delay and avg review score
- Compare same-state vs cross-state orders on delay, freight, and satisfaction

**Why this matters:**
If the delivery problem is geographic in origin — caused by sellers being concentrated in one region while customers are spread across a vast country — then the solution is not purely logistical. It requires a platform strategy change: recruiting sellers closer to customers.

---

### Question 4 — Product Category Performance

**What was investigated:**
How order volume, average price, total revenue, and average review score differ across product categories, and which categories are notably above or below the platform average.

**Method:**
- Join order items with products (English categories) and review scores from the master dataset
- Aggregate to category level, filtering to categories with at least 100 orders
- Compute score deviation from platform average for each category
- Visualize top 20 by volume, top 15 by revenue, best and worst 12 by score, and a price-vs-score bubble chart

**Why this matters:**
Some categories may have structurally worse delivery performance due to product weight or dimensions. Others may attract customers with higher expectations. Identifying which categories underperform allows Olist to target seller quality improvement efforts precisely.

---

### Question 5 — Payment Behavior

**What was investigated:**
How payment type (credit card, boleto, voucher, debit card) and installment count relate to order value, and whether payment behavior is connected to customer satisfaction.

**Method:**
- Aggregate payments to order level with dominant payment type and maximum installments
- Compare order count, average order value, and average review score by payment type
- Analyse credit card installment count vs average order value (to understand installments as an affordability mechanism)
- Test whether payment type has an independent association with review scores

**Why this matters:**
If customers using certain payment types have systematically different satisfaction levels, it may indicate different customer segments, purchase types, or risk profiles. If payment type has no independent effect, it can be deprioritized as a lever.

---

### Question 6 — Root Cause Analysis

**What was investigated:**
A systematic attempt to identify which factors are primary drivers of low review scores (1–2 stars) versus secondary contributors, using multiple analytical approaches.

**Method:**
- Pearson correlation of all continuous features with review score
- Conditional probability: P(low score | late) vs P(low score | on time)
- Relative risk calculation: how much more likely is a low score if the order is late
- % low-score rate by delivery delay bucket
- % low-score rate by customer state
- Late vs on-time low score rate comparison across top states
- Quantification of how many low-score orders would be avoided if late deliveries became on-time

**Why this matters:**
Identifying a driver is not enough. The analysis must distinguish drivers that are primary (strong, consistent, platform-wide) from those that are secondary (conditional, category-specific, or confounded). This distinction determines what leadership should prioritize.

---

## Deep-Dive Analysis

### 1. Delivery Promise Accuracy Over Time
Examines whether Olist's delivery estimation system is improving over the analysis period. Rather than just measuring lateness, this tracks the systematic bias in estimates — whether Olist is consistently under-promising or over-promising delivery times, and whether that bias is changing.

### 2. Seller Behavioral Segmentation
Uses K-Means clustering (k=4) on seller-level metrics — average score, average delay, % late, average price, category breadth, and order volume — to identify distinct seller archetypes. This allows targeted interventions for underperforming segments rather than platform-wide policies.

### 3. Freight-to-Price Ratio as a Satisfaction Driver
Introduces a derived metric: freight as a percentage of order value. Tests whether orders where shipping costs are disproportionately high relative to product value generate lower satisfaction — even when delivery is on time. A customer paying R$30 to ship a R$20 product has a fundamentally different experience from one paying R$30 to ship a R$300 product.

### 4. Black Friday Stress Test
Isolates November 2017 — the platform's highest-volume month — and compares it to surrounding months on delivery delay, % late orders, and average review score. Tests whether the logistics infrastructure degrades under demand spikes.

### 5. Same-State vs Cross-State Order Performance
Compares orders where the seller and customer are in the same state against orders that cross state lines, on delay, freight, score, and % low scores. Directly tests whether the geographic concentration of sellers is the mechanism behind the delivery problem.

### 6. Review Response Speed as a Sentiment Signal
Measures the time between when the survey was sent and when the customer submitted the review. Tests whether very fast responses (under 2 hours) are associated with extreme scores (very high or very low), which would suggest an immediate emotional response rather than a considered one. This has implications for real-time alert systems.

### 7. Product Listing Quality vs Review Score
Uses product-level attributes — number of photos, description length, and name length — as proxies for listing quality. Tests whether better-described products generate higher review scores, which would suggest that customer expectations are being shaped at the listing stage.

### 8. Repeat Customer Cohort Analysis
Uses `customer_unique_id` to identify true repeat buyers. Builds cohort-level retention curves by first-purchase month, calculates the distribution of days between first and second purchase (the natural re-engagement window), and compares average order value between one-time and repeat buyers.

---

## Key Findings

### Finding 1 — Growth and Satisfaction Are Diverging

Order volume and GMV grew approximately 3x between January 2017 and August 2018, reflecting strong platform adoption. Average review scores remained largely flat across the same period. The platform is scaling its volume faster than it is scaling its quality.

### Finding 2 — Delivery Timing Is the Primary Driver of Satisfaction

- Pearson r (delivery delay vs review score) ≈ -0.32, statistically significant (p < 0.001)
- Orders delivered on time or early: approximately 8–10% low-score rate
- Orders delayed 14+ days: 50%+ low-score rate
- Relative risk: a late order is 4–5x more likely to receive a 1–2 star review than an on-time order
- Delivery delay has the strongest correlation with review score of any variable tested, including order value, number of items, freight cost, and payment installments

### Finding 3 — Geographic Imbalance Is the Structural Root Cause

- Approximately 70% of Olist's sellers operate from São Paulo state
- Customers are distributed across all 26 Brazilian states
- Cross-state orders show significantly higher average delays, higher average freight costs, and lower average review scores compared to same-state orders
- States in the North and Northeast (AM, AC, RR, AP, PA, MA) experience the longest delays and pay 2–3x more in freight than states in the South and Southeast
- This is not a courier failure — it is a structural consequence of the seller-customer geographic mismatch

### Finding 4 — Category Performance Varies Significantly

- Health and beauty, sports and leisure, and housewares perform above the platform average
- Office furniture, computers, and certain electronics categories consistently underperform
- Heavier and larger product categories correlate with higher freight costs, longer delivery times, and lower review scores — suggesting a compound effect of product type and logistics difficulty

### Finding 5 — Payment Type Does Not Independently Drive Satisfaction

- Review scores are consistent across credit card, boleto, and voucher payment types
- Higher installment counts correlate with higher order values, confirming that installments function as an affordability mechanism for larger purchases
- Payment behavior is not a meaningful lever for satisfaction improvement

### Finding 6 — Repeat Purchase Rate Is Low But Repeat Buyers Are More Valuable

- Approximately 3% of customers placed more than one order during the dataset period
- The median time between a customer's first and second purchase is approximately 120–180 days
- Repeat buyers generate higher average order values than one-time buyers
- The low repeat rate may reflect insufficient time in the dataset window rather than pure dissatisfaction, but the gap is large enough to warrant attention

---

## Business Recommendations

| Priority | Recommendation | Evidence Basis |
|---|---|---|
| Critical | Reduce delivery lateness — with particular focus on eliminating the 14+ day tail | 14+ day late orders have 50%+ low-score rate; this tail disproportionately damages the platform's NPS |
| Critical | Recruit sellers in underserved high-demand states — MG, RJ, RS, BA, PR | ~70% of sellers in SP creates structural cross-state delays; same-state orders perform significantly better |
| High | Set more conservative delivery estimates for cross-state routes | Over-promising and under-delivering destroys satisfaction even for orders that are only slightly late |
| High | Improve logistics partner SLAs for North and Northeast corridors | These regions show the largest delays and highest freight costs — dedicated agreements are needed |
| High | Flag and address listings where freight exceeds 50% of product value | High freight-to-price ratio independently degrades satisfaction, even on on-time deliveries |
| Medium | Launch a re-engagement communication at 30 days post first purchase | Natural return window is 120–180 days; early nudging can shift the curve and improve repeat rate |
| Medium | Mandate minimum 3 product photos per listing | Listing quality (photos, description length) shows a positive correlation with review scores |
| Medium | Implement seller performance tiers with defined SLAs | K-Means segmentation identifies a distinct cluster of underperforming sellers who are disproportionately responsible for low scores |
| Low | Build a real-time alert for reviews submitted within 2 hours of survey creation | Very fast reviews skew toward extreme scores — early identification allows proactive customer recovery |

---

## Visualizations

All charts are generated and saved automatically when the notebook is run.

| File | Description |
|---|---|
| q1_trends.png | Three-panel time series: monthly order count, GMV, and average review score (Jan 2017 – Aug 2018) |
| q2_delivery_score.png | Three-panel: avg score by delay bucket, % low scores by delay bucket, scatter of delay vs score with binned trend line |
| q2_by_state.png | Top 15 states by avg delivery delay (bar) and state-level delay vs score scatter (bubble = order volume) |
| q3_geography.png | Customer vs seller share by state (grouped bar) and avg freight by customer state (horizontal bar) |
| q4_categories.png | Four-panel: top 20 by volume, top 15 by revenue, best and worst by score, price vs score bubble chart |
| q5_payments.png | Four-panel: order count, avg value, installments vs value, and avg score by payment type |
| q6_root_cause.png | Four-panel: feature correlations, % low scores by delay bucket, % low scores by state, late vs on-time comparison |
| extra_cross_state.png | Four-metric comparison of same-state vs cross-state orders |
| extra_freight_ratio.png | Review score and % low scores by freight-to-price ratio bucket |
| extra_seller_clusters.png | K-Means seller segmentation scatter (delay vs score, bubble = volume) |
| extra_repeat_customers.png | Frequency distribution, revenue pie, and avg order value for repeat vs one-time customers |

---

## Limitations and Assumptions

**1. Observational data only**
All associations identified are correlational. This analysis does not prove causation. Delivery lateness is identified as the primary driver because of the strength, consistency, and magnitude of its association with low scores — not because a controlled experiment was run.

**2. Dataset time window**
The dataset covers September 2016 to October 2018. Growth trends, seller behavior, and customer preferences may have changed significantly after this period. Findings should be interpreted in the context of this specific time window.

**3. Geolocation approximation**
The geolocation table has multiple lat/lng readings per zip code prefix. Mean coordinates per prefix were used for any geographic calculations. This introduces a small approximation but is standard practice for this dataset.

**4. Seller-per-order assignment**
For orders with multiple items from different sellers, the dominant seller state (mode) was used to assign a single seller state to the order. This is an approximation for the ~14% of orders with multiple items.

**5. Review coverage**
Not all orders have reviews. The review table contains 100,000 rows for 99,441 orders, but some orders have no review record. Analysis of review scores implicitly excludes unreviewed orders. If unreviewed orders skew toward a particular satisfaction level, this introduces selection bias.

**6. Delivery delay calculation**
Delivery delay is calculated as `order_delivered_customer_date - order_estimated_delivery_date`. The estimated delivery date is set at the time of purchase. It is not known whether this estimate is generated by Olist's algorithm, provided by the seller, or sourced from the logistics partner.

---

## Technologies Used

| Technology | Version | Purpose |
|---|---|---|
| Python | 3.8+ | Core programming language |
| pandas | Latest | Data manipulation and aggregation |
| numpy | Latest | Numerical computation |
| matplotlib | Latest | Static chart generation |
| seaborn | Latest | Statistical visualization |
| scipy | Latest | Pearson correlation and statistical tests |
| scikit-learn | Latest | K-Means clustering for seller segmentation |
| openpyxl | Latest | Reading Excel workbook format |
| Jupyter Notebook | Latest | Interactive analysis environment |

---

Submitted for the **Data Analytics Hackathon conducted by Gradient**

---

*This analysis uses the Brazilian E-Commerce Public Dataset by Olist, published on Kaggle under a Creative Commons Attribution-NonCommercial-ShareAlike license. Please verify current license terms on the Kaggle page before any commercial or redistribution use.*
