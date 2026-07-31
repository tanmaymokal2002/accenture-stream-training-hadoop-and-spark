# Lesson 4: Networking, Elasticity, Messaging & Containers — Quick Revision Notes

Topics: Networking in the cloud, Elasticity, Messaging & Queuing, Containers & Orchestration (Kubernetes).

---

## 1. Networking in the Cloud

- The **network is the foundation** of cloud infrastructure — reliably carries data globally with high availability.

**Cloud networking covers:**

- Network **architecture**
- Network **connectivity**
- **Application delivery**
- **Global performance**
- **Delivery** (of content to end users)

---

## 2. Elasticity in the Cloud

- **Elasticity** = the ability to **automatically scale up or down** (servers, databases, application resources) based on current load.
- Solves the "capacity guessing" problem — no more over-buying or under-buying infrastructure.

⚠️ MCQ trap: Elasticity ≠ just "scalability" in general — it specifically implies **automatic**, load-based scaling (both up AND down), not manual/one-directional scaling.

---

## 3. Messaging in the Cloud

- Used to **notify users** of application events (e.g., text messages, emails) via cloud-based services.
- Benefits: **lowered costs, increased storage, flexibility**.

### Queues

- A **queue** = data structure holding requests called **messages**.
- Processed in order: **FIFO** (First In, First Out).

**Messaging queues improve:**

- Performance
- Scalability
- User experience

⚠️ MCQ trap: Queues are **FIFO** — remember this is the opposite of a Stack (LIFO — Last In, First Out).

---

## 4. Containerization

**Container** = bundles an application with everything it needs to run into **one isolated unit**:

- Application **code**
- **Dependencies** (libraries, utilities, config files)
- **Runtime environment**

- Uses **OS-level virtualization** — runs multiple isolated processes in parallel.
- Containers **share the host's OS kernel** (unlike VMs) → lighter, faster to start, more resource-efficient.
- Runs consistently across environments: dev laptop → test server → production cloud.

### How Containers Work

- Run on a **container runtime** (e.g., Docker Engine, containerd).
- Use OS-level isolation to keep applications separated.
- Lightweight because they share the host's kernel (no full OS per container).

### Benefits of Containers

| Benefit         | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| **Portability** | Same container image runs on any system with the runtime     |
| **Efficiency**  | Uses fewer resources than VMs                                |
| **Speed**       | Starts in **seconds**                                        |
| **Consistency** | Solves "it works on my machine" problem                      |
| **Scalability** | Easily scaled via orchestration platforms (e.g., Kubernetes) |

### Containers vs Virtual Machines (VMs) — classic MCQ comparison

| Aspect            | Containers                                     | VMs                                        |
| ----------------- | ---------------------------------------------- | ------------------------------------------ |
| **Size**          | MBs (small)                                    | GBs (large)                                |
| **Speed**         | Seconds to start                               | Minutes to boot                            |
| **OS**            | Share host OS kernel                           | Full OS per instance                       |
| **Composability** | Programmatically built, defined as source code | Replicas of a conventional computer system |

⚠️ MCQ trap: Containers do **NOT** require a full OS per instance — that's the VM model. Containers share the **host kernel**.

### Common Container Runtimes/Engines

- **Docker** — standardized packaging format
- **CRI-O** — lightweight runtime for Kubernetes
- **OpenVZ** — open-source, Linux container-based virtualization
- **Containerd** — emphasis on simplicity, robustness, portability
- **Rkt** — app container engine for cloud-native environments
- **LXC and LXD** — distro/vendor-neutral Linux container tech

---

## 5. Container Orchestration & Kubernetes

**Why orchestration?** Running an app at production scale (thousands/millions of users) means managing **thousands of containers** — impossible manually. A **container orchestrator framework** creates, manages, and configures many containers across distributed servers while keeping them connected and reachable.

- Other orchestrators: Docker Swarm, Apache Mesos, CoreOS Fleet — but **Kubernetes** became the industry standard.
- Kubernetes is a **graduated CNCF project** (mature, widely adopted in production).

### Kubernetes Solves 6 Key Challenges:

| Feature               | What it means                                                                                      |
| --------------------- | -------------------------------------------------------------------------------------------------- |
| **Portability**       | Open-source, vendor-agnostic — runs on any infra (public/private/hybrid cloud)                     |
| **Scalability**       | Built-in **HPA** (Horizontal Pod Autoscaler) auto-determines needed replicas                       |
| **Resilience**        | Uses **ReplicaSet**, readiness/liveness probes → self-healing from failures                        |
| **Service Discovery** | Cluster-level **DNS** + routing/load balancing for reachability                                    |
| **Extensibility**     | Rich API, supports **CRDs** (Custom Resource Definitions)                                          |
| **Operational Cost**  | Smart scheduling + **cluster-autoscaler** → efficient resource use, scales cluster size to traffic |

---

## 6. Kubernetes Architecture

A **Kubernetes cluster** = collection of distributed physical/virtual servers called **nodes**.

- **Nodes** → 2 types: **Master nodes** (control plane) and **Worker nodes** (data plane).

### Control Plane (Master Nodes) — makes global cluster decisions

| Component                   | Role                                                                             |
| --------------------------- | -------------------------------------------------------------------------------- |
| **kube-apiserver**          | Nucleus of the cluster — exposes Kubernetes API, handles/triggers all operations |
| **kube-scheduler**          | Places new workloads on nodes with sufficient resources                          |
| **kube-controller-manager** | Handles controller processes — ensures desired config is propagated              |
| **etcd**                    | Key-value store — backs up & stores manifests for the entire cluster             |

### Data Plane (Worker Nodes) — hosts the actual workloads

| Component      | Role                                                                         |
| -------------- | ---------------------------------------------------------------------------- |
| **kubelet**    | Agent on every node — notifies kube-apiserver that node is part of cluster   |
| **kube-proxy** | Network proxy — ensures reachability/accessibility of workloads on that node |

⚠️ Big MCQ trap: **`kubelet` and `kube-proxy` run on ALL nodes** (both master AND worker) — not just worker nodes! They keep the kube-apiserver updated on node status and manage connectivity everywhere.

---

## 🔑 Key Terms Glossary

| Term            | Definition                                                                  |
| --------------- | --------------------------------------------------------------------------- |
| **CRD**         | Custom Resource Definition — extends Kubernetes API with new resource types |
| **Node**        | A physical or virtual server                                                |
| **Cluster**     | Collection of distributed nodes managing/hosting workloads                  |
| **Master node** | Control plane node — makes global, cluster-level decisions                  |
| **Worker node** | Data plane node — hosts actual workloads                                    |

---

## 🔑 Quick MCQ Traps — Cloud Networking/Elasticity/Messaging/Containers

- **Elasticity** = automatic scaling **up or down** based on load (not manual, not one-directional).
- **Queues** = **FIFO** (First In, First Out) — messages processed in order received.
- Containers **share the host OS kernel** — do NOT need a full OS per instance (that's VMs).
- Containers vs VMs: Containers = MBs & seconds to start; VMs = GBs & minutes to boot.
- **Kubernetes** = the dominant container orchestration framework (graduated CNCF project).
- Control plane components: **kube-apiserver, kube-scheduler, kube-controller-manager, etcd**.
- **kubelet** and **kube-proxy** run on **every node** — both master and worker, not just worker.
- **HPA** (Horizontal Pod Autoscaler) → handles scalability; **cluster-autoscaler** → handles cluster-size/operational cost; **ReplicaSet** + readiness/liveness probes → handle resilience/self-healing.
