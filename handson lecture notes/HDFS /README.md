# Hadoop Study Notes — Complete Reference & Command Guide

_Updated 01 Aug 2026 — includes hands-on session notes, real errors hit, and fixes_

---

## How to use this document

Each section starts with **why you'd do this** (the use case), then walks through the actual commands **one at a time**, explaining every word in the command the first time it appears. If a command repeats later, it is used without re-explaining — refer back to its first appearance if you forget.

---

## 1. Hadoop Framework — Overview

**Use case:** Before touching a terminal, you need the mental model — what problem does Hadoop solve? Hadoop lets you store huge amounts of data across many ordinary machines (HDFS) and process it in parallel (MapReduce/YARN) instead of needing one giant expensive server.

Hadoop follows a **master–slave architecture**.

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

**Use case:** You need to know _what's actually running_ on your machine before you can debug anything — when something breaks later, you'll be checking whether these specific processes are alive.

- **Master–Slave** design:
  - **Master → Namenode** — manages metadata, tracks where data lives
  - **Slave → Datanode** — actually stores the data
  - **SecondaryNamenode** — NOT a backup Namenode; it periodically merges/backs up the **Fsimage** and **editlogs**
    - **Fsimage** — snapshot of all commits since day one of HDFS
    - **editlogs** — log of commits over a recent period/size, later merged into Fsimage

**Cluster examples:**

- 4-node cluster → 1 Namenode (master) + 3 Datanodes (slaves)
- 1-node (pseudo-distributed) cluster → master and slave processes run on the same machine (this is what our training VM is)

**Key settings:**

- **Replication Factor** — default 3 (how many copies of each block are stored)
- **Block Size** — Hadoop 1.x = 64MB, Hadoop 2.x and above = 128MB

### Starting the cluster

```bash
start-dfs.sh
```

- `start-dfs.sh` = **one single filename**, no space between "start" and "dfs.sh" — it's a shell script bundled with Hadoop that starts the HDFS daemons (Namenode, Datanode, SecondaryNamenode) over SSH, even on a single machine.
- ⚠️ **Common typo:** typing `start dfs.sh` (with a space) makes the shell think you're running a command called `start` with an argument `dfs.sh` — it will fail with "Command 'start' not found."

```bash
start-yarn.sh
```

- Same naming pattern — one script that starts the YARN daemons: **ResourceManager** (decides how cluster resources are allocated) and **NodeManager** (manages resources on each individual node).

```bash
jps
```

- `jps` = **J**ava **P**rocess **S**tatus — lists all currently running Java processes on the machine, so you can verify the daemons above actually started.
- **Example output (healthy cluster):**
  ```
  2817 NodeManager
  1651 NameNode
  1932 SecondaryNameNode
  1773 DataNode
  3150 Jps
  2686 ResourceManager
  ```
  All 5 real daemons + the `jps` command itself (listed as `Jps`) should appear. If any are missing, that service failed to start.

### ⚠️ Real issue we hit: SSH host key warning

After a VM reset/reimage, `start-dfs.sh` may fail with a big warning block saying **"REMOTE HOST IDENTIFICATION HAS CHANGED"** and refuse to connect. This is SSH protecting you from a potential man-in-the-middle attack — but on a personal training VM where the underlying machine was simply rebuilt, it's just a stale saved key, not a real attack.

**Fix (the warning itself tells you the exact command):**

```bash
ssh-keygen -f "/home/clouduser/.ssh/known_hosts" -R "localhost"
```

- `ssh-keygen` = the tool for managing SSH keys
- `-f "<path>"` = **f**ile — tells it which known_hosts file to operate on
- `-R "localhost"` = **R**emove — deletes the old, mismatched entry for the host named `localhost`

**Example output:**

```
# Host localhost found: line 1
# Host localhost found: line 2
# Host localhost found: line 3
/home/clouduser/.ssh/known_hosts updated.
Original contents retained as /home/clouduser/.ssh/known_hosts.old
```

After this, re-run `start-dfs.sh` — it will ask "Are you sure you want to continue connecting (yes/no)?" the first time; type `yes`.

### ⚠️ Real issue we hit: daemons stop after a break/VM restart

Hadoop daemons do **not** survive a VM reboot or long idle period automatically. If you come back after a break and get:

```
stat: Call From ... to localhost:9000 failed on connection exception: java.net.ConnectException: Connection refused
```

