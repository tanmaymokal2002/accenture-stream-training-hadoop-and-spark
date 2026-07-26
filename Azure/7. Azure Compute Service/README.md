# Lesson 7: Azure Compute Services — Quick Revision Notes

Topics: Compute Service Options, Resource Hierarchy, Virtual Machines, App Service, VM vs App Service decision-making, Azure CLI & SSH, GitHub basics, Key Terms.

---

## 1. Azure Compute Service Options (Overview)

Azure offers many compute services, each with different use cases:

- **Virtual Machines** — IaaS
- **App Services** — PaaS
- **Azure Batch** — large-scale/high-performance compute
- **Azure Functions** — serverless, event-driven
- **Container Instances** — serverless Docker containers
- **Service Fabric** — Microsoft's distributed systems platform (like Kubernetes)
- **Azure Kubernetes Service (AKS)** — managed Kubernetes

This lesson focuses on **Virtual Machines** and **App Services**.

---

## 2. Azure Resource Hierarchy

```
Azure Account
   └── Subscription(s)
         └── Resource Group(s)
               └── Resources (VMs, App Services, Storage, etc.)
```

| Level              | Purpose                                                                                                                                                         |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Account**        | Top level                                                                                                                                                       |
| **Subscription**   | Billing/management boundary — may have multiple (e.g. dev/test vs production)                                                                                   |
| **Resource Group** | Organizes related resources (e.g. by project or region) for easier management                                                                                   |
| **Region**         | Physical location of data center(s); a region can have multiple data centers networked via low-latency links. 60+ regions worldwide, available in 140 countries |

### Choosing a Region — 3 factors:

- **Service availability** — not all services available in all regions
- **Performance** — latency; consider dev/test (near yourself) vs production (near end-user)
- **Cost** — prices vary by region; cheapest region viable if latency isn't a concern

⚠️ MCQ trap: Dev/test → choose region **close to yourself**; production → choose region **close to your end users**.

### Creating a Resource Group (Portal steps)

1. Azure Portal homepage → "Create a Resource Group"
2. Search "Resource group" → Create
3. Select subscription, name it, select region
4. **Review** before creating (check for typos/wrong region — especially important later with CLI, since misnamed resources can break scripts)
5. Click Create

---

## 3. Virtual Machines (IaaS)

Azure VMs = **Infrastructure as a Service** — full virtual machines in the cloud.

**Benefits:**

- Full **access & control** of the VM
- Lower up-front cost vs buying/maintaining physical hardware
- Supports both **Linux and Windows**
- Multiple types (compute-optimized, memory-optimized, etc.) with varying CPU/RAM/storage
- Supports **custom images** — good for migrating on-premises servers to cloud
- Can group VMs for **high availability, scalability, redundancy** via:
  - **Virtual Machine Scale Sets**
  - **Load Balancers**

**Limitations:**

- More **expensive**
- More **time-consuming** for developers to manage vs other compute options

---

## 4. App Service (PaaS)

Azure App Service = **Platform as a Service** — HTTP-based hosting for web apps, REST APIs, mobile backends. Developer focuses on the app; Azure handles infrastructure.

**Benefits:**

- Supports multiple languages: **.NET, .NET Core, Java, Ruby, Node.js, PHP, Python**
- High availability, **auto-scaling**, supports Linux & Windows environments
- **Continuous deployment** via GitHub, Azure DevOps, or any Git repo
- Two scaling types:
  - **Vertical scaling** — change resources (vCPUs/RAM) via pricing tier
  - **Horizontal scaling** — change the number of VM instances running the app
- Three pricing tiers: **Dev/Test, Production, Isolated** (free tier available in Dev/Test)

**Limitations:**

- **Limited access** to host server — can't control underlying OS or install custom software
- You **always pay** for the service plan, even when app isn't actively running
- **Hardware limits**: max ~14 GB memory, 4 vCPU cores per instance
- Limited to the **supported languages only**

⚠️ MCQ trap: App Service = **always billed**, even when idle (unlike some serverless options). VM = **you control the OS**; App Service = **you do NOT control the OS**.

---

## 5. Virtual Machine vs App Service — Decision Guide

