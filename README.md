[README.md](https://github.com/user-attachments/files/35155167/README.md)
# E-commerce Funnel & Customer Retention Analysis (Olist)

**Business question:** Where do orders get lost, and why do so few customers come back after their first purchase?

**Tools:** SQL (SQLite), Power BI, Excel/Google Sheets  
**Data:** [Olist Brazilian E-Commerce Public Dataset (Kaggle)](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), about 99K orders placed 2016-2018

![Dashboard](dashboard/screenshots/dashboard.png)

---

## Key findings

1. **Fulfilment is healthy; retention is the real problem.** 97.0% of orders reach delivery (99,441 placed → 96,476 delivered). Only **[X]%** of customers ever place a second order.
2. **Order losses are small and sit after approval.** Of the ~3% of orders lost, about 55% drop between approval and carrier handoff, about 40% between shipping and delivery, and only 5% at payment approval.
3. **First-order review score does not predict repeat purchase.** Repeat rate was 3.05% for 1-star vs 3.31% for 5-star first orders, a gap within noise.
4. **Late delivery has a real but modest effect.** Late first orders repeat 27% less often (2.18% vs 2.99%; two-proportion z-test, z ≈ 3, p ≈ 0.003). Fixing it would move the overall repeat rate by only about 0.06 points.
5. **Product category matters most.** Repeat rate by first-order category ranges from **0.9% (musical instruments) to 8.4% (home appliances)**. Among large categories, bed_bath_table (4.7%, n=5,370) and fashion_bags_accessories (5.4%, n=1,140) repeat well above the ~3% average, while electronics (1.5%) and telephony (1.9%) sit below it.

## Order fulfilment funnel

| Stage | Orders | Lost vs previous stage |
|---|---|---|
| Placed | 99,441 | - |
| Approved | 99,281 | 160 (0.16%) |
| Shipped | 97,658 | 1,623 (1.63%) |
| Delivered | 96,476 | 1,182 (1.19%) |

This is an order-fulfilment funnel (it starts at order placement), not a browse-to-purchase conversion funnel.

## Recommendation

- **Focus re-engagement spend on home and household categories**, where first-time buyers return at roughly 1.5x the average rate. Treat electronics and phones as acquisition-led categories.
- **Treat late-delivery recovery as a second-priority lever**: the effect is statistically real but small in total impact.

## Proposed A/B test (not run)

See [`ab_test_design.md`](ab_test_design.md). In brief: randomise first-time home and household buyers at delivery to receive a post-delivery cross-sell offer or nothing (holdout). Primary metric is 60-day repeat purchase. Guardrails are contribution margin after coupon cost and complaint rate. The design includes a sample-size calculation and a pre-set decision rule.

## Method

| Step | What I did | File |
|---|---|---|
| 1 | Explored the data and checked date ranges and order statuses | `sql/01_explore.sql` |
| 2 | Built the order fulfilment funnel | `sql/02_funnel.sql` |
| 3 | Measured repeat purchase using `customer_unique_id` | `sql/03_repeat_rate.sql` |
| 4 | Built cohort retention tables | `sql/04_cohorts.sql` |
| 5 | Tested repeat rate by first-order review score | `sql/05_review_score.sql` |
| 6 | Tested repeat rate by first-order product category | `sql/06_category.sql` |
| 7 | Tested repeat rate by late vs on-time delivery | `sql/07_late_delivery.sql` |

**Data handling notes**
- `customer_id` is generated per order, so customers are identified by `customer_unique_id`.
- A customer's first order is isolated with `ROW_NUMBER() OVER (PARTITION BY customer_unique_id ORDER BY order_purchase_timestamp)`.
- Only delivered orders are used in repeat-rate analysis.
- Blank timestamps imported as empty strings are filtered before date comparisons.
- Orders can contain several items, so one item per order (the first) is used to assign a category and avoid counting a customer twice.
- Category results use a minimum of 200 customers per category.

## How to reproduce

1. Download the dataset from Kaggle (link above).
2. Import the CSVs into SQLite (for example with DB Browser for SQLite) and rename the tables: `orders`, `customers`, `items`, `reviews`, `products`, `category_translation`.
3. Run the files in `sql/` in numeric order.
4. Open `dashboard/retention_dashboard.pbix` in Power BI Desktop (Windows) and point it at the CSVs.

## Limitations

- Observational data from one marketplace over about two years; results show association, not causation.
- Customers who first bought late in the data had less time to return, which can lower repeat rates. [Add result of the date-filter re-run here if you complete it.]
- Customers are labelled by the category of their first order's first item; a repeat purchase may be in a different category.
- Small categories (for example home_appliances, n=275) have wide uncertainty ranges and should be read as indicative.
- The A/B test is a proposed design; no live experiment was run.

## Author

**Shashank**: IIT BHU (2020), transitioning into product and data analytics  
[LinkedIn](https://www.linkedin.com/in/your-profile) · Bengaluru, India
