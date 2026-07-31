# Lesson 9: Security and Monitoring Basics — Quick Revision Notes

Topics: Shared Responsibility Model, Azure Security Services, Azure AD/Entra ID, Authentication vs Authorization, OAuth 2.0 & MSAL, Monitoring & Logging.

---

## 1. Shared Responsibility Model (Security across On-Prem / IaaS / PaaS / SaaS)

As you move from On-Premises → IaaS → PaaS → SaaS, more security responsibility shifts to the **cloud provider**.

| Deployment Type | Physical Hardware | OS            | Network / App / Identity                          |
| --------------- | ----------------- | ------------- | ------------------------------------------------- |
| **On-Premises** | Developer/Org     | Developer/Org | Developer/Org (**all** responsibility)            |
| **IaaS**        | **Provider**      | Developer/Org | Developer/Org                                     |
| **PaaS**        | Provider          | **Provider**  | **Shared**                                        |
| **SaaS**        | Provider          | Provider      | Network/App → **Provider**; Identity → **Shared** |

⚠️ Big MCQ trap: **Identity security remains a SHARED responsibility all the way through PaaS and SaaS** — it never fully shifts to the provider, unlike network/app security which does shift fully with SaaS.

### Always the Developer's Responsibility (regardless of deployment type):

- **Account and access management** — giving/revoking the right access to the right people
- **Client endpoints** — securing endpoints handling confidential info
- **Data Governance & Rights Management** — preventing improper data sharing (e.g. emailing confidential docs externally)

⚠️ MCQ trap: No matter how much you move to the cloud (even full SaaS), **you never stop being responsible for**: access management, client endpoints, and data governance.

### Common quiz pattern: "Moving from on-prem to App Service (PaaS) — what are you no longer responsible for?"

- Answer generally: **physical hardware AND operating system security** (since App Service = PaaS, and PaaS shifts OS responsibility to the provider).

---

## 2. Azure Security Services (Overview — know what each does)

| Service                                                    | Purpose                                                                                                                    |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **Azure Active Directory (Azure AD) / Microsoft Entra ID** | Single sign-on (SSO) + multi-factor authentication (MFA)                                                                   |
| **App Configuration**                                      | Stores app settings in one secure location                                                                                 |
| **Key Vault API**                                          | Stores app **keys and secrets** securely                                                                                   |
| **Managed Identities**                                     | Part of Azure AD — streamlines giving an app/user access to other Azure resources (no need to manage credentials manually) |
| **Shared Access Signatures (SAS)**                         | Gives **external parties** limited, defined access to specific Azure resources                                             |
| **Role-Based Access Controls (RBAC)**                      | Manages **internal** access — who can do what to which resources                                                           |
| **Azure Monitor**                                          | Broad monitoring: log analytics, metrics, alerts                                                                           |
| **Application Insights**                                   | Part of Azure Monitor — focused on performance & key metrics                                                               |

### Scenario → Service Matching (classic MCQ format)

| Situation                                                                                     | Best Fit                                                                 |
| --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| Storing access keys/secrets currently on a single on-prem server                              | **Key Vault API**                                                        |
| Want to add single sign-on to applications                                                    | **Azure AD / Entra ID**                                                  |
| External contractor needs limited access to storage resources                                 | **Shared Access Signatures (SAS)**                                       |
| Want to identify/mitigate DDoS attacks                                                        | **Azure Monitor** (with alerts) — general monitoring/alerting capability |
| Need to control which employees can access which internal resources                           | **RBAC**                                                                 |
| Need to give an app itself secure access to another Azure resource (no hardcoded credentials) | **Managed Identities**                                                   |

⚠️ MCQ trap: **RBAC = internal access control; SAS = external, limited access.** Don't mix these up — RBAC manages your own team/organization's permissions, SAS is for outsiders.

---

## 3. Azure Active Directory (Azure AD) / Microsoft Entra ID

- **Microsoft Entra ID** = new name for **Azure Active Directory** (same service).
- Provides **SSO (Single Sign-On)** and **MFA (Multi-Factor Authentication)**.
- A **"tenant"** in Azure AD ≈ **an organization**.
- Used with **MSAL** (Microsoft Authentication Library) to implement "Sign in with Microsoft" buttons in apps.

### App Registration Concepts

- Registering an app in Azure AD does **not** require a deployed app — just a name and account-access settings.
- You must define **which accounts** can sign in:
  - Single tenant → only your org
  - Multi-tenant → any organizational directory
  - Multi-tenant + personal accounts → widest access
- After registration, you get:
  - **Application (client) ID**
  - **Client Secret** (generated separately, shown only once — must be saved immediately)

⚠️ MCQ trap: A common scenario — "I can log in with my account, but my friend cannot" → likely cause: the app registration's **supported account types** setting is too restrictive (e.g. limited to a single tenant/organization), so external/personal Microsoft accounts are blocked.

---

## 4. Authentication vs Authorization

| Term               | Meaning                                                                                 |
| ------------------ | --------------------------------------------------------------------------------------- |
| **Authentication** | Verifying **who the user is** ("are you who you say you are?")                          |
| **Authorization**  | Verifying **what the user is allowed to do/access** ("are you allowed to see/do this?") |

⚠️ Classic MCQ pairing — memorize this distinction cold. Authentication = identity check; Authorization = permission check.

---

## 5. OAuth 2.0 & MSAL

