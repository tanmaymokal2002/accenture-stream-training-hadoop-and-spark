# Lesson 6: Introduction to Microsoft Azure Development — Quick Revision Notes

Topics: What is Cloud Computing, Benefits/Drawbacks, Stakeholders, History of Cloud Computing, Azure Overview, IaaS/PaaS/SaaS.

---

## 1. What is Cloud Computing?

- **Cloud** = a collection of servers on the internet that store/manage data, run apps, deliver content (email, videos, etc.).
- **Cloud computing** = delivery of software and storage **over the Internet**.

**Benefits of cloud computing:**

- **Cost**
- **Scale**
- **Reliability**
- **Security**

### Three Major Types of Cloud Services

| Type        | Notes                                                          |
| ----------- | -------------------------------------------------------------- |
| **Public**  | e.g. Microsoft OneDrive, Microsoft Azure, Microsoft Office 365 |
| **Private** | dedicated to a single organization                             |
| **Hybrid**  | mix of public + private                                        |

- Main differences between them: **where they are deployed** and **who manages them**.

### What Cloud Developers Do

- Plan and design cloud-based apps
- Monitor, maintain, and support cloud applications
- Develop workflows and processes

---

## 2. Advantages & Disadvantages of Cloud Computing

**Advantages:**
| Advantage | Detail |
|-----------|--------|
| **Cost** | Provider handles upfront hardware costs + ongoing maintenance |
| **Scale** | Pay-as-you-go, elastic — expand as needed based on demand |
| **Reliability** | Easier backup/disaster recovery management |
| **Security** | Policies/controls already in place to protect data |

**Disadvantages:**

- Internet-based → prone to **outages** and speed fluctuations.
- Sensitive/private/core data is physically stored on **someone else's server**.
- Cloud services are **tailored/customized** to your specific instance → can make it **hard to switch providers** (vendor lock-in).

⚠️ MCQ trap: "Scale" advantage specifically ties to **elasticity** (pay-as-you-go, expand/contract with demand) — not just "big capacity."

---

## 3. Cloud Developer Stakeholders

As a cloud developer, you interact with:

- **Users**
- **Company executives**
- **I.T. Department**
- **Cloud Service provider**
- **Finance Department**

---

## 4. History/Timeline of Cloud Computing

| Year      | Milestone                                                                                                      |
| --------- | -------------------------------------------------------------------------------------------------------------- |
| **1969**  | JCR Licklider envisions the **"Intergalactic Computer Network"** — access info anywhere globally               |
| **1980s** | Supercomputing Centers form; commercial ISPs emerge (late '80s)                                                |
| **1990**  | Tim Berners-Lee invents the **World Wide Web**                                                                 |
| **1999**  | **Salesforce.com** launches — pioneers **SaaS** (Software-as-a-Service) delivery model                         |
| **2002**  | Amazon launches **AWS** (formal business unit by 2006)                                                         |
| **2006**  | Amazon launches **EC2** (Elastic Compute Cloud) — rent computers to run applications; enabled Netflix, Spotify |
| **2007**  | **Dropbox** launches — cloud storage becomes a commodity                                                       |
| **2008**  | Google launches **Google App Engine (GAE)** — a **PaaS**                                                       |
| **2010**  | **Microsoft Azure** launches (announced 2008) — covers SaaS, PaaS, IaaS                                        |
| **2011**  | IBM launches **SmartCloud** — private/public/hybrid cloud suite                                                |
| **2013**  | Google launches **Google Compute Engine (GCE)** — an **IaaS** component, spins up VMs on demand                |

⚠️ MCQ trap: Know **who pioneered what model**:

- **Salesforce (1999)** → pioneered **SaaS**
- **Google App Engine (2008)** → **PaaS** example
- **Google Compute Engine (2013)** → **IaaS** example (VMs on demand)
- **AWS EC2 (2006)** → early IaaS-style compute rental

---

## 5. Microsoft Azure Overview

Azure = a **public cloud computing platform**. Key value/benefits:

| Feature                        | Detail                                                                      |
| ------------------------------ | --------------------------------------------------------------------------- |
| **Scalability**                | Dynamically handles changes in volume, bandwidth, storage size              |
| **Availability**               | Redundant globally, **99.9%+ uptime** (per SLA by service)                  |
| **Security**                   | Replicated data protects against disasters; authentication secures access   |
| **Delivery pipeline services** | Source control, unit testing, integration testing, delivery, live dev tools |

- Azure spans multiple categories: **Compute, Analytics, Databases, AI, Machine Learning**.

**Azure products focused on in this course:**

- App Services
- Virtual Machines
- Azure SQL Databases
- Blob Storage
- Azure Active Directory
- Aspects of Azure Monitor (logs and alerts)

---

## 6. IaaS, PaaS, SaaS (Service Models)

| Model                                  | What it Provides                                               | Removes/Handles                                                                                  | Azure Example                                                                               |
| -------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| **IaaS** (Infrastructure as a Service) | Raw compute, storage, networking                               | Removes up-front hardware/software/test environment costs — **you manage the OS, runtime, apps** | **Azure Virtual Machines** — rent VMs on demand, you configure OS & software                |
| **PaaS** (Platform as a Service)       | Managed platform: networking, middleware, dev & database tools | Handles infrastructure + runtime for you — **you just deploy your code**                         | **Azure App Services** — deploy web apps without managing servers/OS                        |
| **SaaS** (Software as a Service)       | Complete, ready-to-use software application                    | Everything (infra, platform, app) is managed by provider — **you just use it**                   | **Microsoft Office 365** / **Microsoft OneDrive** — ready-made applications accessed online |

⚠️ MCQ trap — remember the "responsibility ladder":

```
IaaS  → You manage: OS, runtime, apps, data
PaaS  → You manage: apps, data (provider handles OS/runtime/middleware)
SaaS  → You manage: nothing but usage/data input (provider handles everything)
```

- More control + more management responsibility as you move **IaaS → PaaS → SaaS** in reverse (IaaS = most control/most responsibility; SaaS = least control/least responsibility).

---

## 🔑 Quick MCQ Traps — Intro to Azure

- Cloud computing benefits: **Cost, Scale, Reliability, Security** (the core 4 — memorize this list).
- Three cloud types: **Public, Private, Hybrid** — differ in deployment location & management.
- Disadvantage: cloud lock-in due to **customization to your instance**, making it hard to switch providers.
- **Salesforce (1999)** = first major **SaaS** pioneer.
- **AWS EC2 (2006)** → rentable compute; **Google App Engine (2008)** → PaaS; **Google Compute Engine (2013)** → IaaS (VMs on demand).
- **Azure launched in 2010** (announced 2008) — covers all three: IaaS, PaaS, SaaS.
- Azure SLA promises **99.9%+ uptime**.
- IaaS = most control, most management responsibility (e.g. **Azure VMs**).
- PaaS = provider manages infra/runtime, you manage app/data (e.g. **Azure App Services**).
- SaaS = fully managed, ready-to-use software (e.g. **Office 365, OneDrive**).
