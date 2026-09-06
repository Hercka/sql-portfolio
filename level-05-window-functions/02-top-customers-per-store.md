# Level 5: Window Functions — Top Customers per Store (RANK / DENSE_RANK / NTILE)

> **Database**: dvdrental (Sakila)
> **Date**: 2026-09-06
> **Status**: ✅ Solved

---

## Challenge: Top Spenders per Store

**Business Context**: The marketing department is designing a loyalty program. Management needs to know who the top-spending customers are **within each store** (cross-store comparison is meaningless — the stores have different customer bases) and which spending quartile each customer falls into. This is the classic interview question: *"top N users per group."*

### SQL Solution (3 layers in one query: CTE aggregation → window ranking → outer filter)

```sql
SELECT
	store_id,
	Klient,
	ROUND(total,2),
	Rank,
	DenseRank,
	Kwartyl
FROM(
	WITH ranking AS (
		SELECT
			c.customer_id,
			CONCAT ( c.first_name, ' ', c.last_name) AS Klient
			, c.store_id
			, SUM(p.amount) AS total
		FROM customer c
		JOIN payment p ON p.customer_id = c.customer_id
		GROUP BY c.customer_id, c.store_id
		)
	
	SELECT 
		customer_id,
		Klient,
		store_id,
		total,
		RANK() OVER (PARTITION BY store_id ORDER BY total DESC) AS Rank,
		DENSE_RANK() OVER (PARTITION BY store_id ORDER BY total DESC) AS DenseRank,
		NTILE(4) OVER(PARTITION BY store_id ORDER BY total DESC) AS Kwartyl
	FROM ranking)
WHERE Rank <=5
ORDER BY store_id, Rank;
```

### Query Results

**Store 1:**

| Klient | Wydatki ($) | Rank | DenseRank | Kwartyl |
|--------|:---:|:---:|:---:|:---:|
| Eleanor Hunt | 211.55 | 1 | 1 | 1 |
| Clara Shaw | 189.60 | 2 | 2 | 1 |
| Tommy Collazo | 183.63 | 3 | 3 | 1 |
| Marcia Dean | 166.61 | 4 | 4 | 1 |
| Mike Way | 162.67 | 5 | 5 | 1 |

**Store 2:**

| Klient | Wydatki ($) | Rank | DenseRank | Kwartyl |
|--------|:---:|:---:|:---:|:---:|
| Karl Seal | 208.58 | 1 | 1 | 1 |
| Marion Snyder | 194.61 | 2 | 2 | 1 |
| Rhonda Kennedy | 191.62 | 3 | 3 | 1 |
| Ana Bradley | 167.67 | 4 | 4 | 1 |
| Curtis Irby | 167.62 | 5 | 5 | 1 |

### Extended Analysis: Frequency vs. Ticket Size (Top 10 per Store)

```sql
SELECT
	store_id,
	Klient,
	payments,
	ROUND (total/ payments,2) as "Sredni wydatek",
	ROUND(total,2) AS "Wydatki ($)"
FROM(
	WITH ranking AS (
		SELECT
			c.customer_id,
			CONCAT ( c.first_name, ' ', c.last_name) AS Klient,
			COUNT(payment_id) AS payments
			, c.store_id
			, SUM(p.amount) AS total
		FROM customer c
		JOIN payment p ON p.customer_id = c.customer_id
		GROUP BY c.customer_id, c.store_id
		)
	
	SELECT 
		customer_id,
		Klient,
		payments,
		store_id,
		total,
		RANK() OVER (PARTITION BY store_id ORDER BY total DESC) AS Rank
	FROM ranking)
WHERE Rank <=10
ORDER BY store_id, Rank;
```

**Key finding**: among the 20 customers in the top 10 of both stores, the average spend per transaction is remarkably flat — from **$4.27 (Marcia Dean)** to **$5.08 (Ana Bradley)** — while the number of payments ranges from **31 to 45**. Total spend is therefore almost entirely a function of **transaction count**, not ticket size.

---

## Business Insights & Actionable Recommendations

**Insight 1 — Revenue is volume-driven, not price-driven.** With rental prices fixed between $0.99 and $6.99, per-transaction spend is practically constant across the entire customer base (approx. $4.3–$5.1). The single lever that moves revenue is **frequency of renting**. Any loyalty program must therefore target *more rentals per customer*, not higher value per rental.

**Insight 2 — The loyalty mechanism should be "every Nth rental free", not a percentage discount.** A percentage discount shrinks margin on an already-low ticket without creating an incentive to rent more. A "free Nth rental" mechanic rewards the *next* transaction, pushes customers past their usual stopping point, and builds habitual renting. This is the mechanism our analysis supports.

**Insight 3 — Target the critical mass: customers with 20–29 orders.** This band holds **400 customers (66.8% of the base)** — the largest concentration of spend potential. A small increase in their average frequency translates into the biggest aggregate revenue gain. By contrast, the 40+ band contains only **3 customers (0.5%)** who are already at the observed activity ceiling (~1.4 rentals per week) — discounting them would subsidize behavior they already perform without any upside.

**Insight 4 — Validate with tenure analysis before launch.** Low-frequency customers (<15 orders) may simply be **new customers**, not disengaged ones. Before targeting this group, the team should measure how long customers needed to reach their current order count (time-to-N-orders / cohort analysis). Segmenting "new" from "dormant" is a prerequisite for a cost-effective campaign — the same discipline as separating a process that is out of spec from one that simply started recently.

---

## Key Concepts

- **`RANK()` vs `DENSE_RANK()`**: both rank rows within a partition; `RANK` leaves gaps after ties (1,2,2,4), `DENSE_RANK` does not (1,2,2,3). Choose `RANK` when counting "positions" (top 5), `DENSE_RANK` when counting "distinct value levels".
- **`NTILE(n)`**: splits rows into n equal-sized buckets — a quick quartile/decile segmentation (bucket 1 = highest spend).
- **`PARTITION BY`**: resets the ranking per group (here: per store), enabling the "top N per group" pattern.
- **Filtering by rank requires an outer layer**: window functions are computed after `WHERE`, so you cannot filter on a rank in the same `SELECT` that creates it — wrap the query and filter outside.
- **`GROUP BY c.customer_id, c.store_id`**: explicit grouping keeps the query portable (PostgreSQL's functional dependency would allow omitting `store_id`, but MySQL/SQL Server would reject it).
- **Pitfall discovered**: `NTILE(n) OVER ()` **without `ORDER BY`** assigns buckets in arbitrary row order — the buckets are meaningless (the top customer landed in bucket 5). An `ORDER BY` inside `OVER()` is mandatory for NTILE to reflect actual values.