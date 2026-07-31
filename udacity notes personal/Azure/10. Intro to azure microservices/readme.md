# Lesson 10: Intro to Microservices — Quick Revision Notes

Topics: Microservices Architecture, Azure Compute Options (Serverless vs Orchestrators), Microservices Design Patterns, Compute Decision Flowchart, Cost Models (CapEx vs OpEx), Azure Functions Hosting Plans.

---

## 1. What is Microservices Architecture?

- **Microservices** = breaking an application into multiple small, independent components, each responsible for a specific piece of functionality.
- Based on Robert Martin's **Single Responsibility Principle**, but applied at the **architecture level** instead of the function level.
- Opposite of a **monolithic application** (one big interlaced codebase).
- Services can work independently, but may also depend on each other's inputs/outputs via **APIs**.
- Analogy: like organs in a body — each has a specific job; a failure in one can still ripple through the whole system over time (no single point of failure, but not zero risk either).

⚠️ MCQ trap: Microservices reduce (not eliminate) the risk of cascading failure — isolation ≠ total immunity.

### 7 Benefits of Microservices

| Benefit                     | Why                                                                    |
| --------------------------- | ---------------------------------------------------------------------- |
| **Agile product launch**    | Deploy/update each service independently; bugs isolated to one service |
| **Smaller "2-pizza" teams** | Small teams = less management overhead                                 |
| **Manageable codebase**     | Avoids technical debt of large monoliths                               |
| **Polyglot computing**      | Different services can use different languages/frameworks              |
| **Better fault-handling**   | Services decoupled — failure isolated                                  |
| **Decoupled databases**     | Schema changes isolated per service                                    |
| **Scalable infrastructure** | Scale horizontally/vertically per service as needed                    |

📌 Case study: **Netflix** — pioneer of microservices architecture (commonly referenced example).

---

## 2. Azure Microservices Compute Options — Two Major Categories

| Category                  | Description                                                                                | Azure Examples                                            |
| ------------------------- | ------------------------------------------------------------------------------------------ | --------------------------------------------------------- |
| **Serverless (FaaS)**     | Functions as a Service — no server management                                              | **Azure Functions**, **Azure Event Grid** (event routing) |
| **Service Orchestrators** | Manage services on dedicated nodes (VMs); handle health checks, scaling, traffic balancing | **AKS**, Service Fabric, Docker EE / Mesosphere DC/OS     |

### Orchestrator Comparison

| AKS (Azure Kubernetes Service)          | Service Fabric                                                      | Docker EE / DC/OS                                                                      |
| --------------------------------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Microsoft's managed Kubernetes          | Microsoft's own distributed systems platform (since 2001)           | Azure has no dedicated managed service — build your own via IaaS/Marketplace templates |
| Handles patching, upgrades, autoscaling | Originally built for media/home device content; leans Windows-based | —                                                                                      |
| Deploys Docker containers into clusters | Similar goals to Kubernetes but different ecosystem                 | —                                                                                      |

⚠️ **Kubernetes (AKS) has overtaken Service Fabric** as the de facto industry standard — course focuses on **AKS**, especially for **Linux-based Python container apps**.

✅ For this course's app: **Azure Functions** (FaaS, for APIs) + **AKS** (containers/orchestration for deployment).

---

## 3. Microservices Design Patterns (classic matching-type MCQ)

| Pattern                            | Definition                                                                                                                                                                  |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Ambassador**                     | Helper service that handles connectivity tasks (monitoring, logging, routing, authentication) on behalf of internal services — offloads common client-connectivity concerns |
| **Anti-corruption Layer**          | Façade between new and legacy applications — supports interoperability across different versions/dependencies                                                               |
| **Back-ends for Front-ends (BFF)** | Separate backend services created per client type (web, mobile, etc.) — **main pattern used in this course's project**                                                      |
| **Bulkhead**                       | Isolates critical resources (e.g., connection pools, thread handling) to prevent cascading failures — increases system resiliency                                           |

⚠️ MCQ trap — quick matching cheat sheet:

- "Isolates resources / prevents cascading failure" → **Bulkhead**
- "Supports interoperability across versions" → **Anti-corruption Layer**
- "Separate backend per client type" → **Back-ends for Front-ends**
- "Offloads monitoring/logging/routing/security" → **Ambassador**

---

## 4. Azure Compute Decision Flowchart (concept, not walkthrough)

General decision path example (from course):

> New service → doesn't need full control → not high-capacity workload → microservices architecture → event-driven, short-lived processes → **Azure Functions (Serverless/FaaS)**

Key idea: choice narrows down between **Serverless (FaaS)** vs **Service Orchestrators** based on:

- Need for full control over environment?
- Workload capacity/duration?
- Event-driven vs always-on?

---

## 5. Cost Models: CapEx vs OpEx

