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

_More notes to be added._
