# Lesson 8: Azure Storage — Quick Revision Notes

Topics: Storage Benefits & Options, Azure SQL Database, Blob Storage & Storage Accounts, Connecting an App to Storage, Mounting Blob Storage.

---

## 1. Benefits of Azure Storage

- **Automated backup and recovery**
- **Replication** across multiple data centers worldwide — protects against outages (e.g. hardware failure)
- **Data analytics** support
- **Data encryption** for security
- Supports **3 main data types**: relational, non-relational (NoSQL), and unstructured (e.g. images)
- **Scale up/out** when demand is high, scale back when low
- Eliminates cost of buying/installing/maintaining **on-premises hardware**

### Azure Storage Options (Overview)

- **Azure SQL Server / SQL Database** — relational data
- **Azure Blob Storage** — unstructured data (images, videos)
- **Azure CosmosDB** — non-relational/NoSQL
- **Disk Storage** — VM disks
- **Azure Data Lake Storage** — big data analytics
- **HPC Cache** — faster access for high-performance compute

**Blob storage has 3 access tiers:** Hot, Cool, Archive (covered in detail below).

---

## 2. Azure SQL Database

- Used for **structured, relational data**.
- Requires a **SQL Server** first — a single SQL Server can host **multiple SQL databases**.
- **No free tier** (unlike App Services' F1 tier) — but new free Azure accounts get 12 months of up to 250GB SQL storage. Lowest paid tier ~$5/month otherwise.

### Creating an Azure SQL Database — Portal Walkthrough

1. Find **"SQL databases"** service in Azure Portal → Click **"Create SQL Database"**
2. Select subscription & resource group
3. Enter a database name
4. Create a **new SQL Server** if you don't have one — set server name, admin username (⚠️ **cannot be "admin"**), and password (remember these!)
5. Set **location** to match your resource group
6. Keep SQL elastic pool = **"No"** (default)
7. Under **Compute+Storage** → "Configure Database" → switch to **"Basic"** tier (cheap/free for short-lived exercises)
8. Click **"Next: Networking"** → select **"Public Endpoint"** → set both Firewall rules to **"Yes"** (allows app to access the DB later)
9. **Review + Create** → **Create**, then wait for deployment

### Adding Data to the SQL Database

1. Open the deployed database → **"Query editor"** in left menu → log in with SQL Server credentials
2. Paste your SQL script (e.g. table creation query) into the query window → **Run**
3. Check the **"Tables"** folder to confirm the table was created correctly
4. Run `SELECT * FROM <table_name>` to verify the data

---

## 3. Blob Storage

**Blob** (Binary Large Object) = data type for storing **unstructured (binary) data** — e.g. images, videos.

- **Not ideal** for structured, frequently-queried data (e.g. user profile info) — higher latency than memory/local disk, no query-optimized indexing.
- Commonly used **alongside** a database — e.g. DB stores a **URL/reference** pointing to the blob (like a profile picture).

⚠️ MCQ trap: Blobs are for **unstructured** binary data; use SQL/relational storage for **structured, queryable** data.

### Azure Storage Accounts

- **Storage Account** = highest level of the storage hierarchy for data objects (blobs, files, queues, tables).
- Provides a **unique namespace** — every stored item has an address including your unique account name.
- Storage account names: **lowercase letters and numbers only**.
- **General-purpose v2** = recommended type for most scenarios (supports Blobs, Data Lake Gen2, Files, Disks, Queues, Tables).

### Blob Storage Hierarchy

```
Storage Account (e.g. "Udacity")
   └── Container(s) (e.g. "Pictures", "Videos")
         └── Blob(s) (the actual files)
```

- One storage account → multiple **containers** → each container → multiple **blobs**.

### Blob Storage Tiers (Lifecycle)

| Tier        | Use Case                                   | Latency                     | Cost    |
| ----------- | ------------------------------------------ | --------------------------- | ------- |
| **Hot**     | Frequently accessed data                   | Lowest                      | Highest |
| **Cool**    | Infrequently accessed; stored **≥30 days** | Higher                      | Lower   |
| **Archive** | Rarely accessed; stored **≥180 days**      | Very high (retrieval delay) | Lowest  |

- **Lifecycle management rules** can auto-transition blobs between tiers based on age/inactivity (e.g. Hot → Cool after 30 days unmodified → Archive after 90 days → delete after a year).
- Found under: Storage Account → **"Blob Service"** → **"Lifecycle Management"**.

⚠️ MCQ trap: Hot = frequent access + low latency + high cost; Archive = rare access + high latency + low cost. Remember the **inverse relationship** between access frequency and cost/latency.

### Creating a Storage Account & Container — Portal Walkthrough

_(Interface steps, since these resources are commonly created via the Azure Portal UI)_

1. Azure Portal homepage → **"Create a resource"** → search **"Storage account"** → Create
2. Select subscription & resource group; enter a **globally unique, lowercase** storage account name
3. Choose region (match your resource group's region where possible)
4. Performance: Standard; Redundancy: as needed (default is fine for exercises)
5. Redundancy/Account kind defaults to **General-purpose v2**; access tier defaults to **Hot**
6. Review + Create → Create, then wait for deployment
7. Once deployed, go into the storage account → **"Containers"** (under Data storage) → **"+ Container"** → name it (e.g. "images") → set **Public access level** as needed → Create
8. You can then **upload files/images** directly through the Portal UI into the container
9. To set up tier-transition rules: go to **"Lifecycle Management"** under **"Blob service"** and add a new rule

---

## 4. Connecting an App to Azure Storage

**Info needed from each service:**

**From SQL Server/Database:**

- SQL Server name (format: `<servername>.database.windows.net`)
- Admin username & password
- SQL Database name

**From Blob Storage:**

- Storage account name
- Storage account **access key**
- Container name

- Typically managed via a **config.py** file imported into the main app.

### Python: Using Azure Storage Blob Library

```bash
pip install azure-storage-blob   # remember to add to requirements.txt too
```

**Key class: `BlobServiceClient`** — 3 main methods:
| Method | Purpose |
|--------|---------|
| `get_blob_client()` | Creates a blob client using the filename as the blob's name |
| `upload_blob()` | Uploads the file to the blob container |
| `delete_blob()` | Deletes a blob from the container |

**Uploading a blob:**

```python
from azure.storage.blob import BlobServiceClient

blob_container = app.config['BLOB_CONTAINER']
storage_url = "https://{}.blob.core.windows.net/".format(app.config['BLOB_ACCOUNT'])
blob_service = BlobServiceClient(account_url=storage_url, credential=app.config['BLOB_STORAGE_KEY'])

blob_client = blob_service.get_blob_client(container=blob_container, blob=filename)
blob_client.upload_blob(file)
```

⚠️ MCQ trap: `filename` (string, e.g. `"test_image.png"`) ≠ `file` (the actual file object). The blob client is created **using the filename**, then the **file object** is uploaded into that named blob.

**Deleting a blob:**

```python
blob_client = blob_service.get_blob_client(container=blob_container, blob=filename)
blob_client.delete_blob()
```

- No file object needed to delete — just the filename via `get_blob_client()`.

### Endpoint → Resource Matching (common MCQ)

| Endpoint Suffix          | Resource            |
| ------------------------ | ------------------- |
| `.database.windows.net`  | **SQL Server**      |
| `.azurewebsites.net`     | **App Service**     |
| `.blob.core.windows.net` | **Storage Account** |

⚠️ MCQ scenario trap: If images **upload successfully with no errors**, appear correctly **in the container** (Portal), but show as **broken images** in the app — likely cause is the blob container's **public access level** isn't set correctly (e.g., container/blob access isn't public, so the browser can't directly load the image URL) — a permissions/access-level issue, not an upload issue.

---

## 5. Mounting Blob Storage (Conceptual Overview)

- **Mounting** = creating a "shortcut" so Azure Blob Storage appears as a **regular folder** on a computer or Web App — instead of using special API/CLI commands to access it.
- Once mounted, a Web App can interact with blob files using **normal file operations** ("open this file", "list files in this folder") — it doesn't need to know the files are actually stored remotely in the cloud.
- Useful for apps/programs that expect to work with **local file-system-style access** rather than cloud SDK calls.
- Different mounting methods exist depending on OS (Windows/Linux) and use case.

⚠️ Not required for this course's project, but relevant for production-grade apps / Azure Developer Certification.

---

## 🔑 Key Terms Glossary

| Term                   | Definition                                                                         |
| ---------------------- | ---------------------------------------------------------------------------------- |
| **Azure SQL Server**   | Required parent resource for SQL Databases; one server can host multiple databases |
| **Azure SQL Database** | Stores structured, relational data                                                 |
| **Blob**               | Binary Large Object — stores unstructured data (images, video, etc.)               |
| **Storage Account**    | Highest level of storage hierarchy for blobs, files, queues, tables                |
| **Container**          | Organizes blobs within a storage account (e.g. "images" vs "videos")               |
| **Blob Storage**       | Storage specifically for blob (unstructured) data                                  |
| **Table Storage**      | Structured, **non-relational** data                                                |
| **File Storage**       | File-sharing storage solutions                                                     |
| **Disk Storage**       | Disk-based storage (HDD/SSD)                                                       |
| **Data Lake Storage**  | Big data analytics storage                                                         |
| **Hot Storage**        | Frequently accessed data                                                           |
| **Cool Storage**       | Infrequently accessed; min. **30 days** retention                                  |
| **Archive Storage**    | Rarely accessed; min. **180 days** retention; high access latency                  |
| **HPC Cache**          | Cache for faster access to select data during high-performance compute             |
| **CosmosDB**           | Azure's non-relational DB service (supports MongoDB, Cassandra APIs, etc.)         |
| **Retention**          | How long a piece of data is stored (e.g. 30 days)                                  |

---

## 🔑 Quick MCQ Traps — Azure Storage

- Blob = **unstructured** data; SQL Database = **structured/relational** data; Table Storage = **structured but non-relational**.
- Azure SQL requires a **SQL Server** to be created first (parent resource); one server → many databases.
- SQL admin username **cannot be "admin"**.
- Storage account names: **lowercase letters + numbers only**, globally unique.
- Hierarchy: **Storage Account → Container(s) → Blob(s)**.
- Tier order by access frequency (high→low) & cost (high→low): **Hot → Cool → Archive**; latency increases in that same order.
- Cool = min **30 days** retention; Archive = min **180 days** retention.
- Endpoint matching: `.database.windows.net` = SQL Server; `.azurewebsites.net` = App Service; `.blob.core.windows.net` = Storage Account.
- In code: `filename` (string/name) ≠ `file` (actual file object) — `get_blob_client()` uses filename, `upload_blob()` takes the file.
- Broken images despite successful upload → check **public access level** of the container/blob, not the upload code.
- **Mounting** blob storage = makes cloud storage appear as a local folder, enabling normal file-system operations instead of SDK/CLI calls.
