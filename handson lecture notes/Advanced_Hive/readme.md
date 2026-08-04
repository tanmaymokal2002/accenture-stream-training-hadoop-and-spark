# Hive Advanced Notes — Activities & Performance Tuning (Partitioning)

_Compiled 01 Aug 2026_

---

## How to use this document

This document starts with **why** each technique or activity exists (the use case), then walks through the actual commands **one at a time**, explaining every word in the command the first time it appears, followed by real example outputs where available. It covers, in order: a full NYC Yellow Taxi HDFS + Hive external table setup activity, a data validation + analytical queries activity built on top of it, and then the core partitioning concepts (static and dynamic).

---

## Activity A — NYC Yellow Taxi: HDFS Setup + Hive External Table

**Use case:** Before you can run any Hive query on real-world data, the raw files have to get from your local disk into HDFS, and Hive has to be told how to interpret them as a table. This activity walks through that full pipeline once, end to end, using real NYC Yellow Taxi trip data — the same table (`YellowTbl`) is reused as the data source for Activity B below.

### Task list

1. Create an HDFS directory named `YellowTbl` under the HDFS home directory.
2. Place both CSV files from the local path `/home/clouduser/dataset/sourcedata/nyctaxidata/yellow/good-data` into the `YellowTbl` HDFS directory.
3. Create an external table named `YellowTbl`, using the CSV file's schema, pointing to the HDFS directory created above.
4. Display the table type (to confirm it is `EXTERNAL_TABLE`).
5. Display the first 10 records of `YellowTbl`.

### Step 0 — Unzip the source data (prerequisite)

The raw data ships as a zip file under `/home/clouduser/dataset/sourcedata/`. Extract it first so the CSVs are available on the local filesystem.

```bash
cd /home/clouduser/dataset/sourcedata/
unzip nyctaxidata.zip
```

If `unzip` isn't installed:

```bash
sudo apt install unzip -y
```

- `sudo` = run the command with administrator privileges — required to install new software.
- `apt install unzip -y` = Ubuntu's package manager installs the `unzip` tool; `-y` auto-confirms the "do you want to continue?" prompt.

Confirm the CSVs are now present at the expected path:

```bash
ls -lh /home/clouduser/dataset/sourcedata/nyctaxidata/yellow/good-data
```

- `-h` here (combined with `-l`, long listing) = **h**uman-readable file sizes (KB/MB/GB instead of raw bytes) — same flag idea as `hdfs dfs -du -h` from the main Hadoop notes.

You should see the two files:

```
yellow_tripdata_2019-05.csv
yellow_tripdata_2019-06.csv
```

### Step 1 — Create the HDFS directory

```bash
hdfs dfs -mkdir /user/clouduser/YellowTbl
```

Same `mkdir` pattern covered in the main Hadoop notes — since a full leading path (`/user/clouduser/...`) is given here, it lands explicitly in your HDFS home directory regardless of which local folder you're currently sitting in.

Verify:

```bash
hdfs dfs -ls /user/clouduser/
```

### Step 2 — Copy local CSV files into HDFS

```bash
hdfs dfs -put /home/clouduser/dataset/sourcedata/nyctaxidata/yellow/good-data/*.csv /user/clouduser/YellowTbl/
```

- `*.csv` = a **wildcard** — matches every file ending in `.csv` in that folder, uploading both files in a single command instead of running `-put` twice.

Verify:

```bash
hdfs dfs -ls /user/clouduser/YellowTbl/
```

**Expected output — 2 files:**

```
yellow_tripdata_2019-05.csv
yellow_tripdata_2019-06.csv
```

### Step 3 — Create the external Hive table

Check the CSV header first, to match column names exactly:

```bash
head -1 /home/clouduser/dataset/sourcedata/nyctaxidata/yellow/good-data/yellow_tripdata_2019-05.csv
```

- `head` = a Linux utility that prints the first lines of a file; `-1` limits it to just the first line — the CSV's header row, so you can confirm the real column names/order before writing the table schema.

Then launch Hive:

```bash
hive
```

