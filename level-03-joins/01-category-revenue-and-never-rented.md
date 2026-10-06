# Level 3: Joins — Revenue by Category & Never-Rented Films

> **Database**: dvdrental (Sakila)
> **Date**: 2026-09-27
> **Status**: ✅ Solved

---

## Challenge: Category Revenue & Dead Catalogue

**Business Context**: The board wants a revenue report by film genre — but genre is not stored on the `film` table, so the query must traverse six tables to reach `category`. In parallel, the purchasing department asks which films generate **no revenue at all**, to decide how to manage the catalogue.

### SQL Solution — Part 1: Revenue by category (multi-table INNER JOIN)

```sql
SELECT
	c."name" "Category",
	COUNT(r.rental_id) "Rentals",
	SUM(p.amount) "Total Revenue"
FROM payment p
JOIN rental r ON p.rental_id = r.rental_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
JOIN film_category fc ON fc.film_id = f.film_id
JOIN category c ON c.category_id = fc.category_id
GROUP BY c.name
ORDER BY 3 DESC;
```

### Query Results

| Category | Rentals | Total Revenue |
|:---:|:---:|:---:|
| Sports | 1081 | 4892.19 |
| Sci-Fi | 998 | 4336.01 |
| Animation | 1065 | 4245.31 |
| Drama | 953 | 4118.46 |
| Comedy | 851 | 4002.48 |
| New | 864 | 3966.38 |
| Action | 1013 | 3951.84 |
| Foreign | 953 | 3934.47 |
| Games | 884 | 3922.18 |
| Family | 988 | 3830.15 |
| Documentary | 937 | 3749.65 |
| Horror | 773 | 3401.27 |
| Classics | 860 | 3353.38 |
| Children | 861 | 3309.39 |
| Travel | 765 | 3227.36 |
| Music | 750 | 3071.52 |

### SQL Solution — Part 2: Films that were never rented

```sql
SELECT
	f.title "Title"
FROM film f
LEFT JOIN inventory i ON i.film_id = f.film_id
LEFT JOIN rental r ON r.inventory_id = i.inventory_id
GROUP BY f.title
HAVING COUNT(r.rental_id) = 0
ORDER BY 1;
```

**Note on the aggregation column**: `HAVING COUNT(r.rental_id) = 0` counts *rentals*, not inventory copies. Counting `i.inventory_id` would answer a different question ("films with no copy in stock") and would silently miss any title that is stocked but has never been rented.

### Query Results

**42 films** — none of which holds a single copy in `inventory` (required for a purchase-policy decision). Examples:

| Title | Title | Title |
|:---|:---|:---|
| Academy Dinosaur* | Gump Date | Pearl Destiny |
| Alice Fantasia | Apollo Teen | Argonauts Town |
| Ark Ridgemont | Arsenic Independence | Boondock Ballroom |
| Butch Panther | Catch Amistad | Chinatown Gladiator |

\* `Academy Dinosaur` also appears in the false-positive list produced by a row-level filter — see Key Concepts.

---

## Business Insights & Actionable Recommendations

**Insight 1 — Revenue is concentrated at the top, but the spread is narrow.** Sports leads with 4,892.19 $ (1,081 rentals) while Music closes the list with 3,071.52 $ (750 rentals) — the top category earns roughly 1.6x the bottom one. No single genre dominates the catalogue, so revenue is not dependent on one product category.

**Insight 2 — The 42 non-earning films are a supply gap, not a demand problem.** These titles generate zero revenue simply because **the company holds no copy of them in stock** — they were never available to rent in the first place. The zero is a consequence of the catalogue never having been stocked, not of customers rejecting these titles.

**Insight 3 — Recommendation for purchasing.** The purchasing department should acquire these titles, ideally distributing them across both stores, so that catalogue breadth translates into rental revenue. As a default option this closes a revenue gap at low risk, because the titles occupy catalogue space without generating any return today.

---

## Key Concepts

- **Multi-table INNER JOIN chain**: `payment → rental → inventory → film → film_category → category` — five joins to reach the genre name, since the attribute lives at the far end of the schema.
- **`LEFT JOIN`**: keeps every row of the left table and pads unmatched right-side columns with `NULL` — the tool for finding *missing* relationships (films without rentals).
- **Row-level vs. group-level filtering (the core lesson)**: `WHERE r.rental_id IS NULL` filters **rows**, so any film with at least one idle copy survives — it even returned `Academy Dinosaur`, a title with 23 rentals, as "never rented". To ask about a *group* (the film as a whole) the filter must sit at group level: `GROUP BY ... HAVING COUNT(r.rental_id) = 0`.
- **Aggregate the right column**: count `r.rental_id` (rentals) when the question is about rentals — counting `i.inventory_id` answers a different question.
- **`FULL JOIN` is not a substitute for `LEFT JOIN`**: it also returns unmatched rows from the right side. Here the two produced identical output only because the foreign keys were consistent — the result was right by accident, not by design.
- **`EXISTS` / `NOT EXISTS`** is an equivalent and often clearer alternative for anti-joins.