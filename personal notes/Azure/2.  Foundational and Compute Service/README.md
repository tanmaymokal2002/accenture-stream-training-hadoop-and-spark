# Module 2: Foundational & Compute Services

## 🎯 Learning Objectives

In this module, you'll learn about the core services that form the foundation of cloud computing:

- Servers in the Cloud
- Security for Cloud Servers
- Compute Power in the Cloud

---

# Video 1: Introduction

### 💡 Explanation

This lesson introduces the core building blocks of cloud computing. It explains how cloud providers offer servers, computing power, and secure networking without requiring businesses to purchase and maintain physical hardware.

### 🔑 Key Points

- Understand cloud servers.
- Learn how cloud networking improves security.
- Explore different ways cloud providers offer compute power.

### ⭐ Remember

**Everything you build in the cloud ultimately runs on compute resources.**

---

# Video 2: Key Terminology

## Data

### 💡 Explanation

Data is any digital information such as documents, images, videos, application logs, or customer records. In cloud computing, cloud providers store and manage this data using specialized storage services.

### 📌 Real-World Example

Customer orders placed on Amazon are stored as data in cloud databases and storage services.

### ⭐ Remember

**Applications run on compute, but they exist because of data.**

---

## File Store (File System)

### 💡 Explanation

A File Store is used to store and retrieve files such as images, videos, PDFs, backups, and documents. It is optimized for storing large files efficiently.

### 📌 Real-World Example

A social media application stores users' uploaded profile pictures and videos in Azure Blob Storage instead of a database.

### ⭐ Remember

**Files → File Storage**

---

## Database

### 💡 Explanation

A Database stores structured data in an organized format, making it easy to search, update, and retrieve information quickly.

### 📌 Real-World Example

An e-commerce website stores customer information, product details, and orders in Azure SQL Database.

### ⭐ Remember

**Structured data → Database**

---

## Compute

### 💡 Explanation

Compute refers to the processing power (CPU and memory) required to run applications, execute code, and perform calculations.

### 📌 Real-World Example

When you search for a product on Amazon, compute resources process your request and return the results within seconds.

### ⭐ Remember

**No application can run without compute power.**

---

## Server

### 💡 Explanation

A Server is a computer that provides services, applications, or data to other computers (clients) over a network.
In cloud computing, the cloud provider (Azure, AWS, GCP) owns and manages the physical servers in its data centers.

For example:

Microsoft buys powerful physical machines.
These machines are installed in Azure data centers around the world.
You never see or touch them.

### 📌 Real-World Example

When you open Netflix, your request is processed by cloud servers that stream the selected movie to your device.

### ⭐ Remember

**Servers provide services to users or applications.**

---

## Instance

### 💡 Explanation

An Instance is a running virtual or physical server created by a cloud provider.Think of it as a slice of a powerful physical server(created by cloud provider we saw above)

### 📌 Real-World Example

A company launches three Azure Virtual Machine instances to run its web application across different regions.

### ⭐ Remember

**Instance = Running server in the cloud.**

---

## Cluster

### 💡 Explanation

A Cluster is a group of multiple instances working together to improve performance, scalability, and availability.

### 📌 Real-World Example

A Spark Cluster consists of one master node and multiple worker nodes processing large datasets in parallel.

### ⭐ Remember

**Cluster = Multiple instances working together.**

---

## Auto Scaling Group

### 💡 Explanation

An Auto Scaling Group automatically adds or removes instances based on application traffic or workload.

### 📌 Real-World Example

During an IPL ticket sale, Azure automatically launches additional Virtual Machines to handle increased traffic and removes them once demand decreases.

### ⭐ Remember

**Auto Scaling automatically adjusts resources based on demand.**

---

# Video 3: Servers in the Cloud

### 💡 Explanation

Cloud providers allow businesses to create virtual servers within minutes instead of purchasing and installing physical hardware. These servers can easily be upgraded or downgraded as business requirements change.

### 📌 Real-World Example

A startup launches its application on Azure Virtual Machines. As the number of users grows, it upgrades the VM from 2 CPUs to 8 CPUs without replacing any physical hardware.

### 🔑 Key Points

- Servers can be created within minutes.
- Easily increase CPU, RAM, or storage.
- No need to purchase physical servers.
- Pay only for the resources you use.

### ⭐ Remember

**Cloud servers provide flexibility without owning hardware.**

---

# Video 4: Security

### 💡 Explanation

Cloud providers allow organizations to secure their servers using virtual networks, firewalls, private networks, and access controls. Only authorized users and applications can access cloud resources.

### 📌 Real-World Example

A bank hosts its application on Azure. The database is placed inside a private subnet, making it inaccessible from the Internet. Only the application's backend servers inside the virtual network can communicate with the database.

### 🔑 Key Points

- Secure resources using Virtual Networks (VNets).
- Use Public and Private Subnets.
- Restrict access using security rules.
- Improve application security by isolating sensitive resources.

### ⭐ Remember

**Not every server should be accessible from the Internet.**

---

# Video 5: Compute Power in the Cloud

### 💡 Explanation

Cloud providers offer different ways to run applications without worrying about infrastructure. Compute resources can automatically scale, execute code only when needed, and charge based on actual usage.

---

## Serverless Computing

### 💡 Explanation

Serverless allows developers to run code without managing servers. The cloud provider automatically provisions, scales, and maintains the infrastructure.

### 📌 Real-World Example

When a customer uploads a document to Azure Blob Storage, an Azure Function automatically extracts text from the file and stores the processed data in a database.

---

## Continuous Scaling

### 💡 Explanation

Cloud applications can automatically increase or decrease compute resources based on workload.

### 📌 Real-World Example

Microsoft Teams automatically scales its backend services during large online events where thousands of participants join simultaneously.

---

## Event-Driven Computing

### 💡 Explanation

Applications can automatically execute code whenever a specific event occurs.

### 📌 Real-World Example

Whenever a customer completes an online payment, an Azure Function automatically generates an invoice and sends a confirmation email.

---

## Pay Only When Code Runs

### 💡 Explanation

For serverless services, you are billed only while your code is executing.

### 📌 Real-World Example

If an Azure Function runs for 500 milliseconds to process an uploaded image, billing is only for that execution time—not for an idle server running 24×7.

### ⭐ Remember

**Serverless = No server management + Automatic scaling + Pay only for execution time.**

---

# Module Summary

### ✅ What You Learned

- Basic cloud terminology
- Servers in the cloud
- Cloud networking security
- Compute power
- Serverless computing
- Clusters and Auto Scaling
- Why cloud servers are more flexible than traditional servers

### ⭐ Final Takeaway

**Cloud computing gives organizations flexible, secure, and scalable compute resources without requiring them to own or maintain physical infrastructure.**
