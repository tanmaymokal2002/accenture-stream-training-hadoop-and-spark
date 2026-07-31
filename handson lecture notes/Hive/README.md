# Apache Hive — Study Notes

## What is Hive?

- A **data warehouse tool** built for **analysis**
- A **distributed system**
- Query language: **HQL (Hive Query Language)** — SQL-like
- Under the hood: **MapReduce** does the processing, **HDFS** does the storing

## Why Hive?

- Works with **structured data**
- Gives you a familiar **query language**
- Supports **ETL** (clean, transform, aggregate) — good fit for data warehousing
- Supports **partitioning** and **bucketing** for performance
- Supports multiple file formats: **RC, ORC, Parquet, AVRO, Txt, CSV**, etc.

---

## Hive Architecture

| Component                | Role                                                                                        |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| **CLI**                  | Interactive layer where the user connects to Hive                                           |
| **Metastore**            | Stores all metadata — DBs, tables, partitions, views, bucketing, UDFs (backed by **MySQL**) |
| **HQL Process Engine**   | Converts HQL queries into MR jobs                                                           |
| **HQL Execution Engine** | Executes those MR jobs                                                                      |
| **HDFS**                 | Stores the final results                                                                    |

---

## Query Execution Workflow

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

---

## Creating & Managing Databases/Tables

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

---

## Describing & Altering a Database

```sql
DESCRIBE DATABASE hadoopdb;
-- hadoopdb  <location: hdfs://localhost:9000/user/hive/warehouse/hadoopdb.db>  clouduser  USER

ALTER DATABASE hadoopdb SET DBPROPERTIES('created-by'='Dhivya', 'created-for'='Hadoop training');

DESCRIBE DATABASE EXTENDED hadoopdb;
-- now also shows: {created-by=Dhivya, created-for=Hadoop training}
```

`DESCRIBE DATABASE EXTENDED` shows custom properties set via `ALTER DATABASE ... SET DBPROPERTIES`, which plain `DESCRIBE DATABASE` does not.

---

## Managed vs External Tables — In Detail

### Managed Table (default)

Hive **owns** the data. You either `CREATE` + `INSERT`/`LOAD` into it, and if you `DROP` the table, the underlying data in HDFS is deleted too.

```sql
CREATE TABLE emp(
    empno INT, empname STRING, age INT, gender STRING,
    Salary INT, Designation STRING, DeptID STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n';
```

### External Table

Hive only manages the **metadata** — it points at data that already lives somewhere in HDFS. `DROP TABLE` removes the table definition from Hive, but the actual data file is untouched.

```bash
# stage the file in HDFS first
hdfs dfs -mkdir emp_ext
hdfs dfs -put emp.csv emp_ext
```

```sql
CREATE EXTERNAL TABLE emp_ext(
    empno INT, empname STRING, age INT, gender STRING,
    Salary INT, Designation STRING, DeptID STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
LOCATION '/user/clouduser/emp_ext'
TBLPROPERTIES("skip.header.line.count"="1");
```

- `LOCATION` — tells Hive where in HDFS to read the data from (instead of the default warehouse directory)
- `TBLPROPERTIES("skip.header.line.count"="1")` — tells Hive to skip the first row (the CSV header) when reading data
- You can also apply this to an existing table:
  ```sql
  ALTER TABLE emp SET TBLPROPERTIES("skip.header.line.count"="1");
  ```

**Quick comparison:**

|                     | Managed Table                                            | External Table                                               |
| ------------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| Who owns the data   | Hive                                                     | You (Hive just references it)                                |
| `DROP TABLE` effect | Deletes data + metadata                                  | Deletes metadata only, data stays                            |
| Typical use case    | Data Hive fully controls (staging, intermediate results) | Data shared across tools, or you want it to survive a `DROP` |

---

## Loading Data Into a Table

There are a few ways to get data into a Hive table:

### a) Direct INSERT

```sql
INSERT INTO TABLE <tblname> VALUES (1, ....);
```

### b) LOAD from Local File System (LFS)

```sql
LOAD DATA LOCAL INPATH '<LFS path>' INTO TABLE <tblname>;

-- example
LOAD DATA LOCAL INPATH 'emp.csv' INTO TABLE emp;
```

This is functionally equivalent to copying the file straight into the table's warehouse directory:

```bash
hdfs dfs -put emp.csv /user/hive/warehouse/hadoopdb.db/emp
```

### c) LOAD from HDFS

```sql
LOAD DATA INPATH '<HDFS path>' INTO TABLE <tblname>;

-- example
LOAD DATA INPATH 'HadoopDir/emp.csv' INTO TABLE emp;
```

This is equivalent to a `cp` (not `put`, since the file is already in HDFS):

```bash
hdfs dfs -cp HadoopDir/emp.csv /user/hive/warehouse/hadoopdb.db/emp
```

**Key difference:** `LOAD DATA LOCAL INPATH` reads from the machine's local disk; `LOAD DATA INPATH` (no `LOCAL`) reads from HDFS. Both end up moving/copying the file into the table's storage location (or wherever `LOCATION` points, for external tables).

---

## Describing a Table — the DESCRIBE Family

| Command                   | What it shows                                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `DESCRIBE emp;`           | Basic column names and data types                                                                            |
| `DESCRIBE EXTENDED emp;`  | Above + extra metadata (storage location, table type, properties) as one dense line                          |
| `DESCRIBE FORMATTED emp;` | Same info as EXTENDED, but neatly formatted/human-readable — the one to use when actually inspecting a table |

