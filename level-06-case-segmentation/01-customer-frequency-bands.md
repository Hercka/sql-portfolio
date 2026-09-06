# Level 6: Segmentation with CASE WHEN — Customer Frequency Bands

> **Database**: dvdrental (Sakila)
> **Date**: 2026-09-06
> **Status**: ✅ Solved

---

## Challenge: Customer Segmentation by Order Frequency

**Business Context**: The loyalty program needs to know how the customer base is distributed by rental frequency — how many customers sit in each activity band. This distribution decides where the "every Nth rental free" mechanic creates the most lift.

### SQL Solution

```sql
WITH ilosci AS (
	SELECT 
		c.customer_id,
		CONCAT(c.first_name, ' ', c.last_name) AS "Klient",
		COUNT(p.payment_id) AS "Ilosc_platnosci"
	FROM customer AS c
	JOIN payment AS p ON p.customer_id = c.customer_id
	GROUP BY c.customer_id
),
bucketed AS (
	SELECT 
		customer_id,
		CASE 
			WHEN "Ilosc_platnosci" > 40 THEN '> 40'
			WHEN "Ilosc_platnosci" BETWEEN 35 AND 40 THEN '35 - 40'
			WHEN "Ilosc_platnosci" BETWEEN 30 AND 34 THEN '30 - 34'
			WHEN "Ilosc_platnosci" BETWEEN 25 AND 29 THEN '25 - 29'
			ELSE '< 25'
		END AS "Przedzial",
		-- Helper column preserving correct textual sort order
		CASE 
			WHEN "Ilosc_platnosci" > 40 THEN 1
			WHEN "Ilosc_platnosci" BETWEEN 35 AND 40 THEN 2
			WHEN "Ilosc_platnosci" BETWEEN 30 AND 34 THEN 3
			WHEN "Ilosc_platnosci" BETWEEN 25 AND 29 THEN 4
			ELSE 5
		END AS sort_order
	FROM ilosci
)

SELECT 
	"Przedzial",
	COUNT(*) AS "Liczba_klientow"
FROM bucketed
GROUP BY "Przedzial", sort_order
ORDER BY sort_order;
```

### Query Results

| Przedzial | Liczba_klientow |
|:---:|:---:|
| > 40 | 2 |
| 35 – 40 | 12 |
| 30 – 34 | 84 |
| 25 – 29 | 181 |
| < 25 | 320 |

### Refined Banding (finer granularity, used for the chart)

| Band | Customers | % of base |
|:---:|:---:|:---:|
| 40+ | 3 | 0.5% |
| 35–39 | 11 | 1.8% |
| 30–34 | 84 | 14.0% |
| 25–29 | 181 | 30.2% |
| 20–24 | 219 | 36.6% |
| 15–19 | 89 | 14.9% |
| <15 | 12 | 2.0% |

---

## Business Insights & Actionable Recommendations

**Insight 1 — The critical mass is the 20–29 order band.** Combining 20–24 and 25–29 gives **400 customers (66.8% of the base)** — by far the largest concentration of customers with meaningful activity but still room to grow. This is where a small frequency lift produces the biggest aggregate revenue effect.

**Insight 2 — The extreme top band is negligible for growth.** Only **3 customers (0.5%)** exceed 40 orders. They are already at the practical activity ceiling (~1.4 rentals per week over the observed window); rewarding them further would subsidize behavior they already perform.

**Insight 3 — Tenure must be verified before targeting low bands.** The <15 group (2.0%) may consist mostly of **new customers** rather than disengaged ones. Before any win-back campaign, measure how long customers took to reach their current order count (cohort / time-to-N-orders analysis). Segmentation must separate "new" from "dormant" — mirroring quality-control logic: a process that just started is not the same as a process that drifted out of spec.

**Recommended action**: launch the "every Nth rental free" mechanic on the 20–29 band (e.g., every 5th rental free within a rolling period), monitor frequency lift per cohort, and treat <15-order customers as a separate follow-up campaign after tenure analysis.

---

## Key Concepts

- **`CASE WHEN ... THEN ... END`**: maps each customer into a predefined band — the backbone of any segmentation logic.
- **Numeric `sort_order` helper column**: lets you `GROUP BY` and `ORDER BY` text bands in the correct logical sequence ("> 40" would otherwise sort before "35 – 40" alphabetically). Clean, portable pattern.
- **Two-stage CTE flow**: `ilosci` (count payments per customer) → `bucketed` (assign bands) → final aggregation (`COUNT(*)` per band).
- **`GROUP BY` on both the band and the helper**: guarantees one row per band in the right order.