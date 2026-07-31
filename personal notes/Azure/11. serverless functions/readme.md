# Lesson 11: Serverless Functions — Quick Revision Notes

Topics: Azure Functions Fundamentals, Triggers vs Bindings, Function Templates, Azure Cosmos DB & MongoDB, Function Authorization Levels.

---

## 1. Azure Functions — Core Concept

- **Azure Functions** = Microsoft's serverless, event-driven, compute-on-demand platform. Common pattern: **Backend-for-Frontend (BFF)** microservices.
- Write small blocks of code ("functions"), build/debug locally, then deploy.

### Breaking Down the Definition

| Term             | Meaning                                                                                                                      |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **Serverless**   | No need to provision/manage servers; pay only per execution (traditional model wastes ~70% of compute capacity sitting idle) |
| **Event-driven** | Code runs in response to **triggers**; infra managed automatically                                                           |

- **Supported languages**: C#, Java, JavaScript, Python, PowerShell.
- **Use cases**: APIs, microservices, bulk data processing, system integration, IoT.
- Comparable to **AWS Lambda** (if coming from AWS background).

---

## 2. Triggers vs Bindings (high-yield MCQ topic)

|                     | Triggers                                     | Bindings                                                                   |
| ------------------- | -------------------------------------------- | -------------------------------------------------------------------------- |
| **Purpose**         | Cause a function to **run**                  | Declaratively **connect a resource** to the function (input/output params) |
| **Direction**       | **Input only**                               | Can be **input AND output**                                                |
| **Cardinality**     | **One-to-one** — only 1 trigger per function | **Multiple bindings** allowed per function                                 |
| **Config location** | `function.json` (except C#)                  | `function.json` (except C#) — for Python, always in `function.json`        |

⚠️ MCQ trap: **Triggers = input-direction only; Bindings = input or output.** Also: **1 function = 1 trigger**, but can have many bindings.

### Common Trigger Types

- HTTP triggers
- Time (Timer) triggers
- Event Hub triggers
- Blob storage triggers
- Queue triggers

### Function Templates (preset trigger+binding combos)

| Template              | Use                                      |
| --------------------- | ---------------------------------------- |
| **HTTP**              | Triggered by HTTP requests               |
| **Timer**             | Scheduled/periodic events                |
| **Azure Cosmos DB**   | Create/update Cosmos DB data             |
| **Blob storage**      | Create/update Storage blobs              |
| **Queue storage**     | Process Storage queue messages           |
| **Event Grid**        | Process events via subscriptions/filters |
| **Event Hub**         | Process large volumes of events          |
| **Service Bus Queue** | Process Service Bus queue messages       |
| **Service Bus Topic** | Like Service Bus Queue, but topic-based  |

⚠️ MCQ trap (from quiz): To write data to **Azure Table Storage**, you'd use a **Table storage output binding** (not a trigger) — writing/output = binding's job, not a trigger's.

---

## 3. Azure Cosmos DB & MongoDB

- **Cosmos DB** = fully managed, globally scalable **NoSQL** database service; supports **5 database APIs** (incl. MongoDB-compatible API).
- For this course: Cosmos DB used to host a **MongoDB** database.

| MongoDB Concept          | Relational Equivalent |
| ------------------------ | --------------------- |
| **Database**             | Database              |
| **Collection**           | Table                 |
| **Document (JSON-like)** | Row/Record            |

- MongoDB = **document-based NoSQL**; schema is **flexible/unstructured** (can change over time), fast to read/write.

⚠️ MCQ trap: **Region matters** — Azure Functions on the **Linux Consumption Plan** are limited to specific regions; you should pick the **same region** for Cosmos DB as your Function App (reduces cost + latency), even though Cosmos DB itself is available in all regions.

---

## 4. Function Authorization Levels (classic matching MCQ)

Configured in `function.json` via `"authLevel"`.

| Level            | Meaning                                                                                  |
| ---------------- | ---------------------------------------------------------------------------------------- |
| **Anonymous**    | No key needed — endpoint is fully public                                                 |
| **Function**     | Requires a **function-specific API key** — key only works for that one function          |
| **Admin (Host)** | Requires a **master/admin key** — grants access to **all functions** in the Function App |

⚠️ MCQ trap — memorize scope of access per level:

- Anonymous → open to everyone
- Function → key scoped to **one function only**
- Admin/Host → key scoped to **entire function app** (all functions)

### Key Practical Notes

- Auth only applies on the **live/deployed server**, not local development (`localhost:7071`).
- Detailed endpoint security (networking, API Management, B2C) is **out of scope** — course focuses on **Function-level auth** only.
- After changing `authLevel` to `"function"` and republishing, the endpoint URL changes to include an appended **API key** — the old (keyless) URL stops working.

```bash
# Republish after changing authLevel in function.json
func azure functionapp publish <functionAppName>

# Serve functions locally for testing
func start
```

---

## 🔑 Key Terms Glossary

| Term                  | Definition                                                                                           |
| --------------------- | ---------------------------------------------------------------------------------------------------- |
| **Azure Functions**   | Serverless platform to run event-triggered code; used for APIs/microservices                         |
| **Serverless**        | No server provisioning needed; pay-per-execution model                                               |
| **Trigger**           | Causes a function to run; input-only; 1-to-1 with function                                           |
| **Binding**           | Declarative input/output connection to a resource; multiple allowed per function                     |
| **Cosmos DB**         | Fully managed, scalable Azure database service (relational + NoSQL support)                          |
| **MongoDB**           | NoSQL, document-based database; flexible schema; collections = tables, documents = JSON-like records |
| **Anonymous (auth)**  | No key required, public access                                                                       |
| **Function (auth)**   | API key scoped to a single function                                                                  |
| **Admin/Host (auth)** | Master key scoped to all functions in the app                                                        |

---

## 🔑 Quick MCQ Traps — Serverless Functions

- Triggers = **input-only**, **1-to-1** with a function; Bindings = **input/output**, **multiple allowed** per function.
- Writing output data (e.g., to Table Storage) = use an **output binding**, not a trigger.
- Function App on **Linux Consumption Plan** → region-limited; match Cosmos DB region to reduce cost/latency.
- MongoDB mapping: **Collection ≈ Table**, **Document ≈ Row** (but schema-flexible/JSON-like).
- Auth levels: **Anonymous** (public) < **Function** (per-function key) < **Admin/Host** (all-functions key).
- Authorization only takes effect on **deployed/live** functions, not local dev.
