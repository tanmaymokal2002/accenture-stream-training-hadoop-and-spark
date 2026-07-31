# Hadoop & HDFS — Study Notes

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

_More notes to be added._
