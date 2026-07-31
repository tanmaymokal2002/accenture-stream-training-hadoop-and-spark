# Hadoop Study Notes — Updated 31 Jul 2026

_(Catch-up notes from 30 Jul class, compiled from WhatsApp group)_

---

## 1. Hadoop Framework — Overview

Hadoop follows a **master-slave architecture**.

### Core Components

| Component                                  | Role                                                                     |
| ------------------------------------------ | ------------------------------------------------------------------------ |
| **HDFS** (Hadoop Distributed File System)  | Storage layer — the "Data Lake"                                          |
| **MapReduce (MR)**                         | Processing layer — programming model for batch processing                |
| **YARN** (Yet Another Resource Negotiator) | Resource management layer — allocates/deallocates resources across nodes |

### Ecosystem Components (tools built on top of Hadoop)

| Tool          | Purpose                                                   |
| ------------- | --------------------------------------------------------- |
| **Hive**      | Distributed data warehouse, queried using HQL             |
| **Pig**       | ETL on all data types, using Pig Latin scripting language |
| **Sqoop**     | Migrates data between RDBMS and Hadoop (import/export)    |
| **Zookeeper** | Centralized coordination service                          |
| **Kafka**     | Message queue / event streaming                           |
| **Flume**     | Collects, aggregates, and transfers log/event data        |

---

## 2. HDFS Architecture

- **Master–Slave** design:
  - **Master → Namenode** — manages metadata, tracks where data lives
  - **Slave → Datanode** — actually stores the data
  - **SecondaryNamenode** — NOT a backup Namenode; it periodically merges/backs up the **Fsimage** and **editlogs**
    - **Fsimage** — snapshot of all commits since day one of HDFS
    - **editlogs** — log of commits over a recent period/size, later merged into Fsimage

**Cluster examples:**

- 4-node cluster → 1 Namenode (master) + 3 Datanodes (slaves)
- 1-node (pseudo-distributed) cluster → master and slave processes run on the same machine

**Key settings:**

- **Replication Factor** — default 3 (how many copies of each block are stored)
- **Block Size** — Hadoop 1.x = 64MB, Hadoop 2.x and above = 128MB

Start the cluster:

```bash
start-dfs.sh; start-yarn.sh; start-master.sh; start-workers.sh
jps          # verify Namenode, Datanode, SecondaryNamenode, ResourceManager, NodeManager are running
```

---

## 3. File System Paths — LFS vs HDFS

|                             | Home Directory    |
| --------------------------- | ----------------- |
| **LFS** (Local File System) | `/home/clouduser` |
| **HDFS**                    | `/user/clouduser` |

---

## 4. HDFS Permissions (chmod)

Same numeric scheme as Linux:

| Value   | Meaning |
| ------- | ------- |
| Read    | 4       |
| Write   | 2       |
| Execute | 1       |

Format: `rwxrwxrwx` → **Owner / Group / Others**

| Code  | Meaning                                                             |
| ----- | ------------------------------------------------------------------- |
| `777` | 4+2+1 — full access to everyone                                     |
| `700` | Only owner has access; group & others have none                     |
| `766` | `rwxrw-rw-` — owner full; group & others can read+write, no execute |
| `755` | `rwxr-xr-x` — owner full; group & others can read+execute only      |
| `740` | Owner full; group read-only; others no access                       |

```bash
hdfs dfs -chmod -R 740 demodir     # no access for group
hdfs dfs -chmod -R 777 demodir     # full access for all
```

---

## 5. HDFS Shell Commands — Walkthrough

### a) Setup & basic navigation

```bash
start-dfs.sh
hdfs dfsadmin -safemode leave        # only if you hit a "safe mode" error
jps
hdfs dfs -ls
hdfs dfs -mkdir HadoopDir
```

### b) Unzip and stage a local dataset

```bash
unzip dataset.zip
cd dataset/hive
cp dept.csv /home/clouduser
cd
ls
```

### c) Move a file from LFS → HDFS

```bash
hdfs dfs -put dept.csv HadoopDir
# or with full path:
hdfs dfs -put /home/clouduser/dataset/hive/dept.csv HadoopDir

hdfs dfs -ls HadoopDir           # confirm it copied
hdfs dfs -cat HadoopDir/dept.csv # view file content
```

