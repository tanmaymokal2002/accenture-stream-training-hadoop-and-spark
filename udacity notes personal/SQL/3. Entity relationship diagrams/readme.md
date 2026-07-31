# SQL Course — Lesson 2: Entity Relationship Diagrams (ERD)

Topics: What is an ERD, Primary/Foreign Keys, Parch & Posey Database Structure.

---

## 1. What is an ERD?

- **ERD (Entity Relationship Diagram)** = a visual way to represent a database's structure.
- Shows:
  1. **Table names**
  2. **Columns** in each table
  3. **How tables relate/connect** to each other
- Each **box (table)** ≈ one **spreadsheet**.
- The **"crow's foot"** notation on connecting lines shows **how columns in one table relate to columns in another**.

---

## 2. Primary Key vs Foreign Key (core concept — very testable)

| Term                 | Meaning                                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Primary Key (PK)** | Unique identifier for each row in **its own table** (usually `id`)                                         |
| **Foreign Key (FK)** | A column in one table that **references the Primary Key of another table** — this is how tables are linked |

⚠️ MCQ trap: A **foreign key** always points to a **primary key** in a _different_ table — this relationship is what enables **joins** across tables.

---

## 3. Parch & Posey Database — 5 Tables

| Table          | Purpose                   |
| -------------- | ------------------------- |
| **web_events** | Logs web activity events  |
| **accounts**   | Company/account info      |
| **orders**     | Product order details     |
| **sales_reps** | Sales representative info |
| **region**     | Sales region info         |

### Table Structures & Relationships

**`web_events`**
| Column | Description |
|---|---|
| `id` | PK — unique event identifier |
| `account_id` | **FK** → `accounts.id` |
| `occurred_at` | Timestamp of event |
| `channel` | Channel of the web event |

**`accounts`**
| Column | Description |
|---|---|
| `id` | PK — unique account identifier |
| `name` | Account name |
| `website` | Account website |
| `lat` / `lang` | Latitude / Longitude |
| `primary_poc` | Primary point of contact |
| `sales_rep_id` | **FK** → `sales_reps.id` |

**`orders`**
| Column | Description |
|---|---|
| `id` | PK — unique order identifier |
| `account_id` | **FK** → `accounts.id` |
| `standard_qty`, `poster_qty`, `gloss_qty`, `total` | Quantities ordered |
| `occurred_at` | Timestamp of order |
| `standard_amt_usd`, `gloss_amt_usd`, `poster_amt_usd`, `total_amt_usd` | USD amounts |

**`sales_reps`**
| Column | Description |
|---|---|
| `id` | PK — unique sales rep identifier |
| `name` | Sales rep name |
| `region_id` | **FK** → `region.id` |

**`region`**
| Column | Description |
|---|---|
| `id` | PK — unique region identifier |
| `name` | Region name |

### Relationship Chain (how it all links)

```
region (id) ← sales_reps (region_id)
sales_reps (id) ← accounts (sales_rep_id)
accounts (id) ← orders (account_id)
accounts (id) ← web_events (account_id)
```

⚠️ MCQ trap: Notice **`accounts`** is the central hub — both `orders` and `web_events` link to it via `account_id`, and `accounts` itself links to `sales_reps` via `sales_rep_id`.

---

## 🔑 Key Terms Glossary

| Term                     | Definition                                                                     |
| ------------------------ | ------------------------------------------------------------------------------ |
| **ERD**                  | Entity Relationship Diagram — visual map of tables, columns, and relationships |
| **Primary Key (PK)**     | Unique identifier column for rows in its own table                             |
| **Foreign Key (FK)**     | Column referencing a Primary Key in another table — creates relationships      |
| **Crow's Foot Notation** | Diagram notation showing how tables' columns relate                            |

---

## 🔑 Quick MCQ Traps — ERDs

- **Foreign key** in one table = **Primary key** in another — this is the link that makes joins possible.
- Parch & Posey has **5 tables**: web_events, accounts, orders, sales_reps, region.
- **`accounts`** is the central table — connects to `orders`, `web_events`, and `sales_reps`.
- Relationship chain: **region → sales_reps → accounts → (orders, web_events)**.
- Each table = one "spreadsheet"; each row = one record; each column = one attribute.
