# Level 5: Window Functions — Moving Maximum & Moving Average (CTE)

> **Database**: dvdrental (Sakila)
> **Date**: 2026-09-01
> **Status**: ✅ Solved

---

## Challenge: Customer Spend Trend — Moving Maximum & Moving Average

**Business Context**: We want to understand how a customer's per-transaction spend evolves over time. By computing the moving maximum and moving average over the last 3 payments (a 3-row window), we can detect whether a customer's spending is stable, rising, or falling — feeding decisions on loyalty programs, upselling, and customer valuation.

**Customer**: `customer_id = 48` (14 payments during 2007, ordered chronologically).

### SQL Solution

```sql
WITH PAYMENT AS (
  SELECT
    ROW_NUMBER() OVER (ORDER BY p.payment_date ASC) AS "Nr. wplaty",
    p.amount AS "Kwota ($)"
  FROM customer c
  JOIN payment p ON p.customer_id = c.customer_id
  WHERE c.customer_id = 48
)

SELECT
  "Nr. wplaty",
  "Kwota ($)",
  MAX("Kwota ($)") OVER (
    ORDER BY "Nr. wplaty" ASC
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
  ) AS "Max 3 ostatnich",
  ROUND(AVG("Kwota ($)") OVER (
    ORDER BY "Nr. wplaty" ASC
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
  ), 2) AS "Srednia z 3 ostatnich"
FROM PAYMENT;
```

### Query Results

| Nr | Kwota ($) | Max 3 ostatnich | Srednia z 3 ostatnich |
|:--:|:--------:|:--------------:|:---------------------:|
| 1  | 9.99 | 9.99 | 9.99 |
| 2  | 0.99 | 9.99 | 5.49 |
| 3  | 2.99 | 9.99 | 4.66 |
| 4  | 1.99 | 2.99 | 1.99 |
| 5  | 2.99 | 2.99 | 2.66 |
| 6  | 3.99 | 3.99 | 2.99 |
| 7  | 4.99 | 4.99 | 3.99 |
| 8  | 2.99 | 4.99 | 3.99 |
| 9  | 6.99 | 6.99 | 4.99 |
| 10 | 3.99 | 6.99 | 4.66 |
| 11 | 2.99 | 6.99 | 4.66 |
| 12 | 6.99 | 6.99 | 4.66 |
| 13 | 7.99 | 7.99 | 5.99 |
| 14 | 7.99 | 7.99 | 7.66 |

---

## Business Insights & Actionable Recommendations

**Insight (Customer 48 — rising per-payment value):** Excluding the first payment of $9.99 (a clear outlier that artificially inflated the moving windows for rows 1–3), the customer's per-payment amount shows a sustained upward trend from the 4th payment onward. The 3-payment moving average rises monotonically from roughly $1.99 (payments 4–6) to $7.66 (payments 12–14) — approximately a 3.7x increase in per-transaction value.

**Business interpretation:** In the context of a rental business, this is a **positive revenue signal**, not a risk. The customer is increasing the amount they spend per visit, which translates directly into higher revenue contribution. The rising curve therefore points toward customer growth rather than churn or profitability concerns.

**Actionable recommendations:**
1. **Treat customer 48 as a high-value, growing customer** — flag her account for the loyalty program and targeted upselling (e.g., premium rental tiers, bundle offers).
2. **Monitor the trend going forward** — the moving maximum/average is a stable early-warning metric: if the per-payment value later drops back toward the 1.x–2.x range, that would signal reduced engagement worth investigating (e.g., loss of interest, competing service).
3. **Ignore the first-payment outlier in any forecasting** — the $9.99 opening payment (note: possibly a setup/anomalous transaction) should be excluded or downweighted when modeling expected revenue from this customer, as it distorts the first three window averages.

---

## Key Concepts

- **Window functions**: perform calculations across a set of rows related to the current row, without collapsing them like `GROUP BY`.
- **`MAX(...) OVER (...)` / `AVG(...) OVER (...)`**: compute the running maximum / running average over a window frame.
- **`ROW_NUMBER() OVER (ORDER BY ...)`**: assigns a sequential number (1, 2, 3, ...) to each row in the specified order — used here to number payments chronologically.
- **Window frame `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`**: defines a *moving / sliding* window of exactly the current row plus the 2 preceding rows (i.e., the last 3 observations).
- **Partial frames**: at the start of the result set (rows 1–2) the frame contains fewer than 3 rows, so the moving max/avg are computed over only the available values — this is correct, expected behavior, not an error.
- **CTEs (`WITH ... AS`)**: make the query readable by first building a numbered, time-ordered list of payments, then applying the moving-window aggregations on top.