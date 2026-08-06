# Neo4j Practice — Graph Basics & Cypher: Notes (6th Aug 2026)

_New topic: graph databases, following on from the Hive and MongoDB notes. Same format: **what it's doing**, **why**, then the **command to run**._

---

## How to use this document

Neo4j models data as **nodes**, **relationships**, and **properties**, queried with its own language, **Cypher**. This document walks through the building blocks, then a worked example of modelling and building a small graph from scratch.

---

# Part 1 — What Is Neo4j

- **Native graph database**, first released in **2007**.
- The name breaks down as **Neo** ("new") + **4j** (originally built on **Java**).
- Uses **index-free adjacency**: each node holds direct physical pointers to its neighboring nodes, so traversing a relationship doesn't require an index lookup the way a foreign-key join would in a relational database — this is what makes multi-hop traversals fast regardless of overall graph size.
- Query language is **Cypher** — simple, readable, and designed specifically for expressing graph patterns (as opposed to adapting a row/table language like SQL to graph data).
- Supports **ACID transactions**, same guarantees you'd expect from a relational database.

---

# Part 2 — Building Blocks

## 2.1 Node

- Drawn as a **circle** in the Neo4j Browser.
- Every node can have a **label** — a category or type name — and a node can carry **multiple labels** at once.
- Examples: `Person`, `Movie`, `Company`.

```cypher
()
(:Person)
(:Person:Employee)
```

- `()` is an empty/anonymous node with no label; `(:Person:Employee)` shows a single node labelled as both `Person` and `Employee` simultaneously — useful when an entity genuinely belongs to more than one category.

## 2.2 Relationship

- Always has a **direction** — from one node to another.
- Always has a **type**, written in **UPPER_CASE** by convention.
- Must have a **start node** and an **end node**.
- **Unlike SQL**, where a relationship is inferred at query time via a foreign key + join, in Neo4j the relationship is a first-class object **stored directly** on disk, connecting the two nodes.

```cypher
(a) -[:KNOWS]-> (b)
(a) -[:WORKS_FOR]-> (b)
(c) <-[:MANAGES]- (b)
```

- The arrow (`->` or `<-`) shows which node is the start and which is the end; `(c) <-[:MANAGES]- (b)` reads as "`b` MANAGES `c`."

## 2.3 Property

- A **key–value pair** stored on either a node or a relationship.
- Supported types: `String`, `Integer`, `Float`, `Boolean`, `Date`, `List`.

```cypher
(:Person{name:"Ravi", age:24}) -[:WORKS_FOR{since:2023, role:"Dev"}]-> (:Company{name:"Accenture", city:"Chennai"})
```

- **Why this matters:** properties can live on the **relationship itself** (`since`, `role`), not just on the nodes it connects — something a relational join table would need a whole extra table to represent.

---

# Part 3 — Graph Data Modelling: A Company Employee System

**Why model in steps:** going straight to Cypher without first identifying nodes, relationships, and properties tends to produce an inconsistent graph — this four-step process mirrors how you'd design an ER diagram before writing SQL.

## Step 1 — Identify the Nodes

`Employee`, `Department`, `Project`, `City`

## Step 2 — Identify the Relationships

```cypher
Employee -[:WORKS_IN]-> Department
Employee -[:ASSIGNED]-> Project
Employee -[:LIVES_IN]-> City
Department -[:LOCATED_IN]-> City
```

## Step 3 — Attach Properties to Each Node

```
Employee   {name, age, salary, joinDate}
Department {name, budget}
Project    {name, deadline, status}
City       {name, state}
```

## Step 4 — Full Pattern

```
(:City) <-[:LIVES_IN]- (:Employee{name, age}) -[:WORKS_FOR]-> (:Department{name})
                              |
                        [:ASSIGNED_TO]
                              |
                              V
                        (:Project{name, status})
```

- Reading this as a sentence: an `Employee` **lives in** a `City`, **works for** a `Department`, and is **assigned to** a `Project` — the graph shape mirrors how you'd describe the relationships in plain English, which is the main appeal of graph modelling over normalized relational tables.

