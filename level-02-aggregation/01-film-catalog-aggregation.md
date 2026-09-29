# Level 2: Aggregation — GROUP BY, COUNT, SUM, AVG

> **Database**: dvdrental (Sakila)
> **Date**: 2026-09-27
> **Status**: ✅ Solved

---

## Challenge: Film Catalog Aggregation

**Business Context**: The store manager is preparing a purchasing policy and needs to understand how the film catalog is distributed across MPAA ratings: how many films each category holds, the average rental price, average length, and the total replacement value of each group. A second cut checks how the catalog splits by the allowed rental period.

### SQL Solution — Part 1: Catalog by MPAA rating

```sql
SELECT
		rating
	,	COUNT(film_id) as "Film count"
	,	ROUND(AVG(rental_rate),2) as "Avg rental rate"
	,	ROUND(AVG(length),1) as "Avg length"
	,	SUM(replacement_cost) as "Total replacement cost"
FROM film
GROUP BY rating
ORDER BY "Film count" DESC;
```

### Query Results

| rating | Film count | Avg rental rate | Avg length | Total replacement cost |
|:---:|:---:|:---:|:---:|:---:|
| PG-13 | 223 | 3.03 | 120.4 | 4549.77 |
| NC-17 | 210 | 2.97 | 113.2 | 4228.90 |
| R | 195 | 2.94 | 118.7 | 3945.05 |
| PG | 194 | 3.05 | 112.0 | 3678.06 |
| G | 178 | 2.89 | 111.1 | 3582.22 |

### SQL Solution — Part 2: Catalog by allowed rental period

```sql
SELECT
	rental_duration "Rental duration"
	, count(film_id) "Numbers of films"
FROM film
GROUP BY rental_duration
ORDER by rental_duration DESC;
```

### Query Results

| Rental duration | Number of films |
|:---:|:---:|
| 7 | 191 |
| 6 | 212 |
| 5 | 191 |
| 4 | 203 |
| 3 | 203 |

---

## Business Insights & Actionable Recommendations

**Insight 1 — Pricing is not differentiated by MPAA rating.** Average rental rate is effectively flat across all categories (2.89–3.05). However, the average hides the real distribution: the catalog contains only three price points (0.99, 2.99, 4.99), and **every** rating category contains all three in nearly equal proportions (e.g., G: 64 / 59 / 55). A customer therefore faces a 5x price spread *within* any single category. The pricing logic is not driven by the age rating — it must be explained by another dimension of the film data, which warrants a follow-up analysis.

**Insight 2 — Replacement cost differs per unit, not only in total.** PG films have the lowest replacement cost **per film** (18.96 $), against roughly 20.1–20.4 $ for every other category. In **total** terms, however, G is the cheapest group (3582.22 $) simply because it holds the fewest titles (178). Both metrics are reported separately to avoid presenting a misleading figure — a low total driven by volume is not the same as a low unit cost.

**Insight 3 — Rental duration is uniform catalog policy, not customer behaviour.** The 3–7 day split is virtually even (191–212 films per band) and shows no trend. This is a catalog attribute describing how long a film *may* be kept, not how long customers actually keep it. Actual holding time must be measured from `rental_date` → `return_date` in the `rental` table.

**Actionable recommendation:** introduce pricing that scales with the allowed rental period, so the incentive works against over-long holds — a longer allowed period should not be cheaper. Success should be measured by tracking average actual holding time (`return_date - rental_date`) before and after the change.

---

## Key Concepts

- **`GROUP BY`**: collapses rows into groups; every non-aggregated column in `SELECT` must appear in `GROUP BY`.
- **`COUNT`, `SUM`, `AVG`**: aggregate functions that reduce each group to a single value.
- **`ROUND(value, n)`**: controls the precision shown in reports (2 decimals for money, 1 for minutes).
- **`ORDER BY` on an alias**: sorting by `"Film count"` (the column alias) is valid and keeps the query readable.
- **Aggregate vs. detail**: `SUM(replacement_cost)` scales with group size, while `AVG(replacement_cost)` normalises it — reporting one without the other can mislead.
- **The mean hides the distribution**: identical averages across groups can conceal identical *spreads* inside them. Always pair an average with a distribution or a dispersion measure.