### d) Check file properties

```bash
hdfs dfs -stat "%n %o %r %F %u" HadoopDir/dept.csv
```

- `%n` → File name
- `%o` → Block size
- `%r` → Replication factor
- `%F` → File type
- `%u` → Owner

### e) Disk usage / space

```bash
hdfs dfs -du HadoopDir       # size used by directory/files
hdfs dfs -du -h              # human-readable
hdfs dfs -df                 # overall filesystem stats
hdfs dfs -df -h              # human-readable
```

### f) WORM principle

> **W**rite **O**nce **R**ead **M**any — you can't directly alter a file in HDFS. To "update" it, you overwrite: HDFS asks you to delete the old file and add the new one.

### g) Create & test files

```bash
hdfs dfs -touchz HadoopDir/empty.txt     # create an empty file
hdfs dfs -ls HadoopDir

hdfs dfs -test -e HadoopDir/empty.txt    # check existence
echo $?                                  # 0 = true, 1 = false
```

Test flags:
| Flag | Checks |
|---|---|
| `-e` | file/dir exists |
| `-d` | is a directory |
| `-f` | is a file |
| `-z` | file size is zero bytes |
| `-s` | file size > 0 (has content) |

### h) Move / copy between HDFS locations, and to/from LFS

```bash
hdfs dfs -get HadoopDir/empty.txt              # HDFS -> LFS (copyToLocal)
ls                                              # confirm it landed locally

hdfs dfs -mv HadoopDir/empty.txt demodir        # move within HDFS
hdfs dfs -cp demodir/empty.txt HadoopDir        # copy within HDFS
```

### i) Set replication factor on an existing file

```bash
hdfs dfs -setrep 2 demodir/dept.csv
```

---

## 6. Hands-on Activity — "Activity1" Exercise (full command trail)

**Goal:** create a directory, load two CSVs, set replication, move a copy to LFS, check sizes, create a backup with a custom block size, then clean up.

```bash
# 1. Create the HDFS working directory
hdfs dfs -mkdir /Activity1

# 2. Unzip the source dataset locally
cd /home/clouduser/dataset/sourcedata/ipldata
unzip ipldata.zip

# 3. Load both CSVs into HDFS
hdfs dfs -put /home/clouduser/dataset/sourcedata/ipldata/ipldata/deliveries.csv /Activity1/
hdfs dfs -put /home/clouduser/dataset/sourcedata/ipldata/ipldata/matches.csv /Activity1/
hdfs dfs -ls /Activity1

# 4. Set replication factor = 2 for both files
hdfs dfs -setrep 2 /Activity1/deliveries.csv
hdfs dfs -setrep 2 /Activity1/matches.csv

# 5. Copy matches.csv back down to the LFS home directory
hdfs dfs -get /Activity1/matches.csv /home/clouduser/
ls -l /home/clouduser/matches.csv

# 6. Check disk usage of Activity1 and get file/dir counts
hdfs dfs -du -h /Activity1
hdfs dfs -count /Activity1

# 7. Create a backup directory
hdfs dfs -mkdir /Backup_act

# 8. Put matches.csv into Backup_act with a custom block size (2MB)
hdfs dfs -D dfs.blocksize=2097152 -put /home/clouduser/matches.csv /Backup_act/
hdfs dfs -ls /Backup_act

# 9. Check overall disk usage at root
hdfs dfs -du -h /

# 10. Remove Activity1 (and everything inside it)
hdfs dfs -rm -r /Activity1
hdfs dfs -ls /
```

**Why each step matters (quick recap for study):**

- `-setrep` changes replication _after_ the file already exists (vs. setting it at write time).
- `-D dfs.blocksize=<bytes>` overrides the cluster's default block size **just for that one `put`** — here, 2MB = 2097152 bytes, much smaller than Hadoop's default 128MB, so you'd see it split into many more blocks.
- `-count` gives you directory count, file count, and total size in one shot — useful before/after cleanup to sanity check.
- `-rm -r` deletes recursively — needed because `Activity1` is a directory with files inside, not an empty dir.

