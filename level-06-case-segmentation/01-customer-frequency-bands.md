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
			WHEN "Ilosc_platnosci" BETWEEN 20 AND 24 THEN '20 - 24'
			WHEN "Ilosc_platnosci" BETWEEN 15 AND 19 THEN '15 - 19'
			ELSE '< 15'
		END AS "Przedzial",
		CASE 
			WHEN "Ilosc_platnosci" > 40 THEN 1
			WHEN "Ilosc_platnosci" BETWEEN 35 AND 40 THEN 2
			WHEN "Ilosc_platnosci" BETWEEN 30 AND 34 THEN 3
			WHEN "Ilosc_platnosci" BETWEEN 25 AND 29 THEN 4
			WHEN "Ilosc_platnosci" BETWEEN 20 AND 24 THEN 5
			WHEN "Ilosc_platnosci" BETWEEN 15 AND 19 THEN 6
			ELSE 7
		END AS sort_order
	FROM ilosci
)

SELECT 
	"Przedzial",
	COUNT(*) AS "Liczba_klientow",
	ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS "Procent_calosci"
FROM bucketed
GROUP BY "Przedzial", sort_order
ORDER BY sort_order;
```

### Query Results

| Przedzial | Liczba_klientow | Procent_calosci |
|:---:|:---:|:---:|
| > 40 | 2 | 0.33 |
| 35 – 40 | 12 | 2.00 |
| 30 – 34 | 84 | 14.02 |
| 25 – 29 | 181 | 30.22 |
| 20 – 24 | 219 | 36.56 |
| 15 – 19 | 89 | 14.86 |
| < 15 | 12 | 2.00 |

> Sum of customers = 599 (whole base); the band "20–29" (combining 25–29 and 20–24) holds 400 customers = **66.78% of the base**.

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
- **`SUM(COUNT(*)) OVER ()`**: computes the total count across all grouped rows using a window function over the *aggregated* result set — the whole base (599) in a single frame. Dividing a group's `COUNT(*)` by it yields the share of customers per band **entirely in SQL** (no separate total needed in the client).
- **`* 100.0` (not `* 100`)**: forces floating-point division — `ROUND(COUNT(*) * 100.0 / ..., 2)` keeps two decimals (e.g., 14.02, 36.56).
- **Two-stage CTE flow**: `ilosci` (count payments per customer) → `bucketed` (assign bands) → final aggregation (`COUNT(*)` per band).
- **`GROUP BY` on both the band and the helper**: guarantees one row per band in the right order.