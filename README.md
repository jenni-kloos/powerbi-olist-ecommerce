# 🛒 Brazilian E-Commerce Analytics | Power BI Dashboard

A Power BI report built on the **Olist Brazilian E-Commerce** dataset (Kaggle), covering four analytical domains: Sales, Logistics, Payments, and Customer Feedback. Built as a course project simulating a real analyst assignment — from raw CSV files to interactive dashboards.

---

## 📊 Dashboard Preview

| Sales | Logistics |
|---|---|
| ![Sales](screenshots/sales.png) | ![Logistics](screenshots/logistics.png) |

| Payments | Feedback |
|---|---|
| ![Payments](screenshots/payments.png) | ![Feedback](screenshots/feedback.png) |

> Includes a **drill-through page** (Product Details) and a **custom tooltip** (Order Funnel).

---

## 🔍 Key Business Insights & Recommendations

**Sales**
- SP dominates revenue (R$ 335K, 23K orders) — ~3× more than MG (R$ 112K). This geographic concentration risk in fulfillment operations means disruptions in the Southeast disproportionately affect national performance
- Health & Beauty leads by category (R$ 1.41M). Small-sized products drive the most revenue overall

**Logistics**
- 93.23% on-time delivery rate means ~6.7K late deliveries at this order volume — enough to generate significant customer dissatisfaction
- Avg. full cycle time is 12.5 days with 3.2 days in processing. **Recommendation:** reducing warehouse-to-carrier handoff time in SP and MG would lower average cycle time at the national level, given seller concentration in these states

**Payments**
- 51.5% of orders use installments — reflecting Brazilian parcelamento culture. Installment usage may indicate stronger purchase intent, though retention data is not available in this dataset to confirm
- Check payments represent a relatively large share (19.8K orders, R$ 2.87M) and may require additional reconciliation overhead compared to digital methods

**Feedback & Cross-Domain Insight**
- Avg. review score 4.09/5 with 14.69% bad reviews (score 1–2)
- Clear negative correlation between delivery time and review score — a direct link between the Logistics and Feedback domains
- Review scores drop in Brazilian summer months (Dec–Feb), coinciding with peak delivery delays. **Recommendation:** proactive SLA reinforcement with carriers during Nov–Jan would protect both delivery and satisfaction metrics

---

## 🗂️ Data Model

Star schema with a snowflake extension for category translation logic:

```
customers_dataset ─────────┐
products_dataset ──────────┤
product_category_name ─────┤──► orders_dataset (fact)
sellers_dataset ───────────┤        │
Calendar (custom) ─────────┘        ├──► order_items_dataset (fact)
                                    ├──► order_payments_dataset
                                    └──► order_reviews_dataset
```

`product_category_name` is kept as a separate dimension to normalise category translation strings and avoid repeated storage across product rows.

A dedicated `_DAX_Measures` table centralises all measures. A `For Switcher` helper table powers the dynamic metric switcher via field parameters.

> Full schema diagram: [`docs/data-model.png`](docs/data-model.png)

---

## ⚙️ Technical Highlights

### Power Query
- Merged 7 CSV files; translated product categories from Portuguese via merge query with translation table
- Created delivery status column; product size category via computed column in `products_dataset`
- Built custom `Calendar` table; geographic hierarchies on customer and seller location fields

### DAX Measures (selection)
| Measure | Description |
|---|---|
| `Revenue` | Price + freight for delivered orders only |
| `Pipeline Value` | Order value in progress (excl. cancelled / unavailable) |
| `Order Completion Rate` | Delivered orders / all created orders |
| `On Time Delivery %` | Deliveries on or before estimated date / total deliveries |
| `AVG Full Cycle Time` | Avg. days from order creation to customer delivery |
| `Bad Reviews %` | Share of reviews scored 1 or 2 |
| YoY comparison | Current vs. previous period via time intelligence |
| Metric Switcher | Field parameter switching between CNT Sales and Revenue |

### DAX Example — Funnel Measure with USERELATIONSHIP

The orders table has multiple business dates (purchase, approval, carrier handoff, estimated and actual delivery). Since only one relationship to the Calendar table can be active at a time, each funnel stage activates its own date field using `USERELATIONSHIP` inside `CALCULATE`:

```dax
Approved Orders =
CALCULATE(
    COUNTROWS(orders_dataset),
    USERELATIONSHIP(orders_dataset[order_approved_date], Calendar[Date])
)
```

This pattern repeats for each funnel stage, allowing the Calendar slicer to filter each metric by its own relevant date rather than a single shared date.

### UX & Interactivity
- Icon-based navigation panel; slicers for date, state, category, payment type, delivery status, review score
- Drill-through to Product Details page; custom Order Funnel tooltip
- YoY comparison on Sales trend chart

---

## ⚠️ Technical Challenges & Decisions

**Multiple date relationships**
The orders table contains multiple business dates. Funnel measures use `USERELATIONSHIP` inside `CALCULATE` to activate the relevant date context for each stage, since Power BI allows only one active relationship per table pair.

**Payment and review granularity**
Payments and reviews link to orders, not individual items. A multi-item order shares one review score and one payment record — relevant when interpreting per-item averages.

**Delivery status logic**
On-time delivery required comparing two date columns with potential nulls. The measure filters to delivered orders only to avoid blank-value distortion.

---

## 🔧 Performance Considerations

- Removed unused columns and tables after project completion
- Used measures over calculated columns where possible
- Disabled auto date/time to avoid hidden table generation per date field
- Normalised category translation strings into a separate dimension table

---

## ⚠️ Known Limitations

- No profit or margin data available — analysis is revenue-based only
- No customer lifetime or repeat purchase tracking beyond the available order history
- No carrier-level data to identify which logistics providers are responsible for late deliveries
- Dataset covers 2016–2018 and may not reflect current Brazilian e-commerce behaviour

---

## 🗃️ Dataset

**Source:** [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — Kaggle

~100K real orders, 2016–2018, across 7 relational tables covering orders, payments, products, sellers, customers, and reviews.

---

## 🛠️ Tools

Power BI Desktop · Power Query (M) · DAX · Bing Maps

---

## 🚀 How to Use

1. Download `olist-ecommerce-analysis.pbix` and open in **Power BI Desktop**
2. Data is embedded — no additional setup needed
3. Right-click any row in a table visual → **Drill through → Product Details**

---

## 👩‍💻 About

Built during a Data Analyst Weiterbildung as a course capstone project.

**Skills:** Power BI · Power Query · DAX · Data Modeling · ETL · Dashboard Design