this means the Namenode isn't running. **Your data in HDFS is NOT lost** — HDFS data lives on disk, not in memory. You only need to restart the daemons, not redo any file uploads:

```bash
start-dfs.sh
start-yarn.sh
jps
```

### If you hit "safe mode" errors

```bash
hdfs dfsadmin -safemode leave
```

- `dfsadmin` = a subcommand for cluster **admin**istration tasks (different from `dfs`, which is for file operations)
- `-safemode leave` = forces HDFS out of its read-only startup protection mode

---

## 3. File System Paths — LFS vs HDFS

**Use case:** You are always working with _two separate filesystems_ at once — your local Linux disk (LFS) and Hadoop's distributed filesystem (HDFS). A file existing in one does NOT mean it exists in the other. This is the single most common source of confusion for beginners.

|                             | Home Directory    |
| --------------------------- | ----------------- |
| **LFS** (Local File System) | `/home/clouduser` |
| **HDFS**                    | `/user/clouduser` |

```bash
pwd
```

- `pwd` = **p**rint **w**orking **d**irectory — shows which folder you're currently sitting in, on the **local** filesystem only (this command has no HDFS equivalent needed, since `hdfs dfs -ls` with no path always shows your HDFS home directory by default).

```bash
cd ~
```

- `cd` = **c**hange **d**irectory
- `~` = shorthand symbol meaning "my home directory" (`/home/clouduser`)

---

## 4. HDFS Permissions (chmod)

