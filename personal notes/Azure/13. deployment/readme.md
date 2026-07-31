# Lesson 13: Containers & Kubernetes Deployment — Quick Revision Notes

Topics: Containers vs VMs, Container Runtimes, Kubernetes Basics, Pods & Services, Cluster Architecture, Control Plane Components.

---

## 1. Containers — Core Concept

- **Problem solved**: environment consistency when moving an app from dev → production.
- Two ways to bundle an app + its environment/dependencies: **Containers** and **Virtual Machines (VMs)**.
- Both allow apps to run with minimal dev/prod differences and support **horizontal scaling**.

### Container Definition

- Based on **OS-level virtualization** — runs multiple **isolated processes** in parallel.
- A container bundles:
  1. **Application code**
  2. **Dependencies** (libraries, utilities, config files)
  3. **Runtime environment**
- Each container = independent, portable unit — moves easily across environments.

### Containers vs VMs (key distinction)

|                     | Containers                   | VMs                             |
| ------------------- | ---------------------------- | ------------------------------- |
| **Kernel**          | **Share the host OS kernel** | Each VM has its **own full OS** |
| **Overhead**        | **Lower** system overhead    | Higher overhead                 |
| **Isolation level** | Process-level                | Full OS-level                   |

⚠️ MCQ trap: Containers running on the same machine **share the same low-level OS kernel** — this is why they're lighter than VMs.

### Container Runtimes/Engines (build & run containers)

| Runtime        | Note                                                   |
| -------------- | ------------------------------------------------------ |
| **Docker**     | Standardized packaging format — most well-known        |
| **CRI-O**      | Lightweight runtime, built specifically for Kubernetes |
| **containerd** | Focus: simplicity, robustness, portability             |
| **OpenVZ**     | Open-source, Linux container-based virtualization      |
| **Rkt**        | App container engine for cloud-native production       |
| **LXC / LXD**  | Distro/vendor-neutral Linux container tech             |

⚠️ MCQ trap: **Kubernetes does NOT build/run containers itself** — it **orchestrates/manages** them. The actual building/running is done by a **container runtime** (containerd, CRI-O, Docker Engine, etc.).

---

## 2. Kubernetes (K8s) — Overview

- **Kubernetes** = container **orchestration** system, developed by **Google**, open-sourced in **2014**.
- Automates deployment, scaling, and management of containerized apps.

### Key Benefits

- **Horizontal scaling** (scale container instances up/down)
- **Load balancing** + health checks
- **Inter-container networking**
- High availability architecture
- Auto-scaling
- Rich ecosystem
- Service discovery
- Container health management
- Secrets & configuration management

⚠️ MCQ trap: The **trade-off** of all these features = **high complexity / steep learning curve**.

---

## 3. Core Kubernetes Terminology (very testable — matching-style MCQs)

| Term                       | Definition                                                                                       |
| -------------------------- | ------------------------------------------------------------------------------------------------ |
| **Kubernetes**             | Container orchestration system; automates deployment & scaling                                   |
| **Pod**                    | Abstraction of **multiple containers**; smallest deployable unit; **ephemeral** (not persistent) |
| **Service**                | Abstraction of a **set of pods**; provides a stable network interface to reach them              |
| **Cluster**                | A group of machines running Kubernetes                                                           |
| **Master (Control Plane)** | System that controls the cluster — includes API, scheduler, management daemon                    |
| **Node**                   | A machine (VM, physical, or mix) in the cluster; runs the pods                                   |
| **Horizontal Scaling**     | Handling more traffic by adding replicas, splitting traffic across them                          |
| **Load Balancing**         | Distributing traffic across multiple endpoints                                                   |
| **Replica**                | A redundant copy of a resource (backup/load balancing)                                           |
| **Consumer**               | External entity (user/program) interfacing with the app                                          |

⚠️ MCQ trap — classic matching pattern:

- "Abstraction of pods + interface to interact with them" → **Service**
- "Abstraction of multiple containers" → **Pod**
- "Abstraction of an app + its dependencies" → **Container**

⚠️ MCQ trap: **Why wrap pods in a Service instead of exposing pods directly?** → Pods are **ephemeral** (can be destroyed/recreated by the master during scaling, with changing IPs) — a Service gives a **stable, persistent way** to communicate with the underlying (changing) set of pods.

⚠️ MCQ trap: Which K8s object gives a **persistent way to communicate** with a set of Pods? → **Service** (not Pod, not Volume).

---

## 4. Pods — Details

- **Smallest unit** in a Kubernetes cluster.
- A pod = **one or more containers** + shared storage + a **unique IP address**.
- Containers **within the same pod** share:
  - Namespaces
  - Filesystem volumes
- Pods are **NOT persistent** — master can bring them up/down during scaling.
- **Volumes** = attached to pods for **persistent storage** (since pod itself is ephemeral).

---

## 5. Kubernetes Cluster Architecture

- **Cluster** = **Nodes** (run containerized apps) + **Master** (manages nodes).
- Each **Node** needs a **container runtime** (e.g., Docker) installed.
- A node can host **multiple Pods**.
- **Pods are replicated across multiple nodes** → mitigates single point of failure → **high availability**.
- Master provides an **abstraction layer** for external clients/apps.