| Choose...           | When...                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **Virtual Machine** | Need control of underlying OS, using custom/specialized software, dedicated/isolated hardware required (e.g. compliance/security mandates) |
| **App Service**     | Lightweight apps/services, don't need high-performance compute, cost-conscious, want auto-scaling without managing infrastructure          |

### Practice Scenarios (from the lesson)

| Scenario                                                                                           | Best Choice         | Why                                                                      |
| -------------------------------------------------------------------------------------------------- | ------------------- | ------------------------------------------------------------------------ |
| Lightweight microservice APIs, cost-conscious, low scaling needs                                   | **App Service**     | Lightweight workload, low cost priority, minimal infra management needed |
| Government contract requiring **dedicated servers** for security + separate servers per department | **Virtual Machine** | Need isolated/dedicated hardware + full OS control for compliance        |
| New product, uncertain demand, currently ~5GB/2 CPUs but could grow massively                      | **App Service**     | Auto-scaling handles uncertain/variable growth without over-provisioning |

---

## 6. Deploying an App to a VM — Correct Order

1. **Create or utilize a resource group**
2. **Create the Virtual Machine**
3. **Connect to the VM** (via SSH)
4. **Install any app dependencies**
5. **Run the application**

⚠️ MCQ trap: If everything works _inside_ the VM but the **public IP gives a connection error**, a likely cause is that the **inbound port (e.g. port 80) wasn't opened** in the VM's network security rules — the app runs fine locally, but external traffic can't reach it.

### VM Creation Details (Portal)

- Resource Group, VM Name, Region (closest available)
- Image: e.g. Ubuntu Server LTS
- Size: e.g. Standard B1ls
- Authentication: password or SSH key
- **Inbound Port Rules**: must allow relevant ports (e.g. **22** for SSH, **80** for HTTP)

### Connecting & Deploying (typical flow)

```bash
# Get VM's public IP
az vm list-ip-addresses -g <RESOURCE-GROUP> -n <VM-NAME>

# Copy files to VM via secure copy
scp -r <SOURCE-DIR> <ADMIN-NAME>@<PUBLIC-IP>:<TARGET-DIR>

# Connect via SSH
ssh <ADMIN-NAME>@<PUBLIC-IP>

# On the VM: install dependencies, e.g.
sudo apt-get -y update && sudo apt-get -y install nginx python3-venv

# Set up Python virtual environment
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt

# Run the app
python application.py
```

- **Nginx** is commonly configured as a **reverse proxy** — redirects incoming traffic on port 80 to the app running on a local port (e.g. 3000).
- Config file placed in `/etc/nginx/sites-available/`, then symlinked into `/etc/nginx/sites-enabled/`, then `sudo service nginx restart`.

### Cleanup

- Delete the whole **resource group** to remove all associated resources at once (Portal or CLI: `az group delete -n <RESOURCE-GROUP>`).
- ⚠️ Azure free account limits: **750 hours** of free VM hosting — delete VMs after exercises to avoid charges.

---

## 7. Azure CLI & SSH

**Azure CLI** = text-based, command-line alternative to the Azure Portal — useful for automation.

```bash
az login                     # log in to Azure via CLI
```

**Creating a VM via CLI:**

```bash
az vm create \
  --resource-group "resource-group-west" \
  --name "linux-vm-west" \
  --location "westus2" \
  --image "UbuntuLTS" \
  --size "Standard_B1ls" \
  --admin-username "udacityadmin" \
  --generate-ssh-keys \
  --verbose
```

- If no `--location` is passed, it **defaults to the resource group's region** — but note a VM size may not be available in that region, so you may need to specify a different one.

**Opening a port via CLI:**

```bash
az vm open-port \
  --port "80" \
  --resource-group "resource-group-west" \
  --name "linux-vm-west"
```

### SSH (Secure Shell)

- Protocol for securely connecting to remote systems; encrypts the connection.

```bash
ssh-keygen -t rsa -b 2048    # generates public (id_rsa.pub) and private (id_rsa) key in ~/.ssh/
```

**Summary flow:** Install Azure CLI → Create VM → Use SSH to connect.

---

## 8. GitHub Basics

**GitHub** = cloud platform for storing, managing, and collaborating on code; built around **Git** (version control system).

