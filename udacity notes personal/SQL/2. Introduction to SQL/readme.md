# SQL Course — Lesson 1: Introduction to SQL

Topics: What is SQL, Advantages of Relational Databases, SQL vs NoSQL, Table Structure & Data Types, Popular SQL Databases, Basic Query Syntax.

---

## 1. What is SQL?

- **SQL** = **S**tructured **Q**uery **L**anguage — it's a **language**, used beyond just databases, but most known for interacting with **relational databases**.
- Think of a (relational) database as: **a bunch of spreadsheets, all sitting in one place, that can talk to each other**.

---

## 2. Advantages of Relational Databases (SQL) — 5 Key Points

1. **Easy to understand** (readable, close to English syntax)
2. Allows **direct access** to data
3. Allows **auditing and replication** of data
4. Great for **analyzing multiple tables at once** (joins)
5. Enables **more complex analysis** than dashboard tools (e.g., Google Analytics)

⚠️ MCQ trap: SQL's edge over dashboard tools = ability to answer **complex, custom, multi-table** questions — not just pre-built reports.

---

## 3. Why Businesses Like Databases

| Benefit            | Explanation                                                                          |
| ------------------ | ------------------------------------------------------------------------------------ |
| **Data integrity** | Only intended data gets entered; access is controlled by user permissions            |
| **Fast access**    | SQL queries return results quickly, even on large datasets; code can be optimized    |
| **Easy sharing**   | Multiple users access the **same, consistent** data — everyone sees the same results |

---

## 4. SQL vs NoSQL

|                     | SQL                                            | NoSQL                                             |
| ------------------- | ---------------------------------------------- | ------------------------------------------------- |
| **Full form**       | Structured Query Language                      | **N**ot **o**nly **SQL**                          |
| **Data model**      | Structured, tabular (rows/columns), relational | Flexible/non-tabular (e.g., document-based)       |
| **Best suited for** | Structured, spreadsheet-like data              | Web-based data, unstructured/semi-structured data |
| **Example**         | MySQL, PostgreSQL                              | **MongoDB** (most popular NoSQL example)          |

⚠️ MCQ trap: **NoSQL = "Not only SQL"** — not "No SQL at all." It's a different data-handling approach, not a total rejection of SQL concepts.

---

## 5. Table Structure & Data Types

- Data is stored in **tables** (like Excel spreadsheets):
  - **Rows** = individual records (a transaction, person, company, etc.)
  - **Columns** = attributes of each record (name, location, unique ID, etc.)

Example table:
| id | name | website | lat | long | primary_pos | sales_rep_id |
|---|---|---|---|---|---|---|
| 1001 | Walmart | www.walmart.com | 40.2364985 | -75.1032974 | Tamara Tuma | 321500 |
| 1011 | Exxon Mobil | www.exonmobil.com | 41.1691583 | -73.8493737 | Sung Shields | 321510 |

### Key Rule: Column Data Types Must Match

- **All values in the same column must share the same data type** (quantitative, discrete/string, etc.).
- ⚠️ MCQ trap: If **even one row** has a string value in a numeric column, the **entire column** can be forced into a text/string data type — this **breaks math operations** on that column.
- Why it matters: **consistent column types = one of the main reasons databases are fast**, even with huge amounts of data.

---

## 6. Popular SQL Databases

| Database                  | Note                                                                   |
| ------------------------- | ---------------------------------------------------------------------- |
| **Postgres (PostgreSQL)** | Used in this course; open-source; rich library of analytical functions |
| **MySQL**                 | Popular, but missing some date-modification functions Postgres has     |
| **Access**                | Microsoft desktop database tool                                        |
| **Oracle**                | Enterprise-grade DB                                                    |
| **Microsoft SQL Server**  | Enterprise-grade DB                                                    |

- SQL can also be used **within other frameworks**: Python, Scala, Hadoop.

⚠️ MCQ trap: Different SQL databases have **minor syntax/function differences** (e.g., MySQL lacks some Postgres date functions), but **core SQL skills transfer across environments**.

---

## 7. Basic Query Example (syntax preview)

```sql
SELECT account_id,
       occurred_at,
       standard_qty,
       gloss_qty,
       poster_qty
FROM orders
WHERE (standard_qty = 0 OR gloss_qty = 0 OR poster_qty = 0)
AND occurred_at >= '2016-10-01';
```

**What this does:**

- `SELECT` → choose which columns to return
- `FROM` → specify the table (`orders`)
- `WHERE` → filter rows: any order where **at least one** quantity type is `0` (using `OR`), **AND** the order occurred on/after Oct 1, 2016

⚠️ Note the **operator precedence**: the `OR` conditions are grouped in parentheses so they're evaluated together _before_ being combined with `AND` — without parentheses, the logic could change.

---

## 🔑 Key Terms Glossary

| Term                    | Definition                                                               |
| ----------------------- | ------------------------------------------------------------------------ |
| **SQL**                 | Structured Query Language — used to query/manage relational databases    |
| **Relational Database** | Data organized in related tables (like linked spreadsheets)              |
| **NoSQL**               | "Not only SQL" — flexible, non-tabular database approach (e.g., MongoDB) |
| **Table**               | Structure holding rows (records) and columns (attributes)                |
| **Postgres**            | Open-source relational database used in this course                      |
| **SELECT**              | SQL clause to choose columns to return                                   |
| **FROM**                | SQL clause to specify the source table                                   |
| **WHERE**               | SQL clause to filter rows based on conditions                            |

---

## 🔑 Quick MCQ Traps — Intro to SQL

- SQL is a **language**, not just "a database."
- Relational database ≈ **connected spreadsheets**.
- NoSQL = **"Not only SQL"**, not "no SQL." **MongoDB** = most popular NoSQL example.
- SQL's key advantage over dashboards = handling **complex, multi-table** questions.
- **All values in a column must share one data type** — one stray string can force the whole column to text, breaking math operations.
- Consistent column types = **why databases are fast** even at large scale.
- SQL syntax has **minor differences across databases** (e.g., MySQL vs Postgres date functions), but skills are largely **transferable**.
- `WHERE` with mixed `AND`/`OR` → **use parentheses** to control logic grouping.