**Use case:** Just like Linux, you may need to restrict or open up who can read/write/execute a file or folder in HDFS — e.g., locking a folder so only you can touch it.

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
hdfs dfs -chmod -R 740 demodir
```

- `chmod` = **ch**ange **mod**e — sets permission bits on a file/folder
- `-R` = **R**ecursive — applies the change to the folder AND everything inside it
- `740` = the numeric permission code (see table above)
- `demodir` = the target folder

---

## 5. HDFS Shell Commands — Full Walkthrough (with real example outputs)

**Use case:** This is the bread-and-butter toolkit — every single HDFS operation (upload, download, inspect, delete, move) goes through this same command family. Once you know the pattern, every new command is just a new flag.

### Understanding the command structure (explained once, applies to ALL commands below)

```bash
hdfs dfs -ls
```

- `hdfs` = the main Hadoop executable you're calling
- `dfs` = a **subcommand** telling `hdfs` "I want to do a **f**ile**s**ystem operation" — this is different from other subcommands like `hdfs namenode`, `hdfs dfsadmin`, which manage daemons/clusters instead of files. `dfs` is _always_ required for file operations — it is not optional or a typo.
- `-ls` = the actual **action** — **l**i**s**t contents (same idea as Linux `ls`)

**Every command below follows this exact pattern:** `hdfs dfs -<action> <path>`

### a) Setup & basic navigation

```bash
hdfs dfs -ls
```

Lists contents of your **HDFS home directory** (`/user/clouduser`) — NOT your local Linux files.

**Example output:**

```
Found 1 items
drwxr-xr-x   - clouduser supergroup          0 2025-11-30 08:43 wordcount
```

```bash
hdfs dfs -mkdir HadoopDir
```

- `mkdir` = **m**a**k**e **dir**ectory — creates a new folder
- ⚠️ **No leading `/`** here means it's created inside your HDFS home directory → `/user/clouduser/HadoopDir`. Adding a leading `/` (e.g. `/Activity1`) instead creates it at the HDFS **root level**, visible to `hdfs dfs -ls /` instead of the plain `hdfs dfs -ls`.

### b) Unzip and stage a local dataset

```bash
unzip dataset.zip
```

- `unzip` = a standard Linux utility (not Hadoop-specific) that extracts `.zip` archives. Must be run from the folder that actually contains `dataset.zip` — check first with `ls`.

```bash
cp dept.csv /home/clouduser
```

- `cp` = **c**o**p**y — a plain Linux command, copies a file from one local folder to another (this step has nothing to do with HDFS yet — we're just staging the file locally before uploading it).

### c) Move a file from LFS → HDFS

```bash
hdfs dfs -put dept.csv HadoopDir
```

- `put` = uploads a file **from your local disk (LFS) into HDFS**.

⚠️ **Critical gotcha we hit for real:** If the target folder (`HadoopDir` here) does **not already exist** in HDFS, `-put` does NOT error out — instead it silently creates a **file** (not a folder!) with that exact name, containing your uploaded data. This is a trap.

**What it looked like when this went wrong:**

```
$ hdfs dfs -ls HadoopDir
-rw-r--r--   1 clouduser supergroup        108 2026-07-31 06:58 HadoopDir
```

Notice the listing starts with `-` (regular file) instead of `d` (directory) — this means `HadoopDir` itself became the file, not a folder containing your CSV.

**The fix — always run `mkdir` FIRST if you're not sure the folder exists:**

```bash
hdfs dfs -rm /user/clouduser/HadoopDir      # delete the broken file
hdfs dfs -mkdir HadoopDir                    # create it properly as a directory
hdfs dfs -put dept.csv HadoopDir             # now this uploads correctly
```

**Correct result after the fix:**

```
$ hdfs dfs -ls HadoopDir
Found 1 items
-rw-r--r--   1 clouduser supergroup        108 2026-07-31 07:05 HadoopDir/dept.csv
```

Now `dept.csv` is properly _inside_ the `HadoopDir` folder.

```bash
hdfs dfs -cat HadoopDir/dept.csv
```

- `cat` = **cat**enate — prints the file's contents directly to your screen (same as Linux `cat`, just reading from HDFS instead of local disk).

### d) Check file properties

```bash
hdfs dfs -stat "%n %o %r %F %u" HadoopDir/dept.csv
```

- `stat` = **stat**us — reports internal metadata about a file/folder
- The quoted string is a **format string** — each `%x` is a placeholder telling `stat` exactly which fields to print, and in what order:

| Placeholder | Meaning               |
| ----------- | --------------------- |
| `%n`        | File name             |
| `%o`        | Block size (in bytes) |
| `%r`        | Replication factor    |
| `%F`        | File type             |
| `%u`        | Owner                 |

**Example output:**

```
dept.csv 134217728 1 regular file clouduser
```

Breaking this down: block size `134217728` bytes = 128MB (the Hadoop 2.x+ default). Replication `1` — even though the _default_ cluster-wide setting is normally 3, a single-node cluster (only 1 Datanode) can never physically hold more than 1 copy, so new files land at replication 1 automatically.

### e) Disk usage / space

```bash
hdfs dfs -du HadoopDir
```

- `du` = **D**isk **U**sage — same concept as Linux's `du`, but reporting HDFS storage instead of local disk.

**Example output:**

```
108  108  HadoopDir/dept.csv
```

Two numbers are shown: **(1) raw file size** and **(2) size × replication factor**. Since replication here is 1, both numbers match. If a file had replication factor 2, the second number would be double the first (see section 6 for a real example of this).

```bash
hdfs dfs -du -h HadoopDir
```

- `-h` = **h**uman-readable — converts raw bytes into KB/MB/GB automatically instead of showing a long byte count. (On very small files like ours, the number may look the same since there's nothing large enough to convert.)

```bash
hdfs dfs -df -h
```

- `df` = **D**isk **F**ree — unlike `-du` (which reports a _specific file/folder's_ usage), `-df` reports the **entire HDFS cluster's** total capacity, used space, and remaining space.

### f) WORM principle

> **W**rite **O**nce **R**ead **M**any — you can't directly alter a file in HDFS. To "update" it, you overwrite: HDFS asks you to delete the old file and add the new one.

### g) Create & test files

```bash
hdfs dfs -touchz HadoopDir/empty.txt
```

- `touchz` = creates a brand-new, completely **empty** file (zero bytes) — similar to Linux `touch`, but the "z" specifically emphasizes zero-length. Useful for testing write access or as a placeholder.

```bash
hdfs dfs -test -e HadoopDir/empty.txt
echo $?
```

- `-test` = checks a condition about a file/path, without printing anything itself
- `-e` = checks whether the path **e**xists
- `echo $?` = prints the **exit status** of the previous command

**Example output:**

```
0
```

⚠️ **Important — this trips people up:** In shell scripting, exit codes are the OPPOSITE of what you might expect from normal programming:

| Exit Code        | Meaning                          |
| ---------------- | -------------------------------- |
| **0**            | Success / condition is **TRUE**  |
| **1** (non-zero) | Failure / condition is **FALSE** |

So `0` here correctly means **"yes, the file exists."** If you test a file that does NOT exist, you'll see `1` printed instead.

**Other `-test` flags (same `echo $?` pattern applies to each):**

| Flag | Checks                      |
| ---- | --------------------------- |
| `-e` | file/dir exists             |
| `-d` | is a directory              |
| `-f` | is a file                   |
| `-z` | file size is zero bytes     |
| `-s` | file size > 0 (has content) |

### h) Move / copy between HDFS locations, and to/from LFS

```bash
hdfs dfs -get HadoopDir/empty.txt
```

- `get` = downloads a file **from HDFS to your local disk (LFS)** — the exact reverse of `-put`. Also internally called `copyToLocal`. With no destination path given, it downloads into your current local folder.

```bash
hdfs dfs -mv HadoopDir/empty.txt demodir
```

- `mv` = **m**o**v**e a file from one HDFS location to another (source is removed after the move) — same idea as Linux `mv`. Just like `-put`, **make sure the destination folder already exists** (`mkdir` it first) to avoid the same "becomes a file instead of a folder" trap described in section (c).

```bash
hdfs dfs -cp demodir/empty.txt HadoopDir
```

- `cp` = **c**o**p**y a file from one HDFS location to another, but **keeps the original in place** too (unlike `mv`, which removes the source). After this, the file exists in both locations.

### i) Set replication factor on an existing file

```bash
hdfs dfs -setrep 2 demodir/dept.csv
```

- `setrep` = **set** **rep**lication — changes how many copies HDFS should try to maintain for an _already-existing_ file (as opposed to setting it at upload time).

**What actually happens on a single-node cluster:** HDFS records the **target** replication factor immediately in its metadata (so `-stat` will report `2`), even though only 1 Datanode exists to physically store data. The 2nd copy is technically "pending" and can never actually be created until more Datanodes join the cluster — this is called an "under-replicated block," and it's expected behavior on a training VM, not an error.

**Verifying with `-stat` (command explained in section 5d) after `-setrep 2`:**

```
dept.csv 134217728 2 regular file clouduser
```

Notice `%r` now shows `2` — confirming the target was recorded, even though physically only 1 copy exists on disk.

---

## 6. Hands-on Activity — "Activity1" Exercise (full command trail, with real gotchas)

**Use case:** This exercise strings together everything from section 5 into one realistic end-to-end task: load a real dataset (IPL cricket data), tune its replication and block size, back it up, and clean up — mirroring what you'd actually do with a new dataset on a real job.

```bash
# 1. Create the HDFS working directory (note the leading "/" — root level, not home dir)
hdfs dfs -mkdir /Activity1