|                        | Git 🛠️                                           | GitHub ☁️                               |
| ---------------------- | ------------------------------------------------ | --------------------------------------- |
| **What is it?**        | Version control system for tracking code changes | Cloud platform hosting Git repositories |
| **Where does it run?** | Locally on your computer                         | Online, accessible anywhere             |
| **Who maintains it?**  | Open-source (created by Linus Torvalds)          | Owned by Microsoft                      |
| **Main Purpose**       | Track code changes                               | Enable collaboration, hosting, sharing  |

⚠️ MCQ trap: **Git ≠ GitHub** — Git is the underlying version control tool (local); GitHub is a cloud hosting platform built around Git.

**Basic Git workflow:**

```bash
git init                          # initialize a repo
git add .                         # stage all changes
git commit -m "Initial commit"    # save changes locally
git push origin main              # upload to GitHub
```

**Key GitHub features:** version control, collaboration (branches/pull requests), cloud storage, CI/CD integration, public/private repos.

---

## 9. Deploying an App Service (Portal steps)

1. Homepage → "Create a resource" → search "Web App" → Create
2. Select subscription & resource group
3. Enter a **globally unique** app name (unique across all of Azure, not just your account)
4. Publish: "Code"; Runtime Stack: e.g. Python 3.10+; OS: Linux
5. Select region; create/select an **App Service Plan**
6. SKU/size: e.g. **F1 (Free)** tier
7. Review + Create

**Deploying code from GitHub:**

1. Go to **Deployment Center**
2. Choose GitHub → select org/repo
3. Follow prompts (deployment takes a few minutes)
4. Visit the app's URL to confirm deployment

⚠️ Azure free account allows only **1 Linux App Service of size F1** — delete after each exercise before creating new ones.

---

## 🔑 Key Terms Glossary

| Term                               | Definition                                                                                                                                 |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **Subscription**                   | Billing/management boundary; multiple can exist per Azure account                                                                          |
| **Resource Group**                 | Organizes related resources (VMs, App Services, storage) for easier management                                                             |
| **Region**                         | Physical location of Azure data centers; closer region to end-user = lower latency                                                         |
| **ARM Templates**                  | Azure Resource Manager templates — repeatedly spin up the same set of resources                                                            |
| **Virtual Machines**               | IaaS — full OS access/control; Windows or Linux; more maintenance required                                                                 |
| **App Service**                    | PaaS — HTTP-based hosting for web apps/APIs/mobile backends; multi-language, continuous deployment; capped at ~14GB/4 vCPU on highest tier |
| **App Service Plan**               | Defines region, VM instance count/size, and pricing tier for an App Service (App may share underlying VM with other apps)                  |
| **Azure Batch**                    | Large-scale, high-performance compute beyond App Service capability                                                                        |
| **Azure Functions**                | Serverless, event-driven, compute-on-demand platform                                                                                       |
| **Container Instances**            | Deploy serverless Docker containers (no orchestration, unlike AKS)                                                                         |
| **Service Fabric**                 | Microsoft's distributed systems platform (Kubernetes-like)                                                                                 |
| **Azure Kubernetes Service (AKS)** | Microsoft's managed Kubernetes platform                                                                                                    |

---

## 🔑 Quick MCQ Traps — Azure Compute Services

- Hierarchy: **Account → Subscription → Resource Group → Resources**.
- Dev/test region choice → near yourself; production region choice → near end-users.
- VM = **IaaS**, full OS control, more expensive/maintenance; App Service = **PaaS**, no OS control, always billed even when idle.
- App Service hard limits: **~14 GB memory, 4 vCPU cores** max per instance (highest tier).
- Correct VM deployment order: **Resource Group → Create VM → Connect (SSH) → Install dependencies → Run app**.
- App running fine inside VM but unreachable externally → check that the **inbound port** (e.g. 80) is open.
- **Git** = local version control tool; **GitHub** = cloud platform hosting Git repos (owned by Microsoft).
- Azure free tier limits: **750 hrs** free VM hosting; only **1 Linux App Service (F1 free tier)** allowed.
- App Service names must be **globally unique across all of Azure**.
- `az group delete -n <name>` deletes an entire resource group (and everything inside it) in one command.