- **OAuth 2.0** = industry-standard protocol for **authorization**.
- Instead of every app storing its own usernames/passwords, apps **delegate** that responsibility to a **centralized identity provider** (e.g. Azure AD).
- **MSAL** (Microsoft Authentication Library) = Python (or other language) library implementing "Sign in with Microsoft" using OAuth 2.0, working with Azure AD.
- **ADAL** (Active Directory Authentication Library) = older library; ⚠️ does **NOT** support personal Microsoft accounts (MSAL is the modern replacement).

### Simplified OAuth 2.0 / MSAL Flow

1. User clicks "Sign in with Microsoft" → app requests an **authorization code** from `/oauth/v2.0/authorize`.
2. Identity provider (Microsoft) returns the **authorization code**, redirects to your app's specified URL.
3. App exchanges the authorization code (+ client ID + client secret + requested **scope**, e.g. `USER.READ`) for an **access token**, via `/oauth/v2.0/token`.
4. App receives the **access token**.
5. App uses the access token to call a **secure endpoint**.
6. The endpoint **validates the token** (outside your app's control) and returns secure data if valid.

⚠️ MCQ trap: The **token validation step happens on the resource/endpoint side**, not something the requesting app itself does.

### Common Deployment Issue

- App works locally (`localhost`) but authentication breaks after deployment → likely missing step: **the new/live redirect URI wasn't added/registered in Azure AD's app registration** (Azure AD only allows redirects to registered URIs).

---

## 6. Monitoring & Logging in Azure

**Benefits of logging:**

- Troubleshooting existing/preventing future problems
- Improving performance/maintainability
- Automating operations that would otherwise need manual intervention

### Azure Monitoring Capabilities

- Monitor **metrics** (performance, service quotas)
- **App-based logging**
- Send logs to **storage**
- Create **alerts**
- (Advanced, out of scope here: Application Insights, Log Analytics via Kusto query language)

### Scenario → Monitoring Tool Matching (classic MCQ pattern)

| Scenario                                                                  | Tool/Approach           |
| ------------------------------------------------------------------------- | ----------------------- |
| See overall request volume/traffic trends over time                       | **Metrics** monitoring  |
| Track specific failed login attempts on one page (app-level event detail) | **App/console logging** |
| Get notified when request count crosses a threshold in a time window      | **Alerts**              |

⚠️ MCQ trap (from the note in this lesson): Use **console logs**, not **app logs**, when setting up certain alerts — the distinction between log types can matter for which Azure feature captures the data.

### Flask-Specific Logging Note

- `print()` statements **do NOT reliably show up** in Flask app logs the way they would in a plain Python script.
- Must use Flask's **built-in logger** instead, with a configurable **minimum severity level** (e.g., only capture `warning` and above).
- ⚠️ MCQ trap: If log level is set to `error`, only **error-level (and higher, e.g. critical)** messages get logged — lower-severity messages (info, debug, warning) would **NOT** appear.

---

## 🔑 Key Terms Glossary

| Term                                  | Definition                                                                                 |
| ------------------------------------- | ------------------------------------------------------------------------------------------ |
| **Azure Active Directory (Entra ID)** | SSO + MFA provider                                                                         |
| **Tenant**                            | Roughly equivalent to an organization in Azure AD                                          |
| **OAuth2**                            | Industry-standard authorization protocol; delegates auth to a central identity provider    |
| **MSAL**                              | Microsoft Authentication Library — Python lib for Microsoft sign-in via OAuth2             |
| **App Configuration**                 | Secure storage for app settings                                                            |
| **KeyVault API**                      | Secure storage for keys/secrets                                                            |
| **Managed Identities**                | Streamlined Azure resource access for apps (part of Azure AD)                              |
| **Shared Access Signatures**          | Limited, external access to specific Azure resources                                       |
| **RBAC**                              | Internal access control — who can do what to which resources                               |
| **Transient Faults**                  | Temporary issues like lost network connectivity or timeouts; apps should handle gracefully |
| **Azure Monitor**                     | Broad monitoring service — metrics, logs, alerts                                           |
| **Application Insights**              | Azure Monitor component — performance/metrics focus                                        |
| **Authentication**                    | Verifying identity ("who are you")                                                         |
| **Authorization**                     | Verifying permission ("what can you do")                                                   |

---

## 🔑 Quick MCQ Traps — Security & Monitoring

- Security responsibility shifts **On-Prem → IaaS → PaaS → SaaS**, with more moving to the provider each step — but **identity stays shared** even at SaaS level.
- Developer **always** retains: account/access management, client endpoint security, data governance.
- **RBAC** = internal access control; **SAS** = external/limited access — don't confuse these.
- **Authentication** = identity verification; **Authorization** = permission verification.
- **OAuth 2.0** = authorization protocol; delegates login/credential handling to a central identity provider (e.g. Azure AD) instead of each app managing its own.
- **MSAL** supports personal Microsoft accounts; the older **ADAL** does not.
- "Works on localhost, breaks after deployment" (auth context) → usually a **missing/unregistered redirect URI** in Azure AD.
- Token **validation happens at the resource/endpoint**, not within the requesting app.
- Metrics → trends over time; App/console logs → specific event details; Alerts → threshold-based notifications.
- Flask `print()` ≠ reliable logging — use Flask's built-in **logger**, and only messages at or above the configured **severity level** get captured.
