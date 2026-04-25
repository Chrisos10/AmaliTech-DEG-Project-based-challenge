# The "Last Mile" Logistics Auditor
### Delivery Performance Audit for Veridi Logistics

---

## A. Executive Summary

An audit of 96,470 delivered orders from the Olist Brazilian E-Commerce Dataset reveals that Veridi Logistics has two distinct problems, not one. At the national level, approximately 8.1% of orders arrive late, with Super Late deliveries (>5 days late) receiving an average review score of just 1.79/5 compared to 4.29/5 for on-time orders, directly confirming that late logistics is the primary driver of negative customer reviews. Geographically, the problem is concentrated in Northeastern states (AL at 23.9%, MA at 19.6%, CE at 15.4%), though paradoxically, the most remote northern states (AC, RO, AM) perform well because Veridi over-pads delivery estimates for those regions. Most critically, anomaly detection revealed a systemic nationwide logistics failure in March 2018, where 15 out of 16 major states simultaneously spiked 3–4 standard deviations above their normal late delivery rates before recovering in April 2018, suggesting a specific operational event such as a carrier strike or warehouse outage that has never been formally investigated.

---

## B. Project Links

| Deliverable | Link |
|---|---|
| Notebook (Google Colab) |https://colab.research.google.com/drive/1cHGKLEfa3LbFXIH8nfJ0rcHj6WUEg7Ca?usp=sharing|
| Dashboard (Streamlit) | https://amalitechassignment-jvugsfxj98iydovvrwpdug.streamlit.app/ |
| Presentation Slides | https://docs.google.com/presentation/d/1UePVeF3sddicYaLPmsVB0V-GLYLp4Fje/edit?usp=sharing&ouid=104186753624904840744&rtpof=true&sd=true |


---

## C. Technical Explanation

### Data Cleaning

The raw dataset is a relational database spread across multiple CSV files. The cleaning pipeline followed these steps:

**1. Joining**
The five tables were assembled into a single master dataset in the following order:
- `orders` + `customers` on `customer_id`
- `orders` + `reviews` on `order_id`
- `orders` + `order_items` (aggregated to one row per order) on `order_id`
- `order_items` + `products` on `product_id`
- `products` + `translations` on `product_category_name`

**2. Removing Duplicated**
Two steps were required to remove duplicated. The reviews join caused 551 duplicate rows (some orders had multiple reviews), resolved by keeping the most recent review per order. The translations join later introduced 529 additional duplicates, resolved by dropping duplicates on `order_id` keeping the first occurrence.

**3. Filtering**
Only orders with `order_status == 'delivered'` were retained (96,478 of 99,441 total). A further 8 rows with null `order_delivered_customer_date` were dropped, yielding a final clean dataset of **96,470 orders**.

**4. Feature Engineering**
- All date columns were converted from string to `datetime64`
- `days_difference = order_estimated_delivery_date - order_delivered_customer_date` (positive = early, negative = late)
- `delivery_status` classified as: On Time (≥0 days), Late (−5 to 0 days), Super Late (<−5 days)
- `is_late` binary flag (1 = Late or Super Late, 0 = On Time)
- `order_month` extracted from `order_purchase_timestamp` for time-series analysis

---

### Candidate's Choice: Delivery Anomaly Detection

**Feature:** Z-score based anomaly detection on monthly late delivery rates per state, with an interactive heatmap and drill-down trend chart.

**How it works:**
For each state with sufficient data volume (≥30 orders/month, ≥15 months of history, covering 16 states), the mean and standard deviation of the monthly late rate was calculated. Each month's late rate was then expressed as a Z-score (number of standard deviations above or below that state's normal). Any month with a Z-score above 2.0 was flagged as an anomaly.

**What it found:**
22 anomalies were detected across the dataset. Of these, 15 occurred in **March 2018**, simultaneously across nearly every major state. CE spiked to 53.8% late (normal: 12.9%), ES to 44.8% (normal: 9.7%), and PA to 52.2% (normal: 9.7%). All states recovered sharply in April 2018.

**Why it matters to the business:**
Standard reporting would tell a Regional Director that AL has a 23.9% late rate and needs fixing. Anomaly detection tells the CEO something more actionable: *a single operational event in March 2018 caused a nationwide failure that affected every major state simultaneously*. The sharp recovery in April suggests it was caused and resolved by a specific event. Without anomaly detection, this event remains invisible in aggregated statistics. With it, the business can investigate root cause, identify whether it has happened again in smaller form, and build an early warning system to catch the next one before it affects tens of thousands of customers.

---


## D. Technical Stack

| Tool | Purpose |
|---|---|
| Google Colab | Notebook environment |
| Python / Pandas | Data processing and feature engineering |
| Matplotlib / Seaborn | Exploratory charts in notebook |
| Streamlit | Interactive public dashboard |
| Plotly | Dashboard visualizations |
| Google Drive | Aggregated data hosting for dashboard |
| GitHub | Code repository and version control |
