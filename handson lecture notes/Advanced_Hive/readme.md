# NYC Yellow Taxi Data – Hive External Table Activity

## 📌 Activity Description

This activity demonstrates how to load raw CSV data (NYC Yellow Taxi trip records) into HDFS and expose it as a queryable Hive **external table**.

**Tasks:**

1. Create an HDFS directory named `YellowTbl` under the HDFS home directory.
2. Place both CSV files from the local path:
   `/home/clouduser/dataset/sourcedata/nyctaxidata/yellow/good-data`
   into the `YellowTbl` HDFS directory.
3. Create an **external table** named `YellowTbl`, using the CSV file's schema, pointing to the HDFS directory created above.
4. Display the table type (to confirm it is `EXTERNAL_TABLE`).
5. Display the first 10 records of `YellowTbl`.

---

## ✅ Solution

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

Confirm the CSVs are now present at the expected path:

```bash
ls -lh /home/clouduser/dataset/sourcedata/nyctaxidata/yellow/good-data
```

You should see the two files:

```
yellow_tripdata_2019-05.csv
yellow_tripdata_2019-06.csv
```

---

### Step 1 — Create HDFS directory

```bash
hdfs dfs -mkdir /user/clouduser/YellowTbl
```

Verify:

```bash
hdfs dfs -ls /user/clouduser/
```

---

### Step 2 — Copy local CSV files into HDFS

```bash
hdfs dfs -put /home/clouduser/dataset/sourcedata/nyctaxidata/yellow/good-data/*.csv /user/clouduser/YellowTbl/
```

Verify:

```bash
hdfs dfs -ls /user/clouduser/YellowTbl/
```

Expected output — 2 files:

```
yellow_tripdata_2019-05.csv
yellow_tripdata_2019-06.csv
```

---

### Step 3 — Create the external Hive table

Check the CSV header first, to match column names exactly:

```bash
head -1 /home/clouduser/dataset/sourcedata/nyctaxidata/yellow/good-data/yellow_tripdata_2019-05.csv
```

Then launch Hive:

```bash
hive
```

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

> **Note:** `LOCATION` points Hive directly at the HDFS directory — this is what makes it an _external_ table. Hive reads the files in place without moving or owning them, so dropping the table later will **not** delete the underlying CSV files.

---

### Step 4 — Display the table type

```sql
DESCRIBE FORMATTED YellowTbl;
```

Look for the `Table Type` row in the output — it should read:

```
Table Type:             EXTERNAL_TABLE
```

Alternative (from shell, filters directly to the relevant line):

```bash
hive -e "DESCRIBE FORMATTED YellowTbl;" | grep "Table Type"
```

---

### Step 5 — Display the first 10 records

```sql
SELECT * FROM YellowTbl LIMIT 10;
```

---

## 📂 Summary of Commands

| Step                 | Command                                                                        |
| -------------------- | ------------------------------------------------------------------------------ |
| 0. Unzip source data | `unzip nyctaxidata.zip`                                                        |
| 1. Create HDFS dir   | `hdfs dfs -mkdir /user/clouduser/YellowTbl`                                    |
| 2. Upload CSVs       | `hdfs dfs -put <local-path>/*.csv /user/clouduser/YellowTbl/`                  |
| 3. Create table      | `CREATE EXTERNAL TABLE YellowTbl (...) LOCATION '/user/clouduser/YellowTbl/';` |
| 4. Check type        | `DESCRIBE FORMATTED YellowTbl;`                                                |
| 5. Preview data      | `SELECT * FROM YellowTbl LIMIT 10;`                                            |

---

## ⚠️ Notes / Gotchas

- If `head -1` shows a different column order/count than listed above, update the `CREATE TABLE` column list to match — mismatched columns cause `NULL` values, not errors.
- If the CSV has no header row, remove the `TBLPROPERTIES ("skip.header.line.count"="1")` line.
- Since this is an **external** table, `DROP TABLE YellowTbl;` only removes Hive's metadata — the CSV files remain safely in HDFS.