---

## 7. Apache Hive

### What is Hive?

- A **data warehouse tool** built for **analysis**
- A **distributed system**
- Query language: **HQL (Hive Query Language)** — SQL-like
- Under the hood: **MapReduce** does the processing, **HDFS** does the storing

### Why Hive?

- Works with **structured data**
- Gives you a familiar **query language**
- Supports **ETL** (clean, transform, aggregate) — good fit for data warehousing
- Supports **partitioning** and **bucketing** for performance
- Supports multiple file formats: **RC, ORC, Parquet, AVRO, Txt, CSV**, etc.

### Hive Architecture

| Component                | Role                                                                                        |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| **CLI**                  | Interactive layer where the user connects to Hive                                           |
| **Metastore**            | Stores all metadata — DBs, tables, partitions, views, bucketing, UDFs (backed by **MySQL**) |
| **HQL Process Engine**   | Converts HQL queries into MR jobs                                                           |
| **HQL Execution Engine** | Executes those MR jobs                                                                      |
| **HDFS**                 | Stores the final results                                                                    |

### Query Execution Workflow

```sql
SELECT Deptid, count(*) FROM Employee GROUP BY Deptid;
```

1. **CLI** — user runs the query
2. **Driver** — checks syntax
3. **Compiler** — starts converting HQL into an execution plan (semantic analysis); sends a metadata request
4. **Metastore** — responds with the requested metadata
5. **Compiler** — builds the logical plan, then the physical plan
6. **Execution Engine (MapReduce)**:
   - Resource Manager checks for available resources
   - → Node Manager
   - → Application Master
   - → Container
   - → Namenode
   - → Datanode
   - → back to Resource Manager
   - → back to Execution Engine
7. **Execution Engine → Driver → CLI → Result** displayed to the user

### Creating & Managing Databases/Tables

```sql
CREATE DATABASE demodb;
SHOW DATABASES;
-- creates a directory "demodb.db" under the Hive warehouse dir:
-- /user/hive/warehouse

USE demodb;
CREATE TABLE emp(empno INT, empname STRING);
SHOW TABLES;
```

There's also a distinction between:

- **Managed Tables** — Hive owns the data; dropping the table deletes the data too
- **External Tables** — Hive only manages metadata; dropping the table leaves the underlying data untouched

### Describing & Altering a Database

```sql
DESCRIBE DATABASE hadoopdb;
-- hadoopdb  <location: hdfs://localhost:9000/user/hive/warehouse/hadoopdb.db>  clouduser  USER

ALTER DATABASE hadoopdb SET DBPROPERTIES('created-by'='Dhivya', 'created-for'='Hadoop training');

DESCRIBE DATABASE EXTENDED hadoopdb;
-- now also shows: {created-by=Dhivya, created-for=Hadoop training}
```

`DESCRIBE DATABASE EXTENDED` shows custom properties set via `ALTER DATABASE ... SET DBPROPERTIES`, which plain `DESCRIBE DATABASE` does not.

---

## 8. Suggested Study Order (to catch up efficiently)

1. **Concepts first:** Hadoop core components (HDFS, MR, YARN) → HDFS master-slave architecture → block size & replication factor. This is the foundation everything else builds on.
2. **Ecosystem map:** Skim what Hive, Pig, Sqoop, Zookeeper, Kafka, Flume each do — just enough to know _when_ you'd reach for each.
3. **Hands-on HDFS shell:** Actually run the commands in section 5 yourself in order — create dir → put a file → check stats → du/df → touchz + test flags → chmod → get/mv/cp. Typing them yourself will cement it faster than reading.
4. **Do the Activity1 exercise (section 6)** end-to-end on your VM — it combines almost everything from step 3 into one realistic task.
5. **Then move to Hive:** why it exists → architecture → walk through the query execution flow with the sample `SELECT ... GROUP BY` query until you can explain each arrow from memory → practice `CREATE DATABASE` / `CREATE TABLE` / `DESCRIBE` / `ALTER` commands.
6. **Permissions (chmod) can be reviewed anytime** — it's a self-contained topic, good as a quick 10-minute review before a quiz.