---

# Part 4 — Neo4j Browser Basics

## 4.1 Browser commands vs. Cypher queries

| Prefix   | Type            | Example                   | Purpose                                             |
| -------- | --------------- | ------------------------- | --------------------------------------------------- |
| `:`      | Browser command | `:clear`, `:help`, `:use` | Controls the **tool** (the Neo4j Browser UI itself) |
| _(none)_ | Cypher query    | `MATCH (n) RETURN n`      | Controls the **database**                           |

```cypher
:clear    -- clear all result cards from the screen
:help     -- show help for all browser commands
:use      -- switch database
```

- **Why this distinction matters:** a command starting with `:` never touches your data — it's purely a UI action — while anything without the colon is sent to the database engine as an actual query.

## 4.2 Return a plain text value

```cypher
RETURN "Accenture" AS Message
```

- `RETURN` on its own (no `MATCH`) evaluates a literal expression — handy for testing syntax or doing quick calculations without touching the graph at all.

## 4.3 Return numbers and booleans

```cypher
RETURN
  100 AS Integer_value,
  99.67 AS Float_value,
  true AS Boolean_value,
  "String1" AS String_value
```

## 4.4 Return calculations

```cypher
RETURN
  50000 * 12 AS Annual_Salary,
  100 / 4 AS Quarter,
  2026 - 2023 AS Experience,
  10 % 3 AS Remainder
```

- Standard arithmetic operators work directly inside `RETURN`, same as they would in a spreadsheet formula.

## 4.5 Return string operations

```cypher
RETURN
  toUpper("Hello world") AS Upper_example,
  toLower("HELLO") AS Lower_Example,
  size("Hello") AS String_Length,
  "Hello" + " " + "Neo4j" AS Concatenated
```

- `size()` on a string returns its character length; `+` concatenates strings directly, unlike SQL's `CONCAT()` or `||`.

---

# Part 5 — Creating & Querying Nodes

## 5.1 Create a first node, returning it

```cypher
CREATE (e:Employee {
  empId: 1,
  name: "Ravi Kumar",
  age: 24,
  salary: 45000
}) RETURN e
```

- `CREATE` adds a new node to the graph; the trailing `RETURN e` immediately shows you the node you just created, confirming the write.

## 5.2 Create a node without `RETURN`

```cypher
CREATE (e:Employee {
  empId: 2,
  name: "Priya",
  age: 23,
  salary: 48000
})
```

- **Why skip `RETURN`:** the node is still created either way — `RETURN` is purely for displaying output in that same query, not required for the write to take effect.

To see everything that currently exists in the graph:

```cypher
MATCH (n) RETURN n
```

- `MATCH (n)` with no label or filter matches **every node**, regardless of type — the graph equivalent of `SELECT * FROM <every table>`.

## 5.3 Match by label

```cypher
MATCH (e:Employee) RETURN e
```

- Restricts the match to nodes carrying the `Employee` label — equivalent to filtering by table name in a relational query.

## 5.4 Match by a property

```cypher
MATCH (e:Employee {name: "Priya"}) RETURN e
```

- Property filters go inside the `{}` right on the node pattern — equivalent to a SQL `WHERE name = 'Priya'`, but expressed as part of the pattern itself rather than a separate clause.

## 5.5 Return specific properties

```cypher
MATCH (e:Employee) RETURN
  e.name AS Name, e.age AS Age
```

```cypher
MATCH (e:Employee) RETURN
  e.name, e.age
```

- `e.property` accesses a single property off the matched node, same dot-notation you'd expect from any object-like structure; aliasing with `AS` is optional — without it, the column header just becomes the raw expression (`e.name`).

## 5.6 Count nodes

```cypher
MATCH (e:Employee) RETURN count(e) AS total_emp
```

```cypher
MATCH (n) RETURN count(n) AS Total
```

- `count()` works the same way as SQL's `COUNT()` — scoping the `MATCH` pattern (by label or not) determines what's being counted.

---

# Part 6 — Building a Mini Graph