Useful for checking whether a table is Managed or External, without needing to remember how you created it:

```sql
DESCRIBE FORMATTED emp;
DESCRIBE FORMATTED emp_ext;
```

---

## Dropping Tables & Cleanup

```sql
DROP TABLE emp;
SHOW TABLES;

DROP TABLE emp_ext;
SHOW TABLES;
```

Remember: dropping `emp` (managed) deletes its data from HDFS. Dropping `emp_ext` (external) only removes the table definition — the file at its `LOCATION` remains in HDFS.

You can always double check the raw warehouse directory:

```bash
hdfs dfs -ls /user/hive/warehouse/
```

---

## Accessing the Hive Metastore Directly (MySQL)

Hive's metastore (table/DB metadata) is backed by MySQL, and can be queried directly if needed:

```bash
mysql -u root -p
# Password: root
```

---

## YARN Config Snippet (Reference)

These properties came up in the context of configuring YARN's NodeManager to bind to a specific VM's IP (relevant if Hive/MapReduce jobs weren't reaching the NodeManager correctly):

```xml
<property>
  <name>yarn.nodemanager.hostname</name>
  <value>YOUR_VM_IP</value>
</property>
<property>
  <name>yarn.nodemanager.bind-host</name>
  <value>0.0.0.0</value>
</property>
<property>
  <name>yarn.nodemanager.webapp.address</name>
  <value>YOUR_VM_IP:8042</value>
</property>
```

After editing YARN config, restart the cluster:

```bash
stop-all.sh
start-dfs.sh; start-yarn.sh
```

---

## Hands-On Activity 1 — GoShopping IP Lookup

**File:** `goShopping_IpLookup.txt`
**Path:** `/home/lkmuser/dataset/sourcedata/webclicksdata`

**Schema** (all fields as `STRING`):

| Field       | Description                                            |
| ----------- | ------------------------------------------------------ |
| `IP`        | The IP address of the user                             |
| `Country`   | Country where the IP accessed the website              |
| `State`     | State of the country where the IP accessed the website |
| `City`      | City where the IP accessed the website                 |
| `ApproxLat` | Approximate latitude of access                         |
| `ApproxLng` | Approximate longitude of access                        |

**Tasks:**

1. Create a database named `Webclicks`
2. Create an **external** table named `goshopping` with the schema above
3. Load the data from LFS into the table
4. Display the table properties along with table type (`DESCRIBE FORMATTED`)
5. Display the data in the table (`SELECT * FROM goshopping;`)

**Worked approach (following the pattern from `emp_ext` above):**

```sql
CREATE DATABASE Webclicks;
USE Webclicks;

CREATE EXTERNAL TABLE goshopping(
    IP STRING,
    Country STRING,
    State STRING,
    City STRING,
    ApproxLat STRING,
    ApproxLng STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n';

LOAD DATA LOCAL INPATH '/home/lkmuser/dataset/sourcedata/webclicksdata/goShopping_IpLookup.txt'
INTO TABLE goshopping;

DESCRIBE FORMATTED goshopping;

SELECT * FROM goshopping;
```

---

## Hands-On Activity 2 — GoShopping Web Clicks

**File:** `goShopping_WebClicks.dat`
**Path:** `/home/lkmuser/dataset/sourcedata/webclicksdata`
**Note:** fields are separated by a **tab (`\t`)** delimiter, not a comma.

**Schema** (9 fields):

| Field            | Data Type | Description                                     |
| ---------------- | --------- | ----------------------------------------------- |
| `date`           | STRING    | Date the user (IP) accessed the website         |
| `time`           | STRING    | Time the user (IP) accessed the website         |
| `hostIp`         | STRING    | IP address of the machine hosting the website   |
| `cs-method`      | STRING    | Request method used to access the content       |
| `customer-ip`    | STRING    | IP address of the user who accessed the website |
| `url`            | STRING    | URL accessed by the user                        |
| `time-spent`     | INT       | Total time (seconds) spent on the URL           |
| `redirectedFrom` | STRING    | Source URL the request was redirected from      |
| `deviceType`     | STRING    | Type of device used to access the website       |

**Tasks:**

1. Place the file into an HDFS location
2. Create an **external** table named `webclicks` (inside the `Webclicks` DB) with the schema above, referencing the file from that HDFS location
3. Display the table properties along with table type
4. Display the data in the table

**Worked approach:**

```bash
hdfs dfs -mkdir /user/clouduser/webclicks_ext
hdfs dfs -put /home/lkmuser/dataset/sourcedata/webclicksdata/goShopping_WebClicks.dat /user/clouduser/webclicks_ext
```

```sql
USE Webclicks;

CREATE EXTERNAL TABLE webclicks(
    `date` STRING,
    `time` STRING,
    hostIp STRING,
    `cs-method` STRING,
    `customer-ip` STRING,
    url STRING,
    `time-spent` INT,
    redirectedFrom STRING,
    deviceType STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
LOCATION '/user/clouduser/webclicks_ext';

DESCRIBE FORMATTED webclicks;

SELECT * FROM webclicks;
```

> Note: `date` and `time` are reserved-ish words in some SQL dialects — backticks (`` `date` ``) keep Hive from misreading them as keywords. `time-spent` and similar hyphenated names also need backticks since Hive would otherwise parse the hyphen as subtraction.

---