| CapEx (Capital Expenditure)                        | OpEx (Operational Expenditure)                 |
| -------------------------------------------------- | ---------------------------------------------- |
| Upfront investment (hardware, servers, racks)      | Pay-as-you-go, day-to-day usage cost           |
| Fixed costs, large upfront + long-term maintenance | Variable costs, little/no upfront cost         |
| Traditional on-prem model                          | **Cloud model** — matches usage to actual need |

⚠️ MCQ trap: Cloud computing (Azure) shifts organizations from **CapEx → OpEx**.

📌 Tool: **Azure Pricing Calculator** — used to estimate costs (concept only, no walkthrough needed).

---

## 6. Azure Functions Hosting Plans (high-yield MCQ topic)

| Plan                        | Timeout                                                               | Scaling                                                 | Best For                                                                            | Notes                                                                                                       |
| --------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Consumption**             | Default 5 min, **max 10 min**                                         | Auto scales up/down, incl. to zero                      | Default plan; pay only for runtime, no idle VM cost                                 | On-the-fly, no reservation needed                                                                           |
| **Premium**                 | Default 30 min, Azure **guarantees 60 min** (theoretically unlimited) | Pre-warmed/higher-performing VMs (1, 2, 4 core options) | Apps needing consistent readiness/performance                                       | Pay for cores/sec + memory used                                                                             |
| **Dedicated (App Service)** | v1: unlimited default & max; v2/v3: 30 min default, **unlimited max** | Manual VM scaling                                       | Legal/compliance needs (healthcare, legal, defense), or already using dedicated VMs | **Must manually enable "Always On"** or function won't fire properly; billed like standard App Service plan |

⚠️ MCQ traps:

- **Consumption max timeout = 10 minutes** (not unlimited) — need Premium/Dedicated for longer.
- **Dedicated Plan v1** = only version with unlimited default AND max timeout.
- Forgetting to enable **"Always On"** in Dedicated Plan → function app won't fire correctly.
- Healthcare/legal/defense clients → typically require **Dedicated Plan** (isolation/compliance).

### App Service Plan components:

- Region
- VM instance count
- VM instance size
- Pricing tier (Free, Shared, Premium, etc.) — course mostly uses **Free/Shared** tiers (billed by CPU minutes)

### Other Related Costs

| Item                     | Note                                                                                            |
| ------------------------ | ----------------------------------------------------------------------------------------------- |
| **Storage account**      | Auto-created if none exists; used by Functions runtime + storage triggers/bindings              |
| **Application Insights** | Monitoring for function apps (not enabled in course exercises); free tier has limited telemetry |
| **Network bandwidth**    | Free within same Azure region; **charged** for cross-region or outside-Azure transfers          |

---

## 🔑 Key Terms Glossary

| Term                               | Definition                                                                                 |
| ---------------------------------- | ------------------------------------------------------------------------------------------ |
| **Microservices Architecture**     | App split into small, independently deployable services, each with a single responsibility |
| **Monolithic Application**         | Single, tightly-coupled codebase (opposite of microservices)                               |
| **FaaS**                           | Functions as a Service — serverless compute (e.g., Azure Functions)                        |
| **Service Orchestrator**           | Manages deployment/health/scaling of services on VMs (e.g., AKS)                           |
| **AKS**                            | Azure Kubernetes Service — Microsoft's managed Kubernetes offering                         |
| **Ambassador Pattern**             | Offloads connectivity tasks (logging, routing, auth) from services                         |
| **Anti-corruption Layer**          | Façade enabling old/new app version interoperability                                       |
| **Back-ends for Front-ends (BFF)** | Separate backend per client type                                                           |
| **Bulkhead Pattern**               | Isolates resources to prevent cascading failure                                            |
| **CapEx**                          | Upfront, fixed capital cost model (on-prem)                                                |
| **OpEx**                           | Pay-as-you-go, variable operational cost model (cloud)                                     |
| **Consumption Plan**               | Default Azure Functions plan; auto-scale, pay-per-runtime, 10 min max timeout              |
| **Premium Plan**                   | Higher-performance Azure Functions plan; up to ~60 min guaranteed runtime                  |
| **Dedicated (App Service) Plan**   | Manual-scale plan on dedicated VMs; needs "Always On" enabled                              |

---

## 🔑 Quick MCQ Traps — Microservices & Compute

- Microservices reduce, but don't eliminate, cascading failure risk.
- **AKS** (not Service Fabric) is the modern industry-standard orchestrator taught in this course — used for Linux/Python containers.
- Pattern matching: **Bulkhead** = isolation/resiliency; **Anti-corruption Layer** = version interoperability; **BFF** = per-client backend; **Ambassador** = offloaded connectivity tasks.
- Cloud shifts cost model from **CapEx → OpEx**.
- **Consumption Plan max timeout = 10 minutes** — a very testable number.
- **Dedicated Plan v1** = only version with unlimited default + max timeout.
- Must manually enable **"Always On"** for Dedicated Plan functions to fire properly.
- Network transfer is **free within the same Azure region**, costs money cross-region/outside Azure.
- Healthcare/legal/defense → typically need **Dedicated Plan** for compliance/isolation.