This drops you into the interactive Hive shell (`hive>` prompt), where SQL-style HQL commands run directly, same as `create-emp-dept.hql` style commands from earlier in the main notes.

Create the external table (standard NYC Yellow Taxi schema — adjust columns if your header differs):

```sql
CREATE EXTERNAL TABLE YellowTbl (
    VendorID INT,
    tpep_pickup_datetime TIMESTAMP,
    tpep_dropoff_datetime TIMESTAMP,
    passenger_count INT,
    trip_distance DOUBLE,
    RatecodeID INT,
    store_and_fwd_flag STRING,
    PULocationID INT,
    DOLocationID INT,
    payment_type INT,
    fare_amount DOUBLE,
    extra DOUBLE,
    mta_tax DOUBLE,
    tip_amount DOUBLE,
    tolls_amount DOUBLE,
    improvement_surcharge DOUBLE,
    total_amount DOUBLE,
    congestion_surcharge DOUBLE
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/clouduser/YellowTbl/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

- `TIMESTAMP` / `DOUBLE` = new data types here compared to earlier examples — `TIMESTAMP` stores a full date+time value (needed for pickup/dropoff times), `DOUBLE` stores decimal numbers (needed for money and distance fields, where `INT` would lose precision).
- `STORED AS TEXTFILE` = explicitly tells Hive the underlying files are plain text (CSV), as opposed to a binary format like ORC or Parquet mentioned earlier in the main notes.

**Note:** `LOCATION` points Hive directly at the HDFS directory — this is what makes it an external table. Hive reads the files in place without moving or owning them, so dropping the table later will not delete the underlying CSV files.

### Step 4 — Display the table type

```sql
DESCRIBE FORMATTED YellowTbl;
```

- `FORMATTED` = an extended version of `DESCRIBE` (more detailed than the plain `DESCRIBE` or `DESCRIBE EXTENDED` used earlier) — dumps full metadata including storage details, table type, and location.

Look for the `Table Type` row in the output — it should read:

```
Table Type:             EXTERNAL_TABLE
```

**Alternative (from shell, filters directly to the relevant line):**

```bash
hive -e "DESCRIBE FORMATTED YellowTbl;" | grep "Table Type"
```

- `hive -e "<query>"` = runs a single HQL query directly from the Linux shell, without opening the interactive `hive>` prompt first.
- `| grep "Table Type"` = pipes the query's output into `grep`, which filters and prints only the line(s) containing the text "Table Type" — useful for pulling one specific fact out of a long metadata dump.

### Step 5 — Display the first 10 records

```sql
SELECT * FROM YellowTbl LIMIT 10;
```

- `LIMIT 10` = caps the result set to the first 10 rows returned — standard SQL, prevents flooding your screen when previewing a large table.

### Summary of commands

| Step                 | Command                                                                        |
| -------------------- | ------------------------------------------------------------------------------ |
| 0. Unzip source data | `unzip nyctaxidata.zip`                                                        |
| 1. Create HDFS dir   | `hdfs dfs -mkdir /user/clouduser/YellowTbl`                                    |
| 2. Upload CSVs       | `hdfs dfs -put <local-path>/*.csv /user/clouduser/YellowTbl/`                  |
| 3. Create table      | `CREATE EXTERNAL TABLE YellowTbl (...) LOCATION '/user/clouduser/YellowTbl/';` |
| 4. Check type        | `DESCRIBE FORMATTED YellowTbl;`                                                |
| 5. Preview data      | `SELECT * FROM YellowTbl LIMIT 10;`                                            |

### ⚠️ Notes / Gotchas

- If `head -1` shows a different column order/count than listed above, update the `CREATE TABLE` column list to match — mismatched columns cause `NULL` values, not errors. Hive won't warn you; it just silently misaligns the data.
- If the CSV has no header row, remove the `TBLPROPERTIES ("skip.header.line.count"="1")` line.
- Since this is an external table, `DROP TABLE YellowTbl;` only removes Hive's metadata — the CSV files remain safely in HDFS.

---

## Activity B — Validate Trip Details & Run Analytical Queries

**Use case:** Raw ingested data is rarely clean — sensor errors, cancelled trips, or bad GPS logging can leave rows with zero distance, zero fare, missing location IDs, or a pickup time identical to the dropoff time (data that clearly doesn't represent a real trip). Before running any real analysis, you filter this out into a **validated** table, then treat that clean table — not the raw one — as the source of truth for all reporting queries below.

This activity builds directly on `YellowTbl` from Activity A.

### Step 1 — Create the validated table

**Validation criteria:**

1. `tpep_pickup_datetime` and `tpep_dropoff_datetime` should **not** be the same.
2. `passenger_count > 0`, `trip_distance > 0`, `fare_amount > 0`, `total_amount > 0`.
3. `PULocationID` should not be null, `DOLocationID` should not be null.

```sql
CREATE TABLE ValidYellowTrip AS
SELECT *
FROM YellowTbl
WHERE tpep_pickup_datetime != tpep_dropoff_datetime
  AND passenger_count > 0
  AND trip_distance > 0
  AND fare_amount > 0
  AND total_amount > 0
  AND PULocationID IS NOT NULL
  AND DOLocationID IS NOT NULL;
```

- `CREATE TABLE ... AS SELECT` (often abbreviated **CTAS**) = creates a brand-new table whose structure _and_ contents are both derived directly from a query's result — you don't need to declare columns manually, Hive infers them from the `SELECT`.
- `!=` = "not equal to" — the standard SQL inequality operator, used here to exclude rows where pickup and dropoff time are identical.
- `IS NOT NULL` = a specific SQL check for missing/absent values — you cannot use `= NULL` in SQL (it never evaluates to true), `IS NOT NULL` is the correct syntax.
- Every condition is joined with `AND`, meaning a row only survives into `ValidYellowTrip` if it satisfies **all** of them simultaneously.

**Sanity check — confirm it worked and see how many rows survived filtering:**

```sql
SELECT COUNT(*) FROM ValidYellowTrip;
```

---

### Analytical queries on `ValidYellowTrip`

Each query below explains any new SQL concept the first time it appears. Run these against your own loaded data — the actual row counts and totals will depend on your specific dataset, so no fabricated output is shown here; use `SELECT ... LIMIT 10` first if you want to eyeball the shape of the result before trusting the aggregate.

**1. List all trips made by Vendor 1**

```sql
SELECT * FROM ValidYellowTrip WHERE VendorID = 1;
```

A simple filter — same `WHERE` pattern seen throughout this document.

**2. Find trips with congestion surcharge applied**

```sql
SELECT * FROM ValidYellowTrip WHERE congestion_surcharge > 0;
```

Any trip where a nonzero congestion fee was actually charged.

**3. Display all trips where tip amount is greater than 0**

```sql
SELECT * FROM ValidYellowTrip WHERE tip_amount > 0;
```

**4. Find total number of trips for each vendor**

```sql
SELECT VendorID, COUNT(*) AS total_trips
FROM ValidYellowTrip
GROUP BY VendorID;
```

- `COUNT(*)` = an **aggregate function** — counts the number of rows in each group, rather than returning individual rows.
- `AS total_trips` = an **alias** — renames the output column for readability; without it, the column would just be labeled with the raw expression `count(*)`.
- `GROUP BY VendorID` = collapses all rows sharing the same `VendorID` into a single summary row — this is what makes `COUNT(*)` calculate "per vendor" instead of one grand total across the whole table.

**5. Find total trip distance travelled by each vendor**

```sql
SELECT VendorID, SUM(trip_distance) AS total_distance
FROM ValidYellowTrip
GROUP BY VendorID;
```

- `SUM(...)` = another aggregate function — adds up a numeric column's values within each group, instead of counting rows.

**6. Find total fare amount earned by each vendor, for each pickup location id**

```sql
SELECT VendorID, PULocationID, SUM(fare_amount) AS total_fare
FROM ValidYellowTrip
GROUP BY VendorID, PULocationID;
```

- `GROUP BY VendorID, PULocationID` = grouping by **two columns** together — each unique _combination_ of vendor and pickup location gets its own summary row, giving a more granular breakdown than grouping by vendor alone.

**7. Find total travel time in hours by each vendor for each drop location id — save the result as `vendor_travel_time`**

```sql
CREATE TABLE vendor_travel_time AS
SELECT VendorID, DOLocationID,
       SUM((UNIX_TIMESTAMP(tpep_dropoff_datetime) - UNIX_TIMESTAMP(tpep_pickup_datetime)) / 3600.0) AS total_travel_time_hours
FROM ValidYellowTrip
GROUP BY VendorID, DOLocationID;
```

- `UNIX_TIMESTAMP(...)` = converts a `TIMESTAMP` value into a plain number — specifically, the number of seconds elapsed since 1 Jan 1970 ("Unix epoch"). Subtracting two of these gives you a trip's duration **in seconds**.
- `/ 3600.0` = converts seconds into hours (there are 3600 seconds in an hour); the `.0` forces decimal (floating-point) division instead of accidentally truncating to a whole number.
- Since the query is wrapped in `CREATE TABLE vendor_travel_time AS`, the aggregated result is saved as its own permanent table, not just displayed once and discarded.

**8. Count trips by payment type**

```sql
SELECT payment_type, COUNT(*) AS trip_count
FROM ValidYellowTrip
GROUP BY payment_type;
```

**9. Find total revenue for each payment type**

```sql
SELECT payment_type, SUM(total_amount) AS total_revenue
FROM ValidYellowTrip
GROUP BY payment_type;
```

**10. Calculate trip duration in minutes**

```sql
SELECT *,
       (UNIX_TIMESTAMP(tpep_dropoff_datetime) - UNIX_TIMESTAMP(tpep_pickup_datetime)) / 60.0 AS trip_duration_minutes
FROM ValidYellowTrip;
```

Same duration calculation as query 7, but divided by 60 (seconds per minute) instead of 3600, and computed **per row** here (via `SELECT *,`) rather than summed per group — so every trip gets its own individual duration value rather than a vendor/location total.

**11. Which payment type is most commonly used?**

```sql
SELECT payment_type, COUNT(*) AS cnt
FROM ValidYellowTrip
GROUP BY payment_type
ORDER BY cnt DESC
LIMIT 1;
```

- `ORDER BY cnt DESC` = sorts the grouped results by the `cnt` alias, largest first (`DESC` = **desc**ending order; the opposite, `ASC`, is ascending/smallest-first and is the default if omitted).
- `LIMIT 1` = keeps only the very top row after sorting — i.e. whichever payment type had the highest trip count.

---

Once a Hive table grows large (millions of rows), a query that filters on one column — e.g. "show me only department D001's employees" — still has to scan the **entire table** unless Hive knows how to skip irrelevant data. This is slow and wasteful.

**Partitioning** solves this by splitting the table's underlying data into smaller physical sub-folders based on a column's value, so a filtered query only reads the relevant folder instead of the whole table.

- Split the data into more, smaller parts based on one or more columns (called **partition keys**).
- Example physical layout on disk:
  ```
  hiveadv.db -> emp (sub-dir) -> dept_id=D001 (D1), dept_id=D002 (D2)....
  ```
  Each distinct department value gets its own sub-folder — so a query for `deptid='D001'` only touches the `D1` folder, not the entire table.

---

## Types of Partitions

- **a. Static Partitioning** — you manually specify the exact partition value yourself when inserting data (e.g. `PARTITION(deptid='D001')`), one value at a time.
- **b. Dynamic Partitioning** — Hive automatically creates partitions for you based on the values found in the data, without you having to list them one by one.

Dynamic partitioning requires two session settings to be turned on first:

```sql
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode= nonstrict;
```

- `SET` = changes a Hive session configuration property, for the current session only.
- `hive.exec.dynamic.partition = true` = the master switch — turns dynamic partitioning **on** (it is off by default).
- `hive.exec.dynamic.partition.mode = nonstrict` = Hive has two modes for dynamic partitioning:
  - **strict** — requires at least one partition column to be static/manually specified, as a safety net against accidentally creating a huge number of partitions by mistake.
  - **nonstrict** — allows _all_ partition columns to be determined dynamically from the data, with no manual value required at all.

  Setting it to `nonstrict` removes that safety restriction, letting Hive fully auto-detect every partition value from the source data.

---

## Steps — Static Partitioning, end to end

### 1. Create a Parent (source) table

```sql
CREATE EXTERNAL TABLE emp_ext(empno INT, empname STRING, age INT, gender STRING, Salary INT, Designation STRING, DeptID STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY'\n'
LOCATION '/user/clouduser/emp_ext'
TBLPROPERTIES("skip.header.line.count"="1");
```

- `CREATE EXTERNAL TABLE` = `EXTERNAL` means Hive only manages the _metadata_ for this table — the actual data file stays wherever you point `LOCATION` to, and dropping the table later won't delete the underlying data (contrast with a plain `CREATE TABLE`, which is "managed" and deletes data on drop).
- `ROW FORMAT DELIMITED FIELDS TERMINATED BY ','` = tells Hive the raw source file is comma-separated (i.e. a CSV).
- `LINES TERMINATED BY '\n'` = each new line in the file represents one new row/record.
- `LOCATION '/user/clouduser/emp_ext'` = the HDFS path where the actual data file for this table physically lives.
- `TBLPROPERTIES("skip.header.line.count"="1")` = tells Hive to ignore the first line of the file (the CSV header row containing column names like `empno,empname,...`) so it isn't mistakenly read in as an actual data row.

### 2. Create the partitioned (target) table

```sql
CREATE TABLE emp_sp(empno INT, empname STRING, age INT, gender STRING, Salary INT, Designation STRING)
PARTITIONED BY(DeptID STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY'\n';
```

- `PARTITIONED BY(DeptID STRING)` = the key new clause — declares that this table's data should be physically split into sub-folders based on the `DeptID` column.
- ⚠️ Notice `DeptID` is **not** repeated in the main column list above it. Hive treats partition columns as separate, virtual columns layered on top of the real data columns — listing it twice would be a syntax error.

**Verify the structure:**

```sql
DESCRIBE emp_sp;
```

**Example output:**

```
col_name       data_type    comment
empno          int
empname        string
age            int
gender         string
salary         int
designation    string
deptid         string

# Partition Information
# col_name     data_type    comment
deptid         string
```

Notice `deptid` appears **twice** in this output — once listed with the regular columns, and again explicitly under the "Partition Information" block. This double-listing is exactly how Hive confirms a column is being treated as a partition key rather than an ordinary data column.

### 3. Insert data into the partition table from the parent table (static example)

```sql
INSERT OVERWRITE TABLE emp_sp PARTITION(deptid='D001')
SELECT empno, empname, age, gender, salary, designation FROM emp_ext WHERE deptid='D001';
```

- `INSERT OVERWRITE TABLE` = replaces any existing data in the target table (or target partition) with the new query's results.
- `PARTITION(deptid='D001')` = this is the **static** part of static partitioning — you are manually telling Hive exactly which partition value this insert belongs to, rather than letting Hive infer it automatically from the data itself.
- `SELECT ... WHERE deptid='D001'` = pulls only the matching rows from the external source table `emp_ext`, so only D001's employees get written into this specific partition.

### 4. Display the contents of the partition table and its partition names

```sql
SELECT * FROM emp_sp;
```

**Example output:**

```
emp_sp.empno  emp_sp.empname  emp_sp.age  emp_sp.gender  emp_sp.salary  emp_sp.designation  emp_sp.deptid
1201          gopal           45          Male            50000          AM                  D001
1203          khalil          27          Male            30000          SSE                 D001
1207          bhavya          24          Female           15000          ASE                 D001
1212          Arshad          23          Male             20000          ASE                 D001
Time taken: 0.189 seconds, Fetched: 4 row(s)
```

All 4 returned rows correctly show `D001` in the `deptid` column, since that's the only partition we've loaded so far.

```sql
SHOW PARTITIONS emp_sp;
```

- `SHOW PARTITIONS` = lists every partition value that currently exists for this table — i.e. every distinct sub-folder that has actually been created on disk so far.

**Example output:**

```
partition
deptid=D001
```

Only one partition exists (`D001`), because that's the only value we've loaded using static partitioning. If you repeated step 3 with `PARTITION(deptid='D002')`, `PARTITION(deptid='D003')`, etc., each would appear as an additional line here. This is also exactly where **dynamic partitioning** saves effort — instead of repeating step 3 manually for every department, turning on `hive.exec.dynamic.partition` with `nonstrict` mode lets a single `INSERT` statement (without a hardcoded `PARTITION(deptid=...)` value) automatically create and populate a partition for every distinct `deptid` found in the source data, all at once.

---

## Steps — Dynamic Partitioning, end to end

**Use case:** With static partitioning, loading 4 departments means writing 4 separate `INSERT` statements by hand (one per `PARTITION(deptid='...')`). Dynamic partitioning lets Hive look at the incoming data itself and automatically create — and populate — a partition for every distinct value it finds, in a single statement. This is what `hive.exec.dynamic.partition = true` and `nonstrict` mode (explained above) unlock.

### 1. Create the partitioned (target) table — same shape as before

```sql
CREATE TABLE emp_dp(empno INT, empname STRING, age INT, gender STRING, Salary INT, Designation STRING)
PARTITIONED BY(DeptID STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY'\n';
```

This is structurally identical to `emp_sp` from the static example — same columns, same single partition key (`DeptID`). Only how we _insert into_ it will differ.

```sql
DESCRIBE emp_dp;
```

**Example output:**

```
col_name       data_type    comment
empno          int
empname        string
age            int
gender         string
salary         int
designation    string
deptid         string

# Partition Information
# col_name     data_type    comment
deptid         string
```

Same double-listing of `deptid` as before, confirming it's a partition key.

### 2. Insert data — the dynamic way (no partition value specified)

```sql
INSERT OVERWRITE TABLE emp_dp PARTITION(deptid) SELECT * FROM emp_ext;
```

- `PARTITION(deptid)` = notice there is **no `='D001'` value here**, unlike the static example (`PARTITION(deptid='D001')`). Leaving the value out is exactly what tells Hive "figure out the partition values yourself, from the data."
- `SELECT * FROM emp_ext` = pulls **every row** from the source table, not filtered to one department this time — Hive will sort each row into the correct partition automatically based on that row's own `deptid` value.
- ⚠️ **Column order matters here:** the partition column (`deptid`) must be the **last** column in the `SELECT` list, matching its position in the `PARTITIONED BY` clause — Hive maps columns positionally, not by name, for this kind of insert.

### 3. Verify — one statement created ALL partitions at once

```sql
SHOW PARTITIONS emp_dp;
```

**Example output:**

```
partition
deptid=D001
deptid=D002
deptid=D003
deptid=D004
Time taken: 0.08 seconds, Fetched: 4 row(s)
```

Compare this to the static example, where `SHOW PARTITIONS emp_sp` showed only `deptid=D001` after one manual insert. Here, a **single** dynamic `INSERT` created all 4 department partitions in one shot.

```sql
SELECT * FROM emp_dp;
```

**Example output (excerpt):**

```
emp_dp.empno  emp_dp.empname  emp_dp.age  emp_dp.gender  emp_dp.salary  emp_dp.designation  emp_dp.deptid
1201          gopal           45          Male            50000          AM                  D001
1202          manisha         40          Female           50000          AM                  D002
1205          kiran           29          Male             40000          Lead                D003
1206          laxmi           29          Female           35000          Lead                D004
...
Time taken: 0.147 seconds, Fetched: 15 row(s)
```

All 15 rows from the source table landed correctly, each automatically routed into its matching `deptid` partition.

---

## Dynamic Partitioning with Multiple Partition Keys

**Use case:** You're not limited to partitioning by a single column. If you frequently filter queries by _combinations_ of columns (e.g. "show me D001's Female employees"), you can partition by more than one key — Hive will nest the sub-folders, one level per key.

- **Partition keys used here:** `DeptID`, `gender`
- **Source:** loading from a local file `emp.csv` this time, instead of from another Hive table.

### 1. Create the multi-partition table

```sql
CREATE TABLE emp_mulitpart(empno INT, empname STRING, age INT, Salary INT, Designation STRING)
PARTITIONED BY(DeptID STRING, gender STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY'\n'
TBLPROPERTIES("skip.header.line.count"="1");
```

- `PARTITIONED BY(DeptID STRING, gender STRING)` = **two** partition keys instead of one. Notice both `DeptID` and `gender` are absent from the main column list — same rule as before, just applied to two columns now instead of one.
- The resulting folder structure on disk will be **nested**: a `DeptID` folder, containing `gender` sub-folders inside it (see the `SHOW PARTITIONS` output below).

### 2. Loading data — why `LOAD DATA` doesn't work here

```sql
LOAD DATA LOCAL INPATH 'emp.csv' INTO TABLE emp_mulitpart PARTITION(deptid, gender);
```

- `LOAD DATA LOCAL INPATH 'emp.csv'` = a command that copies a file directly from your **local filesystem** into a Hive table's HDFS location, without needing an intermediate `SELECT` query.
- ⚠️ **This does NOT work for dynamic partitioning with a different source structure.** `LOAD DATA` performs a simple, structural file copy — it has no way to _inspect_ each row's values to figure out which partition it belongs to. Dynamic partitioning needs Hive to actually **read and evaluate** each row's partition column values, which only a `SELECT`-based `INSERT` can do.

**The working alternative — insert from a query instead:**

```sql
INSERT OVERWRITE TABLE emp_mulitpart PARTITION(deptid, gender)
SELECT empno, empname, age, Salary, designation, deptid, gender FROM emp_ext;
```

- Same dynamic-partition pattern as the single-key example — `PARTITION(deptid, gender)` lists both partition keys with **no values**, letting Hive determine both automatically per row.
- ⚠️ Again, column order matters: `deptid` and `gender` are placed **last** in the `SELECT` list, in the same order they're declared in `PARTITIONED BY(DeptID STRING, gender STRING)`.

### 3. Verify — nested partitions for every DeptID + gender combination

```sql
SHOW PARTITIONS emp_mulitpart;
```

**Example output:**

```
partition
deptid=D001/gender=Female
deptid=D001/gender=Male
deptid=D002/gender=Female
deptid=D002/gender=Male
deptid=D003/gender=Female
deptid=D003/gender=Male
deptid=D004/gender=Female
deptid=D004/gender=Male
Time taken: 0.096 seconds, Fetched: 8 row(s)
```

Notice the `/` in each partition name — this reflects the actual **nested folder structure** on disk (e.g. `deptid=D001/gender=Female/`). With 4 departments × 2 genders, Hive automatically created all 8 combinations in one single `INSERT` statement — this is the real payoff of dynamic partitioning at scale: no one had to write 8 separate manual `INSERT ... PARTITION(...)` statements.

---

## Quick recap

| Concept                        | Why it matters                                                                                                                                          |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Partitioning                   | Speeds up filtered queries by physically splitting data so irrelevant folders are never scanned                                                         |
| Static partitioning            | You specify the partition value manually (`PARTITION(deptid='D001')`) — simple, safe, but repetitive for many values                                    |
| Dynamic partitioning           | Leave partition values blank (`PARTITION(deptid)`) and Hive infers all of them from the data in one `INSERT` — requires `nonstrict` mode enabled        |
| Multi-key dynamic partitioning | Partition by more than one column (`PARTITIONED BY(DeptID STRING, gender STRING)`) — creates nested sub-folders for every combination found in the data |
| Column order in `SELECT`       | For dynamic inserts, partition columns must appear **last** in the `SELECT` list, in the same order as `PARTITIONED BY`                                 |
| `LOAD DATA ... PARTITION(...)` | Does **not** work for dynamic partitioning — it's a plain file copy with no ability to inspect row values; use an `INSERT OVERWRITE ... SELECT` instead |
| External table as source       | Keeps raw ingested data safe from accidental deletion, separate from the partitioned/managed table built on top of it                                   |
| `SHOW PARTITIONS`              | Your sanity check — confirms exactly which partitions (or partition combinations) physically exist at any point in time                                 |