# 2. Unzip the source dataset locally
cd /home/clouduser/dataset/sourcedata/ipldata
unzip ipldata.zip

# 3. Load both CSVs into HDFS
hdfs dfs -put /home/clouduser/dataset/sourcedata/ipldata/ipldata/deliveries.csv /Activity1/
hdfs dfs -put /home/clouduser/dataset/sourcedata/ipldata/ipldata/matches.csv /Activity1/
hdfs dfs -ls /Activity1
```

⚠️ **Real mistake made during this session:** a typo (`/Activity` instead of `/Activity1`) combined with the same "missing folder" trap from section 5c caused HDFS to create a stray **file** called `/Activity` at root level, holding the entire `deliveries.csv` content under the wrong name:

```
Found 5 items
-rw-r--r--   1 clouduser supergroup   18235327 2026-07-31 18:24 /Activity
drwxr-xr-x   - clouduser supergroup          0 2026-07-31 18:27 /Activity1
```

**Fix:**

```bash
hdfs dfs -rm /Activity                              # delete the wrongly-named file
hdfs dfs -put deliveries.csv /Activity1              # re-upload into the correct folder
hdfs dfs -put matches.csv /Activity1
```

**Confirmed correct result:**

```
Found 2 items
-rw-r--r--   1 clouduser supergroup   18235327 2026-07-31 18:27 /Activity1/deliveries.csv
-rw-r--r--   1 clouduser supergroup     140113 2026-07-31 18:27 /Activity1/matches.csv
```

```bash
# 4. Set replication factor = 2 for both files
hdfs dfs -setrep 2 /Activity1/deliveries.csv
hdfs dfs -setrep 2 /Activity1/matches.csv
```

```bash
# 5. Copy matches.csv back down to the LFS home directory
hdfs dfs -get /Activity1/matches.csv /home/clouduser/
ls -l /home/clouduser/matches.csv
```

- `ls -l` = **l**ong listing format — an ordinary Linux flag (lowercase **L**, not the number 1) that shows extra detail: permissions, owner, size, and date, instead of just the filename.

```bash
# 6. Check disk usage of Activity1 and get file/dir counts
hdfs dfs -du -h /Activity1
hdfs dfs -count /Activity1
```

- `count` = a new command not covered earlier — gives one summary line: number of directories, number of files, and total size (in bytes) under the given path. Useful as a quick sanity check before/after cleanup.

**Understanding the two-column `-du -h` output with replication applied:**

```
17.5 M   35.0 M   /Activity1
```

Column 1 (17.5M) is the raw data size. Column 2 (35.0M) is size × replication factor — and `35.0 / 17.5 ≈ 2`, which directly confirms the `-setrep 2` command from step 4 actually registered in HDFS's accounting (even though, as explained in section 5i, only 1 physical copy truly exists on a single-node cluster).

```bash
# 7. Create a backup directory
hdfs dfs -mkdir /Backup_act
```

```bash
# 8. Put matches.csv into Backup_act with a custom block size (2MB)
hdfs dfs -D dfs.blocksize=2097152 -put /home/clouduser/matches.csv /Backup_act/
hdfs dfs -ls /Backup_act
```

- `-D dfs.blocksize=2097152` = a one-time **override flag** that changes the block size used for just this single command, instead of the cluster's default (128MB = `134217728` bytes, as seen in section 5d). `2097152` bytes = exactly **2MB**. Since `matches.csv` (~140KB) is far smaller than even 2MB, it still fits in a single block here — but on a much larger file, a 2MB block size would cause it to be split into many more, smaller blocks compared to the default 128MB setting.

```bash
# 9. Check overall disk usage at root
hdfs dfs -du -h /
```

```bash
# 10. Remove Activity1 (and everything inside it)
hdfs dfs -rm -r /Activity1
hdfs dfs -ls /
```

- `-r` (lowercase, combined with `rm`) = **r**ecursive — required here because `/Activity1` is a directory containing files, not empty. A plain `-rm` without `-r` will refuse to delete a non-empty directory.

**Why each step matters (quick recap for study):**

- `-setrep` changes replication _after_ the file already exists (vs. setting it at write time).
- `-D dfs.blocksize=<bytes>` overrides the cluster's default block size **just for that one `put`**.
- `-count` gives you directory count, file count, and total size in one shot — useful before/after cleanup to sanity check.
- `-rm -r` deletes recursively — needed because `Activity1` is a directory with files inside, not an empty dir.

---

## 7. Suggested Study Order (to catch up efficiently)

1. **Concepts first:** Hadoop core components (HDFS, MR, YARN) → HDFS master-slave architecture → block size & replication factor. This is the foundation everything else builds on.
2. **Ecosystem map:** Skim what Hive, Pig, Sqoop, Zookeeper, Kafka, Flume each do — just enough to know _when_ you'd reach for each.
3. **Hands-on HDFS shell:** Actually run the commands in section 5 yourself in order — create dir → put a file → check stats → du/df → touchz + test flags → chmod → get/mv/cp. Typing them yourself will cement it faster than reading.
4. **Do the Activity1 exercise (section 6)** end-to-end on your VM — it combines almost everything from step 3 into one realistic task.
5. **Permissions (chmod) can be reviewed anytime** — it's a self-contained topic, good as a quick 10-minute review before a quiz.

---

## 8. Quick Troubleshooting Cheat Sheet (from real errors hit in this session)

| Symptom                                                                   | Cause                                                                                         | Fix                                                                |
| ------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `Command 'start' not found`                                               | Typed `start dfs.sh` with a space                                                             | Run `start-dfs.sh` as one word                                     |
| `REMOTE HOST IDENTIFICATION HAS CHANGED`                                  | Stale SSH key after VM reset                                                                  | `ssh-keygen -f "/home/clouduser/.ssh/known_hosts" -R "localhost"`  |
| `Connection refused` on any `hdfs dfs` command                            | Hadoop daemons not running (e.g. after a break/reboot)                                        | `start-dfs.sh` then `start-yarn.sh` then `jps` to confirm          |
| `-ls` on a folder shows a single file starting with `-rw-` instead of `d` | Uploaded into a folder that didn't exist yet — HDFS created a **file** with that name instead | `hdfs dfs -rm <path>` then `hdfs dfs -mkdir <path>` then re-upload |
| `put: '<path>': File exists`                                              | Trying to upload into a path that's already a file (see above)                                | Same fix as above                                                  |
| `unzip: cannot find or open X.zip`                                        | Wrong working directory                                                                       | `pwd` to check location; `cd` to where the zip actually is         |
