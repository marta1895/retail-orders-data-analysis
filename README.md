# Retail Orders — Sales Diagnostic (2022–2023)

End-to-end analysis of two years of global mart sales data across Furniture, Office Supplies, and Technology categories. The goal is to move past headline revenue and find where the business actually grew, where it stalled, and which sub-categories carry the franchise.

## Dashboard preview

<img width="1600" height="1073" alt="image" src="https://github.com/user-attachments/assets/5cf1a07e-cd74-4885-8388-be3bd68d8a81" />

Interactive version on Tableau Public: *[<add link here, or note that the packaged workbook `Sales_analysis.twbx` is included in the repo>](https://public.tableau.com/app/profile/marta.narozhnyak/viz/shared/ZGQTXF3CW)*

## Data

Source: [Retail Orders Dataset — Kaggle](https://www.kaggle.com/datasets/ankitbansal06/retail-orders/)

A single transactional table covering January 2022 – December 2023. Each row is one product line within an order, with customer region, product category and sub-category, list price, cost, discount percentage, ship mode, and order date. Three top-level categories: **Furniture**, **Office Supplies**, **Technology**.

## Tech stack

- **Python / Jupyter** — Kaggle API import via `opendatasets`, null handling, dtype fixes, and derivation of `discount`, `sale_price`, and `profit` columns from the raw pricing fields. Cleaned data loaded into PostgreSQL via SQLAlchemy.
- **PostgreSQL (pgAdmin)** — schema redefined with appropriate types after the initial load, then a five-section analytical SQL pass.
- **Tableau Desktop / Public** — summary dashboard of the findings.

## Repository structure

```
.
├── orders.csv                   # raw dataset from Kaggle
├── sales_analysis.ipynb         # Python: cleaning + load to PostgreSQL
├── sales_analysis.sql           # PostgreSQL: 5-question analytical EDA
├── Sales_analysis.twbx          # Tableau packaged workbook
├── dashboard.png                # Final dashboard
└── README.md
```

## Analysis structure (SQL)

The SQL file is organized around five diagnostic questions, each tied to a decision someone in the business would actually need to make.

| # | Question | Why it matters |
| - | -------- | -------------- |
| 1 | Which products drive the most revenue, and how concentrated is that revenue? | Identifies the SKUs to protect from stockouts and the long tail at risk of being culled. |
| 2 | Which sub-categories dominate volume in each region? | Surfaces regional mix differences for category buyers and merchandising. |
| 3 | Did 2023 outperform 2022, and where did the growth come from? | Distinguishes broad-based growth from a few category wins. |
| 4 | When does each category peak, and how aligned are the peaks? | Drives inventory and promo timing across the three categories. |
| 5 | Which sub-category grew fastest year-over-year? | Flags candidates for incremental investment. |

## Cleaning notes

Six rows had missing `Ship Mode` values; these were retained rather than dropped, since the missingness did not correlate with any other variable. `order_date` was stored as a string and converted to `datetime`. Three derived fields (`discount`, `sale_price`, `profit`) were calculated from `list_price`, `cost_price`, and `discount_percent`, after which the originals were dropped — all downstream analysis uses the derived columns. `order_id` was verified as unique and non-null and is treated as the primary key.

The cleaned table was first loaded into PostgreSQL via SQLAlchemy with `if_exists='replace'`, which forced pandas to default to the widest possible types (`bigint`, `text`). The table was then dropped, the schema was redefined with appropriate types (numeric, date, varchar) and an explicit primary key, and the data was reloaded with `if_exists='append'` to enforce the schema.

## Key findings

> Numbers below to be filled in from your own query outputs. Each finding is framed; you plug in the values.

**1. Revenue is concentrated in a small number of products.** The top 10 SKUs account for *[X%]* of total revenue across both years. *[Add: which categories the top products fall into — Technology-heavy? Furniture-heavy? — and what that implies for assortment risk.]*

**2. Regional sub-category mix diverges meaningfully.** The top three sub-categories by volume are *[similar / different]* across regions. *[Add: which region is the outlier, which sub-category over- or under-indexes there, and what that suggests about local demand or distribution coverage.]*

**3. 2023 vs 2022 — growth was *[broad / narrow]*.** Total revenue moved from *[2022 total]* to *[2023 total]*, a *[X%]* change. *[Add: was growth driven by one category, by specific months, was any category in decline?]*

**4. Category peaks are *[aligned / staggered]*.** Furniture peaked in *[month]*, Office Supplies in *[month]*, Technology in *[month]*. *[Add: one-line implication for staffing, promotions, or inventory timing.]*

**5. The fastest-growing sub-category was *[name]*, with a *[X%]* year-over-year increase.** *[Add: did it grow from a small base or a meaningful one? Is the growth coming from more orders, larger orders, or smaller discounts?]*

## Limitations

- **Two-year window.** Year-over-year comparisons are based on a single pair of years; the 2023 signal could be a one-off rather than a trend. A third year would meaningfully reduce uncertainty.
- **No customer dimension.** The dataset captures order lines but not customer identity, so retention, lifetime value, and repeat-purchase behavior cannot be analyzed.
- **No external context.** Price changes, marketing spend, and competitor actions are not in the dataset; growth or decline cannot be attributed to specific causes without that context.
- **Discount logic.** `discount_percent` is applied per line. If the source applied promotions at order level (e.g., a basket-level coupon), per-line attribution would understate true discount depth on bundled purchases.





