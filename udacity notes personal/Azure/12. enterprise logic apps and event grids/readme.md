# Lesson 12: Enterprise Logic Apps & Event Grid — Quick Revision Notes

Topics: Azure Logic Apps, Consumption vs Standard Plan, Logic Apps vs Azure Functions, SendGrid, Azure Event Grid Concepts.

---

## 1. Azure Logic Apps

- **Logic Apps** = workflows triggered by an event (e.g., data updates, app updates, conditional logic).
- Target audience: **business users** — "Designer-First" (**declarative**, visual/no-code) experience, unlike Functions which is code-first.
- Big strength: **200+ pre-built connectors** — includes social media (Facebook, Twitter), email, and services **outside Azure** too.

### Logic Apps vs Azure Functions

|                         | Logic Apps                                    | Azure Functions                  |
| ----------------------- | --------------------------------------------- | -------------------------------- |
| **Audience**            | Business users                                | Developers                       |
| **Approach**            | Declarative / visual designer                 | Code-first                       |
| **Flexibility/Control** | Lower (pre-built connectors, workflow-driven) | Higher (full coding flexibility) |
| **Trigger model**       | Event-triggered workflow                      | Event-triggered code             |

⚠️ MCQ trap: Both are **event-triggered**, but the key differentiator is **audience + approach**: Logic Apps = declarative/business-user-friendly; Functions = code/developer-friendly.

### Consumption vs Standard Plan (Logic Apps)

| Plan            | Model                          | Notes                                                                          |
| --------------- | ------------------------------ | ------------------------------------------------------------------------------ |
| **Consumption** | Pay-per-use                    | Fully managed, **multi-tenant** hosting; simpler; used in this course's lab    |
| **Standard**    | More complex pricing structure | Despite the name, requires **advanced configuration** (not the simpler option) |

⚠️ MCQ trap: **"Standard" plan ≠ simpler/default option** — Consumption is actually the simpler, pay-per-use, recommended-for-lab plan.

📌 Logic Apps can run **anywhere Azure Functions can run** — doesn't have to be hosted on Azure cloud.

---

## 2. SendGrid

- **SendGrid** = third-party email delivery service.
- Lets you send emails **without needing a dedicated email server**.
- Useful in Logic Apps for email-related automation (e.g., RSS feed update → send email).
- Advantage over Gmail/Outlook connectors: works with **any email account**, not tied to one provider.

---

## 3. Azure Event Grid — Core Concepts

- **Event Grid** = centralized **event management service** in Azure; integrates across multiple Azure services and external apps.
- Architecture style: think of it as **Microsoft's pub-sub (publish-subscribe)** system.

### Key Building Blocks

| Term                    | Definition                                                                             |
| ----------------------- | -------------------------------------------------------------------------------------- |
| **Events**              | Small packets of info describing **what happened** in a system                         |
| **Event Sources**       | **Where** the event originated                                                         |
| **Publishers**          | Entities (people/orgs) that send events to Event Grid — **can be outside Azure** too   |
| **Topic**               | The **endpoint** where publishers send their events                                    |
| **Event Subscriptions** | Define **what topics to listen to**; can **filter** incoming events                    |
| **Event Handlers**      | The service that **reacts** to the event (e.g., **Azure Functions** used as a handler) |

⚠️ MCQ trap: **Publishers are NOT limited to Azure-based sources** — Event Grid can accept events from outside Azure cloud too.

### Event Flow (conceptual)

```
Event Source → Publisher → Topic → Event Subscription (filter) → Event Handler (reacts)
```

### Operational Details

| Concept                           | Detail                                                                                                                     |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **Polling**                       | Repeatedly hitting an endpoint to check for new data (contrast with push-based event delivery)                             |
| **Event Subscription Expiration** | Events **expire at the end of the date** they were sent                                                                    |
| **Batching**                      | Custom topics publish events as an **array/batch** for efficiency; **max batch size = 1MB**; typical event size ≈ **64KB** |
| **Redelivery**                    | Event Grid **redelivers** an event if it doesn't receive confirmation the subscriber's endpoint accepted it                |

⚠️ MCQ traps (numbers to remember):

- Max **batch size = 1MB**.
- Typical **individual event size ≈ 64KB**.
- No delivery confirmation → Event Grid **retries/redelivers**.

---

## 🔑 Key Terms Glossary

| Term                              | Definition                                                                                       |
| --------------------------------- | ------------------------------------------------------------------------------------------------ |
| **Logic Apps**                    | Declarative, event-triggered workflows for business users; 200+ connectors                       |
| **Consumption Plan (Logic Apps)** | Pay-per-use, fully managed, multi-tenant — simpler option, used in labs                          |
| **Standard Plan (Logic Apps)**    | More complex pricing, requires advanced config — despite the name, not the default/simple choice |
| **SendGrid**                      | Third-party email delivery service; no dedicated mail server needed                              |
| **Event Grid**                    | Centralized Azure event management service; pub-sub style architecture                           |
| **Events**                        | Small data packets describing what happened                                                      |
| **Event Sources**                 | Where the event occurred                                                                         |
| **Publishers**                    | Senders of events to Event Grid (can be non-Azure)                                               |
| **Topic**                         | Endpoint where publishers send events                                                            |
| **Event Subscriptions**           | Define which topics to listen to + filtering rules                                               |
| **Event Handlers**                | Services that react to events (e.g., Azure Functions)                                            |
| **Polling**                       | Repeatedly checking an endpoint for new data                                                     |
| **Batching**                      | Grouping multiple events into one array publish (max 1MB)                                        |
| **Redelivery**                    | Re-sending an event if no acceptance confirmation received                                       |

---

## 🔑 Quick MCQ Traps — Logic Apps & Event Grid

- Logic Apps = **declarative/business-user-focused**; Azure Functions = **code/developer-focused**. Both are event-triggered.
- **Consumption Plan** = simpler, pay-per-use; **Standard Plan** = despite its name, the more complex/advanced option.
- Logic Apps can run **anywhere Functions can run** — not limited to Azure hosting.
- Event Grid publishers **can be external to Azure**.
- Event Grid = **pub-sub architecture**: Publisher → Topic → Subscription (filter) → Handler.
- Batch size limit = **1MB**; typical event size ≈ **64KB**.
- No delivery confirmation → Event Grid **redelivers** the event.
- Event subscriptions **expire at end of the send date**.
