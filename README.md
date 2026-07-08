# SQL Portfolio — dvdrental (Sakila) Database

A structured collection of SQL queries demonstrating progressive mastery — from basic `SELECT` statements to window functions and cohort analysis. Built on the **dvdrental** (Sakila sample) database.

## 🎯 Learning Approach

This portfolio follows a **project-driven, level-based progression**:

1. **Learn a concept** — Theory and fundamentals via [DataCamp](https://datacamp.com) (SQL Fundamentals track)
2. **Apply immediately** — Each level tackles a realistic business question on the dvdrental database
3. **Document & ship** — Every query is documented with context, SQL code, and results, then pushed here

This mimics real-world analyst workflow: receive a business question → write the SQL → present the output.

## 📁 Repository Structure

```
sql-portfolio/
├── README.md                          ← You are here
├── level-01-basics/                   ← SELECT, ORDER BY, LIMIT, basic filtering
├── level-02-aggregation/              ← GROUP BY, aggregate functions (SUM, COUNT, AVG)
├── level-03-joins/                    ← INNER/LEFT JOIN across multiple tables
├── level-04-advanced/                 ← HAVING, multi-table JOINs, subqueries
├── level-05-window-functions/         ← DENSE_RANK, ROW_NUMBER, PARTITION BY, CTEs
├── level-06-case-segmentation/        ← CASE WHEN for customer segmentation
├── level-07-date-analysis/            ← Date functions, EXTRACT, TO_CHAR, cohort patterns
└── level-08-running-totals/           ← Cumulative sums, running totals with window functions
```

Each level folder contains:
- `README.md` — Business context and questions for that level
- `queries.sql` — All SQL queries for the level
- `results.md` — Query outputs (markdown tables)

## 🗄️ Database: dvdrental

The **dvdrental** database (also known as Sakila) models a DVD rental store chain. Key tables:

| Table      | Description                          | ~Rows |
|------------|--------------------------------------|-------|
| `film`     | Movie catalog                        | 1,000 |
| `actor`    | Actors appearing in films            | 200   |
| `customer` | Registered customers                 | 599   |
| `rental`   | Rental transactions                  | 16,044|
| `payment`  | Payment records                      | 14,596|
| `inventory`| Physical copies of films per store   | 4,581 |
| `category` | Film genres                          | 16    |

Full schema: [PostgreSQL Tutorial — dvdrental ER Diagram](https://www.postgresqltutorial.com/postgresql-getting-started/postgresql-sample-database/)

## 🛠️ Tools

- **Database**: PostgreSQL (accessed via Hermes Agent MCP)
- **Editor**: VS Code / Hermes TUI
- **Version control**: Git + GitHub
- **Learning platform**: [DataCamp — SQL Fundamentals](https://app.datacamp.com/)

## 📊 Progress

| Level | Topic                          | Status |
|-------|--------------------------------|--------|
| 01    | Basics (SELECT, ORDER, LIMIT)  | ⬜     |
| 02    | Aggregation (GROUP BY)         | ⬜     |
| 03    | Joins (INNER, LEFT, multi)     | ⬜     |
| 04    | Advanced (HAVING, subqueries)  | ⬜     |
| 05    | Window Functions (DENSE_RANK)  | ⬜     |
| 06    | Segmentation (CASE WHEN)       | ⬜     |
| 07    | Date Analysis                  | ⬜     |
| 08    | Running Totals                 | ⬜     |

## 👤 Author

**Maciej Hercka** — aspiring Data Analyst / BI Developer, transitioning from 15+ years in HR and quality control to data.

- [GitHub: Hercka](https://github.com/Hercka)
- Learning in progress: SQL, Python, Power BI

---

*"Every analyst starts with a single SELECT."*