### High-Level Kubernetes Workflow (5 steps)

1. Create a Kubernetes cluster
2. Deploy an application into the cluster
3. Expose application ports
4. Scale the application
5. Update the application

### Setting Up a Cluster — Two Methods

| Method                      | Notes                                                                                    |
| --------------------------- | ---------------------------------------------------------------------------------------- |
| **Local cluster**           | e.g., via Docker Desktop (enable Kubernetes) — out of scope for deep dive in this course |
| **Cloud cluster (managed)** | Provider-managed K8s service                                                             |

### Managed Kubernetes by Cloud Provider

| Provider            | Service                                                  |
| ------------------- | -------------------------------------------------------- |
| **AWS**             | Amazon EKS                                               |
| **Google Cloud**    | GKE (Google Kubernetes Engine)                           |
| **Microsoft Azure** | **AKS (Azure Kubernetes Service)** ← used in this course |

---

## 6. Control Plane (Master) Components

The **Master = Control Plane**, manages worker nodes + pods (e.g., deciding when to start a new pod, scheduling).

| Component                    | Role                                                                                                                                                      |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **kube-apiserver**           | **Front-end/core** of control plane; exposes the Kubernetes **HTTP API**; allows querying/manipulating API objects (Pods, Namespaces, ConfigMaps, Events) |
| **etcd**                     | **Key-value store** for all cluster data                                                                                                                  |
| **kube-scheduler**           | Monitors pods; **assigns a worker node** to newly created pods                                                                                            |
| **kube-controller-manager**  | Abstract layer running **controller processes**                                                                                                           |
| **cloud-controller-manager** | Embeds cloud-specific logic; links cluster to the **cloud provider's API**                                                                                |

⚠️ MCQ trap: **kube-apiserver = the front door** to the cluster — everything (users, cluster parts, external tools) talks to Kubernetes through it.

---

## 7. Worker Node Components

| Component             | Role                                                                                      |
| --------------------- | ----------------------------------------------------------------------------------------- |
| **kubelet**           | Agent on each node; ensures containers are running correctly in their Pod                 |
| **kube-proxy**        | Maintains **network rules** on nodes; enables communication to Pods (internal & external) |
| **Container runtime** | Actually runs the containers (Docker, containerd, CRI-O, etc.)                            |

⚠️ MCQ trap — Master components vs Node components:

- **Master/Control Plane**: kube-apiserver, etcd, kube-scheduler, kube-controller-manager, cloud-controller-manager
- **Worker Node**: kubelet, kube-proxy, container runtime

---

## 🔑 Key Terms Glossary

| Term                     | Definition                                                                                         |
| ------------------------ | -------------------------------------------------------------------------------------------------- |
| **Container**            | Isolated process bundling app code + dependencies + runtime; shares host OS kernel                 |
| **VM**                   | Full virtualized machine with its own OS — heavier than a container                                |
| **Kubernetes (K8s)**     | Container orchestration system (Google, open-sourced 2014)                                         |
| **Pod**                  | Smallest K8s unit; group of containers sharing namespace/filesystem; ephemeral                     |
| **Service**              | Stable abstraction/interface to reach a set of pods                                                |
| **Cluster**              | Group of machines running Kubernetes                                                               |
| **Node**                 | A machine in the cluster (runs pods + container runtime)                                           |
| **Master/Control Plane** | Manages the cluster: kube-apiserver, etcd, scheduler, controller-manager, cloud-controller-manager |
| **kube-apiserver**       | Front-end API exposing Kubernetes API                                                              |
| **etcd**                 | Key-value store for cluster state                                                                  |
| **kube-scheduler**       | Assigns nodes to new pods                                                                          |
| **kubelet**              | Node agent ensuring containers run correctly                                                       |
| **kube-proxy**           | Manages network rules on nodes                                                                     |
| **AKS**                  | Azure's managed Kubernetes service                                                                 |
| **Horizontal Scaling**   | Adding replicas to handle more traffic                                                             |
| **Replica**              | Redundant copy of a resource                                                                       |

---

## 🔑 Quick MCQ Traps — Containers & Kubernetes

- Containers **share the host OS kernel** → lower overhead than VMs (which have their own OS each).
- **Kubernetes orchestrates**, it does **not** build/run containers — that's the job of a **container runtime** (Docker, containerd, CRI-O).
- **Pods = ephemeral**; **Services = stable/persistent** interface to reach pods — this is _why_ we wrap pods in services.
- Matching: Service = pods abstraction; Pod = containers abstraction; Container = app+dependencies abstraction.
- **kube-apiserver** = front-end/core of control plane (the entry point for all cluster communication).
- Master components: **kube-apiserver, etcd, kube-scheduler, kube-controller-manager, cloud-controller-manager**.
- Node components: **kubelet, kube-proxy, container runtime**.
- Azure's managed Kubernetes offering = **AKS**.
- Pods replicated across multiple nodes → avoids single point of failure → high availability.
