# E-commerce Growth & Customer Behaviour: an Analytical Dashboard in Power BI

A three-page Power BI report built on Google's public `thelook_ecommerce` dataset in BigQuery. It asks what is driving the growth of an online clothing store, which product categories actually earn the money, and how customers behave after their first purchase.

**Tools:** BigQuery SQL · Power BI Desktop (Power Query, data modelling, DAX)
**Dashboard type:** Analytical (one in a planned series covering strategic, operational, analytical and tactical dashboards)
**Data period:** January 2019 – 31 October 2026
**Last updated:** November 2026

> The data is synthetic. Everything below describes patterns in that dataset, and I say where that limits what can be concluded.

---

## Contents

1. [Headline findings](#1-headline-findings)
2. [Dashboard preview](#2-dashboard-preview)
3. [The business question, and how it changed](#3-the-business-question-and-how-it-changed)
4. [Data](#4-data)
5. [Definitions](#5-definitions)
6. [Design decisions](#6-design-decisions)
7. [Data model](#7-data-model)
8. [Dashboard pages and findings](#8-dashboard-pages-and-findings)
9. [Validation](#9-validation)
10. [Limitations](#10-limitations)
11. [What I would do with this in a real business](#11-what-i-would-do-with-this-in-a-real-business)
12. [Repository structure and how to reproduce](#12-repository-structure-and-how-to-reproduce)
13. [Appendix A: snapshot SQL](#appendix-a-snapshot-sql)
14. [Appendix B: key DAX](#appendix-b-key-dax)

---

## 1. Headline findings

- **Revenue has grown every year, and growth is accelerating.** Net revenue went from $69K in 2019 to $2.48M in 2026 (January to October only). That 2026 figure is 77.5% above the same ten months of 2025. Within 2026 the gap widens from about +52% in January to +119% in October, compared with the same months of 2025.
- **Growth comes mostly from new customers.** In 2026, 17,817 of 24,146 buying customers (74%) were buying for the first time.
- **Revenue and profit do not line up across categories.** Men's Jeans is the second-largest category ($562K) but earns only a 46.8% margin. Men's Suits & Sport Coats sells less ($459K) at a 59.8% margin and ends up with more gross profit ($275K against $263K).
- **Most customers buy once.** 29.8% of customers have ordered more than once. 43,592 have ordered exactly once.
- **Repeat customers matter a lot.** That 30% of customers accounts for 50.2% of net revenue.
- **When customers do return, they take a long time.** Among customers who placed a later order, 64% waited more than six months, and 22% waited more than two years.
- **Acquisition channel and country made no measurable difference.** This was a null result, and it is the reason the original question had to change (see section 3).

---

## 2. Dashboard preview

| Page | Question it answers |
|---|---|
| **1. Overview** | What is driving growth? |
| **2. Categories** | Which categories earn the money? |
| **3. Customers** | How do customers behave after the first order? |

![Page 1: Overview] (images/Analytical Dashboard Business Overview.png)
![Page 2: Categories](images/Analytical Dashboard Categories.png)
![Page 3: Customers](images/Analytical Dashboard Customers.png)

A PDF export of all three pages is in `dashboard/Analytical_Dashboard.pdf`.

---

## 3. The business question, and how it changed

I did not start with the question the dashboard now answers. The first version was dropped after the SQL exploration and data checks, before any dashboard work began.

### Initial question

> *Which acquisition channels bring the most valuable, most loyal customers, and does that differ by product category?*

I chose it because `traffic_source` (Search, Organic, Facebook, Email, Display) is a natural way to segment customers, and channel analysis is a classic analytical use case.

### What the data showed

I ran the checks below on the full public dataset before freezing a snapshot.

| Check | Search | Organic | Facebook | Email | Display |
|---|---|---|---|---|---|
| Users | 69,947 | 14,793 | 6,153 | 5,046 | 4,061 |
| Share of users who placed an order | 79.9% | 80.1% | 80.1% | 80.4% | 80.1% |
| Net buyers* | 46,182 | 9,757 | 4,038 | 3,302 | 2,686 |
| Repeat rate among net buyers | 30.6% | 30.3% | 30.5% | 30.3% | 30.2% |
| Average spend per net buyer | $122.93 | $121.50 | $124.08 | $121.82 | $128.41 |

\*Customers with at least one order that was not cancelled or returned.

Every channel converts at about 80%, repeats at about 30%, and spends $121–$128 per buyer. Display's slightly higher spend comes from the smallest group (2,686 buyers), and I treated it as noise. A dashboard built around channel comparison would have had nothing to say.

I also looked at geography, and it showed the same thing: revenue per buyer is $99–$104 in every country with a meaningful number of buyers. Return rates are about 10% in every category.

### Revised question

> *What is driving the business's growth, and which categories and customer behaviours actually earn the money?*

Three things in the data did have structure:

1. **Growth.** Volume more than doubled between October 2025 and October 2026, so there is a real story about where it comes from (new versus returning customers).
2. **Categories.** Margins range from about 38% to 62% and do not follow revenue.
3. **Repeat purchasing.** About 37% of customers who bought at all bought more than once, so a retention analysis was viable.

Channel and country remain in the project as documented null results, not as dashboard pages. Reporting that something does not matter is a legitimate finding, and it kept me from inventing a story.

---

## 4. Data

### Source

`bigquery-public-data.thelook_ecommerce`, a synthetic e-commerce dataset (a fictitious online clothing store) published by Google as a BigQuery public dataset. Tables used: `orders`, `order_items`, `products`, `users`.

### Snapshot

The public dataset keeps receiving new rows, so I froze a copy. All orders created before **2026-11-01** are included, which means the data ends on **31 October 2026**. The copy lives in three tables in my own BigQuery dataset, built by the SQL in [Appendix A](#appendix-a-snapshot-sql) and imported into Power BI in Import mode, so the `.pbix` file opens without any Google login.

| Table | Rows in snapshot |
|---|---|
| Order items (fact) | 169,354 |
| Users | 96,765 |
| Products | 29,120 |

Orders run from January 2019 to 31 October 2026.

### Data quality checks

| Check | Result | What I did |
|---|---|---|
| Table sizes (full dataset) | 125,042 orders · 181,719 order items · 29,120 products · 100,000 users | Small enough to import comfortably |
| Missing prices | None. Prices range from $0.02 to $999 | Nothing needed |
| Orphan records | No order item points to a missing product or user | Safe to relate the tables |
| Orders placed before the customer signed up | 0 of 125,042 | Signup date is safe for cohort logic |
| **Item status mix** | The mix is nearly identical in every month: about 25% Complete, 30% Shipped, 20% Processing, 15% Cancelled, 10% Returned. Orders from 2019 are still "Processing" | Statuses are not tied to order age, so I could not treat "Complete" as the only settled revenue (see section 5) |
| Orders per customer | 79,958 customers ordered at least once: 50,017 once, 19,866 twice, 5,007 three times, 5,068 four times. Nobody has more than 4 | Cohort curves are short, and this is noted in the limitations |
| **Latest month** | Daily volume sits at 260–300 items until mid-November 2026, then climbs to 1,924 on 22 November. The table also contains orders dated after the day I explored it | I cut the snapshot at 31 October and excluded November entirely |
| Early volumes | January 2019 has 3 items, and all of 2019 has 737 new customers | Kept all history. The cohort matrix starts in 2022 because early cohorts are too small to read |
| Country labels | Inconsistent in places ("Brasil", "España"), and a few countries have fewer than 10 buyers | Country is not used in the dashboard, so this has no effect on any figure |

---

## 5. Definitions

These are the rules every number on the dashboard follows.

| Term | Definition |
|---|---|
| **Net revenue** | Sale price of order items with status Complete, Shipped or Processing (everything except Cancelled and Returned) |
| **Completed revenue** | Sale price of items with status Complete only. About 34% of net revenue ($2.55M of $7.56M). Because statuses in this dataset do not age realistically, it is a conservative lower bound, not "settled" revenue |
| **Gross margin** | Net revenue minus the product cost of the same items. **Gross margin %** = gross margin / net revenue |
| **Net orders** | Distinct orders containing at least one net item |
| **Buying customers** | Distinct customers with at least one net item in the selected period |
| **Average order value** | Net revenue / net orders |
| **New customer** (in a period) | A customer whose first net order falls inside the period |
| **Returning customer** (in a period) | A customer who bought in the period and whose first net order was before it. New plus returning always equals buying customers |
| **Repeat customer** | A customer with two or more net orders, counted over their whole history. **Repeat purchase rate** = repeat customers / buying customers |
| **One-time customer** | A customer with exactly one net order |
| **Cohort month** | The calendar month of a customer's first net order |
| **Reorder** | A net order placed in a later calendar month than the first. A customer who ordered twice within their first month and never again is therefore not counted as a reorder (about 650 customers) |
| **Retention % (matrix)** | Customers in a cohort who bought N months after their first order, divided by the cohort size |
| **12-month reorder rate** | Of customers whose first order was at least 12 months before the end of the data (cohorts up to October 2025, 41,593 customers), the share who reordered within 12 months |
| **Revenue growth vs. same period last year** | Year-to-date revenue for the latest year in view, compared with the same days of the previous year. For 2026 that means January–October against January–October 2025 |

---

## 6. Design decisions

| Decision | Reasoning |
|---|---|
| Net revenue excludes only Cancelled and Returned | Excluding Shipped and Processing too would have removed about half of all business and left a random 25% slice, because statuses do not follow order age |
| Show Completed revenue next to Net revenue | A viewer can see how much the choice matters, and the definition is visible instead of hidden |
| Snapshot cutoff at 31 October 2026 | November was incomplete and contained anomalous and future-dated rows |
| Keep 2019–2020 data despite tiny volumes | Trimming it would make early customers look new in later years and distort the new-versus-returning split |
| Product cost sits in the fact table, not the products table | Margin becomes a simple sum, and cost lives in exactly one place |
| Order date comes from the `orders` table | One date definition for the whole project, including first-order logic |
| Revenue statuses are applied in DAX, not in SQL | The definition stays visible in the measures, and both net and completed views are possible |
| Repeat customers measured over lifetime, ignoring the date filter | "Bought at least twice" is a property of the customer, not of a month |
| Cohort matrix starts in 2022 | Cohorts before 2020 have between 13 and a few hundred customers, so a single repeat buyer shows as an 8–15% spike that means nothing |
| A time-to-reorder chart alongside the matrix | Retention is only 1–2% in any single month, so the matrix is pale. The chart shows the real pattern, which is slow and spread out |

---

## 7. Data model

A star schema, with all relationships single-direction and many-to-one from the fact table.

```
Dim_Date ───< Order items Fact >─── Users Dim
                    │
                    v
              Products Dim
```

| Table | Role | Key columns |
|---|---|---|
| `Order items Fact` | One row per order item | order_item_id, order_id, user_id, product_id, order_date, item_status, sale_price, product_cost |
| `Users Dim` | One row per customer | user_id, country, traffic_source, age, signup_date |
| `Products Dim` | One row per product | product_id, product_name, brand, category, department |
| `Dim_Date` | Calendar, 2019-01-01 to 2026-10-31, marked as the date table | Date, Year, Quarter, Month (sorted by Month Number), Year-Month |
| `Measure Table` | Holds all measures | |

**Calculated columns**

- On `Users Dim`: `First Order Date`, `Cohort Month`, `Months to First Reorder`, `Reorder Window` (with `Reorder Window Sort`)
- On `Order items Fact`: `Months Since First Order` (blank for cancelled and returned items)

**Measures**

- *Revenue and profit:* Net Revenue, Completed Revenue, Net Cost, Gross Margin, Gross Margin %, Net Orders, Average Order Value
- *Growth:* Net Revenue PY, Revenue YoY %, Revenue Growth vs Same Period LY
- *Customers:* Buying Customers, New Customers, Returning Customers
- *Repeat behaviour:* Repeat Customers, Repeat Purchase Rate, Repeat Customers Revenue Share, 12-Month Reorder Rate, Median Months to Reorder
- *Cohorts:* Cohort Size, Retention %
- *Categories:* Top Category, Top Category Revenue

The most important formulas are in [Appendix B](#appendix-b-key-dax).

---

## 8. Dashboard pages and findings

### Page 1: Overview

KPI cards for net revenue, gross margin %, buying customers and repeat purchase rate (the first and third show the latest year, 2026), a like-for-like growth card, a yearly revenue line, and a new-versus-returning customer chart. The date hierarchy on the charts supports drill-down from year to month.

| Year | Net revenue | Growth vs previous year |
|---|---|---|
| 2019 | $69,266 | n/a |
| 2020 | $209,913 | +203% |
| 2021 | $389,775 | +86% |
| 2022 | $600,857 | +54% |
| 2023 | $832,038 | +38% |
| 2024 | $1,201,940 | +44% |
| 2025 | $1,769,290 | +47% |
| 2026 (Jan–Oct) | $2,484,876 | +77% (same ten months of 2025) |

The yearly figures add up to the total net revenue of $7,557,955. In 2026 the share of buying customers who were new was 74% (17,817 new, 6,329 returning), so growth is mostly acquisition.

### Page 2: Categories

A scatter chart of gross margin % against revenue (each dot is a department and category), gross margin in dollars by category, a top-category card that reacts to the department and year slicers, and written takeaways.

- There are 36 department–category combinations. The top five account for only 31% of revenue, so no single category carries the business.
- Men account for 53% of net revenue ($4.03M against $3.52M). Both departments earn about 52% gross margin.
- Overall, the top category is **Outerwear & Coats** ($923K with both departments combined). For men it leads at $600K and a 55.8% margin. For women the top category is **Dresses** at $326K.

| Department and category | Net revenue | Gross margin | Margin % |
|---|---|---|---|
| Men, Outerwear & Coats | $600K | $335K | 55.8% |
| Men, Jeans | $562K | $263K | 46.8% |
| Men, Suits & Sport Coats | $459K | $275K | 59.8% |
| Women, Dresses | $326K | $179K | 54.9% |
| Women, Intimates | $321K | $151K | 46.9% |

- Margins range from 37.8% (Women's Clothing Sets) to 62.1% (Women's Blazers & Jackets). Returns are about 10% everywhere.
- **What the data cannot tell me:** why margins differ. Given that the dataset is synthetic, I did not try to explain it with material costs or pricing strategy.

### Page 3: Customers

Repeat purchase rate and one-time customer cards, a card with three headline metrics, a cohort retention matrix, and a chart of how long returning customers take to place their next order.

| Metric | Value |
|---|---|
| Customers with two or more net orders | 18,479 of 62,071 (29.8%) |
| Customers with exactly one net order | 43,592 |
| Repeat customers' share of net revenue | 50.2% |
| Reordered within 6 months (customers 12+ months old) | 8.7% |
| Reordered within 12 months (customers 12+ months old) | 16.2% |
| Median wait to a later-month order (among those who reordered) | 11 months |

Time from first order to the next one, for the 17,833 customers who placed a later-month order:

| Window | Customers | Share |
|---|---|---|
| 1 month | 1,460 | 8% |
| 2–3 months | 2,380 | 13% |
| 4–6 months | 2,561 | 14% |
| 7–12 months | 3,535 | 20% |
| 13–24 months | 3,970 | 22% |
| 25+ months | 3,927 | 22% |

In the cohort matrix, only about 1–2% of a cohort is active in any month after the first. As an example, the January 2024 cohort has 797 customers, and 9 of them (1.1%) bought in month 1. The low monthly rate is a result of timing: reorders are spread over years instead of clustering shortly after the first purchase.

---

## 9. Validation

- **Reconciliation with the source.** Net revenue computed in Power BI ($7,557,954.54) matches the SQL calculation on the snapshot, and the row count (169,354) matches too.
- **Cross-checks inside the model.** Buying customers (62,071) equals the number of users with a first order date. New plus returning customers equals buying customers in every year. Yearly net revenue adds up to the grand total.
- **Mistakes caught by testing.** Several measures went through a test-and-fix loop: a typo in a status name that silently dropped Processing orders (it showed 63,865 orders instead of 87,091), a misplaced parenthesis that broke the margin measures, and a date filter that was not fully cleared for repeat-rate measures. All are fixed, and the figures in this README come from the corrected model.
- **A first version of the channel check was wrong.** It averaged over all users instead of buyers only, which gave a repeat rate of about 20% instead of about 30%. The conclusion (no difference between channels) was the same either way.
- **Test cohort.** The January 2024 cohort was recalculated independently of the matrix: 797 customers, all active in month 0, and 9, 11 and 11 in months 1, 2 and 3.

---

## 10. Limitations

- **The data is synthetic.** Flat channel results, a maximum of four orders per customer, and randomly assigned statuses all point to generated data. Patterns here illustrate method, and they do not describe a real business.
- **Statuses do not age.** Net revenue is a defined metric here, not an accounting truth. Completed revenue is shown as a conservative comparison.
- **2026 is a partial year (January–October).** The growth comparison is like-for-like, but the 2026 total is not a full-year figure.
- **Lifetime repeat rate understates recent customers.** Someone who first bought last month has had little time to return. The 12-month reorder rate corrects for this by looking only at customers with at least a year of history.
- **The 11-month median is a floor.** Recent customers cannot yet show long waits, so the real typical wait is probably longer.
- **Revenue is sale price only.** There are no discounts, shipping costs or overheads in the data, so margin means sale price minus product cost.
- **Short cohort curves.** With at most four orders per customer, long-run loyalty cannot be studied.
- **No forecasting and no causal claims.** The dashboard describes what happened and where, and it does not say why.

---

## 11. What I would do with this in a real business

These are illustrative, since the data is synthetic, and each is a question to test, not a conclusion.

1. **Judge retention over at least a year.** If real customers also take 7 to 24 months to reorder, a win-back campaign evaluated after six weeks would look like a failure when it is not.
2. **Treat repeat purchasing as a lever.** Half of revenue comes from 30% of customers, so a few points of improvement in repeat rate would have a large effect.
3. **Look at the mix of high-revenue, low-margin categories.** Jeans and Intimates bring volume at 46–47% margin, while categories such as Suits & Sport Coats earn nearly 60%. I would check whether pricing or promotion explains the gap.
4. **Do not use this data to allocate marketing spend by channel.** It shows no difference, and a real dataset should be checked before drawing the same conclusion.

---

## 12. Repository structure and how to reproduce

```
.
├── README.md
├── dashboard/
│   ├── Analytical_Dashboard.pbix
│   └── Analytical_Dashboard.pdf
├── images/                     # page screenshots
└── sql/
    ├── 01_exploration.sql      # data checks from section 4
    └── 02_snapshot.sql         # Appendix A
```

**To open the report:** install Power BI Desktop and open `Analytical_Dashboard.pbix`. The data is stored inside the file, so no login or connection is needed.

**To rebuild from source:**

1. In BigQuery, create a dataset called `portfolio` in the **US** location (the public data is in the US, and BigQuery will not write across regions).
2. Run the three statements in `sql/02_snapshot.sql`.
3. In Power BI Desktop, connect with the Google BigQuery connector in Import mode and load the three tables.
4. Rebuild the model, columns and measures from sections 7 and 5.

---

## Appendix A: snapshot SQL

<details>
<summary>Three tables, all cut at 31 October 2026</summary>

```sql
CREATE OR REPLACE TABLE portfolio.fact_order_items AS
SELECT
  oi.id          AS order_item_id,
  oi.order_id,
  oi.user_id,
  oi.product_id,
  DATE(o.created_at) AS order_date,
  oi.status      AS item_status,
  oi.sale_price,
  p.cost         AS product_cost
FROM `bigquery-public-data.thelook_ecommerce.order_items` oi
JOIN `bigquery-public-data.thelook_ecommerce.orders` o
  ON o.order_id = oi.order_id
JOIN `bigquery-public-data.thelook_ecommerce.products` p
  ON p.id = oi.product_id
WHERE o.created_at < TIMESTAMP('2026-11-01');

CREATE OR REPLACE TABLE portfolio.dim_users AS
SELECT
  id AS user_id,
  country,
  traffic_source,
  age,
  DATE(created_at) AS signup_date
FROM `bigquery-public-data.thelook_ecommerce.users`
WHERE created_at < TIMESTAMP('2026-11-01');

CREATE OR REPLACE TABLE portfolio.dim_products AS
SELECT
  id AS product_id,
  name AS product_name,
  brand,
  category,
  department
FROM `bigquery-public-data.thelook_ecommerce.products`;
```

Check query (expected: last date 2026-10-31, net revenue about $7,557,955, 169,354 rows):

```sql
SELECT
  COUNT(*) AS rows_in_fact,
  MIN(order_date) AS first_date,
  MAX(order_date) AS last_date,
  ROUND(SUM(IF(item_status NOT IN ('Cancelled', 'Returned'), sale_price, 0)), 0) AS net_revenue
FROM portfolio.fact_order_items;
```

</details>

---

## Appendix B: key DAX

<details>
<summary>Core measures and customer logic</summary>

```dax
Net Revenue =
CALCULATE(
    SUM('Order items Fact'[sale_price]),
    'Order items Fact'[item_status] IN {"Complete", "Shipped", "Processing"}
)

Completed Revenue =
CALCULATE(
    SUM('Order items Fact'[sale_price]),
    'Order items Fact'[item_status] = "Complete"
)

Gross Margin = [Net Revenue] - [Net Cost]

Gross Margin % = DIVIDE([Gross Margin], [Net Revenue])

Buying Customers =
CALCULATE(
    DISTINCTCOUNT('Order items Fact'[user_id]),
    'Order items Fact'[item_status] IN {"Complete", "Shipped", "Processing"}
)
```

Like-for-like growth, comparing the latest year in view year-to-date with the same days of the year before:

```dax
Revenue Growth vs Same Period LY =
VAR EndDay = MAX(Dim_Date[Date])
VAR YearStart = DATE(YEAR(EndDay), 1, 1)
VAR CY =
    CALCULATE(
        [Net Revenue],
        FILTER(ALL(Dim_Date), Dim_Date[Date] >= YearStart && Dim_Date[Date] <= EndDay)
    )
VAR PY =
    CALCULATE(
        [Net Revenue],
        FILTER(
            ALL(Dim_Date),
            Dim_Date[Date] >= EDATE(YearStart, -12) && Dim_Date[Date] <= EDATE(EndDay, -12)
        )
    )
RETURN
DIVIDE(CY - PY, PY)
```

New and returning customers:

```dax
New Customers =
VAR MinD = MIN(Dim_Date[Date])
VAR MaxD = MAX(Dim_Date[Date])
RETURN
CALCULATE(
    [Buying Customers],
    'Users Dim'[First Order Date] >= MinD,
    'Users Dim'[First Order Date] <= MaxD
)

Returning Customers =
VAR MinD = MIN(Dim_Date[Date])
RETURN
CALCULATE([Buying Customers], 'Users Dim'[First Order Date] < MinD)
```

Customer columns on `Users Dim`:

```dax
First Order Date =
CALCULATE(
    MIN('Order items Fact'[order_date]),
    'Order items Fact'[item_status] <> "Cancelled",
    'Order items Fact'[item_status] <> "Returned"
)

Cohort Month =
IF(
    ISBLANK('Users Dim'[First Order Date]),
    BLANK(),
    DATE(YEAR('Users Dim'[First Order Date]), MONTH('Users Dim'[First Order Date]), 1)
)

Months to First Reorder =
IF(
    NOT ISBLANK('Users Dim'[Cohort Month]),
    CALCULATE(
        MIN('Order items Fact'[Months Since First Order]),
        'Order items Fact'[Months Since First Order] > 0
    )
)
```

Months since first order, on `Order items Fact` (net items only):

```dax
Months Since First Order =
IF(
    'Order items Fact'[item_status] IN {"Complete", "Shipped", "Processing"},
    DATEDIFF(
        RELATED('Users Dim'[Cohort Month]),
        DATE(YEAR('Order items Fact'[order_date]), MONTH('Order items Fact'[order_date]), 1),
        MONTH
    )
)
```

Repeat behaviour:

```dax
Repeat Customers =
CALCULATE(
    COUNTROWS(FILTER('Users Dim', [Net Orders] >= 2)),
    REMOVEFILTERS(Dim_Date)
)

Repeat Purchase Rate =
DIVIDE(
    [Repeat Customers],
    CALCULATE([Buying Customers], REMOVEFILTERS(Dim_Date))
)

Repeat Customers Revenue Share =
DIVIDE(
    SUMX(FILTER('Users Dim', [Net Orders] >= 2), [Net Revenue]),
    [Net Revenue]
)

Cohort Size =
COUNTROWS(FILTER('Users Dim', NOT ISBLANK('Users Dim'[Cohort Month])))

Retention % = DIVIDE([Buying Customers], [Cohort Size])
```

</details>

---

## Author

**Jan Pruszczyński**, Warsaw · [LinkedIn](www.linkedin.com/in/jan-pruszczynski) · [GitHub](https://github.com/JanPruszczynski)

