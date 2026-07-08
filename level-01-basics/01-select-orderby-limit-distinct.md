# Level 1: SELECT, ORDER BY, LIMIT, DISTINCT & Subqueries

> **Database**: dvdrental  
> **Date**: 2026-07-08  
> **Status**: ✅ Solved

---

## Challenge: High-Cost Short Films

**Business Context**: The store manager wants to analyze financial risk associated with short films (under 60 minutes) that have high replacement costs.

### Part 1: Top 5 Most Expensive-to-Replace Short Films

Find the 5 films under 60 minutes with the highest replacement cost, sorted descending.

```sql
SELECT 
    title AS "Tytuł",
    length AS "Długość (min)",
    replacement_cost AS "Koszt zastąpienia ($)",
    rental_rate AS "Cena wypożyczenia ($)"
FROM film
WHERE length < 60
ORDER BY replacement_cost DESC
LIMIT 5;
```

| Tytuł | Długość (min) | Koszt zastąpienia ($) | Cena wypożyczenia ($) |
|-------|:------------:|:--------------------:|:--------------------:|
| Doctor Grail | 57 | 29.99 | 2.99 |
| Cupboard Sinners | 56 | 29.99 | 2.99 |
| Ridgemont Submarine | 46 | 28.99 | 0.99 |
| Lust Lock | 52 | 28.99 | 2.99 |
| Side Ark | 52 | 28.99 | 0.99 |

### Part 2: Distinct Ratings Among Top 5 Films

Uses a **derived table** (subquery in `FROM` clause) to find unique ratings within the top 5 results.

```sql
SELECT DISTINCT rating
FROM (
    SELECT 
        title AS "Tytuł",
        rating,
        length AS "Długość (min)",
        replacement_cost AS "Koszt zastąpienia ($)",
        rental_rate AS "Cena wypożyczenia ($)"
    FROM film
    WHERE length < 60
    ORDER BY replacement_cost DESC
    LIMIT 5
) AS a;
```

| rating |
|--------|
| G |
| PG-13 |
| R |

---

## Key Concepts

- **`SELECT` with aliases** (`AS "alias"`) — renames columns for readability
- **`WHERE`** — filters rows before aggregation/ordering
- **`ORDER BY ... DESC`** — sorts results descending
- **`LIMIT`** — restricts number of returned rows
- **`DISTINCT`** — returns unique values only
- **Subquery in `FROM` (derived table)** — allows applying `DISTINCT` to an already filtered/limited result set