**Target shape:**

```
        (Chennai)
            |
        LOCATED_IN
            |
(Ravi) --[:WORKS_IN]--> (Engineering) <--[:WORKS_IN]-- (Arjun)
                              |
                        [:MANAGED_BY]
                              |
                          (Priya)
```

## Step 1 — Clean start

**Why:** clear out any leftover nodes/relationships from earlier experiments before building a fresh, known graph.

```cypher
MATCH (n) DETACH DELETE n
```

- `DETACH DELETE` removes a node **and** any relationships attached to it in one step — plain `DELETE` would fail on a node that still has relationships pointing to or from it.

Verify the graph is empty:

```cypher
MATCH (n) RETURN count(n) AS Total
```

## Step 2 — Create the City node

```cypher
CREATE (city:City {
  name: "Chennai",
  state: "Tamilnadu",
  country: "India"
}) RETURN city
```

## Step 3 — Create the Department nodes

```cypher
CREATE
  (d1:Department {
    deptId: 101,
    name: "Engineering",
    budget: 500000
  }),
  (d2:Department {
    deptId: 102,
    name: "HR",
    budget: 200000
  })
RETURN d1, d2
```

- A single `CREATE` can define **multiple nodes at once**, comma-separated — both are written in one statement instead of two separate `CREATE` calls.

## Step 4 — Create the Employee nodes

```cypher
CREATE
  (e1:Employee {
    empId: 1,
    name: "Ravi",
    age: 24,
    salary: 45000,
    joinDate: date("2023-06-01")
  }),
  (e2:Employee {
    empId: 2,
    name: "Priya",
    age: 28,
    salary: 52000,
    joinDate: date("2024-04-15")
  }),
  (e3:Employee {
    empId: 3,
    name: "Arjun",
    age: 26,
    salary: 65000,
    joinDate: date("2022-03-10")
  })
RETURN e1, e2, e3
```

- `date("YYYY-MM-DD")` converts a string literal into Neo4j's native `Date` type, so date arithmetic and comparisons work correctly later — storing it as a plain string would lose that.

## Delete a specific node

```cypher
MATCH (e:Employee {name: "Priya"}) DETACH DELETE e
```

- Same pairing as Step 1's cleanup, but scoped to a single matched node instead of the whole graph — removes `Priya` and any relationships she's connected to, without touching `Ravi` or `Arjun`.

> **Where this leaves off:** the `City`, `Department`, and `Employee` nodes above are all created, but the connecting relationships from the target shape (`WORKS_IN`, `MANAGED_BY`, `LOCATED_IN`) haven't been created yet in the notes as given — that's the natural next step to complete the mini graph.

---

## Quick recap

| Concept                          | Key point                                                                                              |
| -------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Node                             | `()`, drawn as a circle; can carry multiple labels, e.g. `(:Person:Employee)`                          |
| Relationship                     | Directed, typed (`UPPER_CASE`), stored directly — not inferred via a join                              |
| Property                         | Key–value pair on a node **or** a relationship; types: String, Integer, Float, Boolean, Date, List     |
| Index-free adjacency             | Nodes hold direct pointers to neighbors — traversals skip index lookups entirely                       |
| Cypher                           | Neo4j's graph-native query language                                                                    |
| Browser commands (`:`) vs Cypher | `:` commands control the UI tool; everything else is a query against the database                      |
| `CREATE`                         | Adds node(s)/relationship(s); multiple nodes can be comma-separated in one statement                   |
| `MATCH (n)`                      | No label/filter = matches every node, like `SELECT *`                                                  |
| `MATCH (:Label {prop: value})`   | Filters by label and/or property, inline in the pattern                                                |
| `count()`                        | Same role as SQL's `COUNT()`, scoped by whatever the `MATCH` pattern matched                           |
| `DETACH DELETE`                  | Deletes a node **and** all its relationships in one step; plain `DELETE` fails if relationships remain |
| `date("YYYY-MM-DD")`             | Converts a string literal to Neo4j's native `Date` type                                                |
