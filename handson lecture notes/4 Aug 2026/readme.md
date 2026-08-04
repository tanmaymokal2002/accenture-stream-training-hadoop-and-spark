# Hive Practice — Partitioning & Bucketing: Tasks & Solutions

_Builds on the NYC Yellow Taxi setup (`YellowTbl`) and the validated table (`ValidYellowTrip`) from the earlier Hive Advanced Notes._

---

## How to use this document

Every task below follows the same pattern: **what it's asking for**, **why it's asking for it**, then the **HQL to run**. All solutions assume `ValidYellowTrip` already exists and that dynamic-partitioning / bucketing session settings are turned on where needed.

Session settings used throughout — run these once per Hive session before any dynamic partition or bucket load:

```sql
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;
SET hive.enforce.bucketing = true;
```

---

# Part 1 — Partitioning

## 1.1 Create `YellowTrip_Payment_Part`, partitioned by `payment_type`, loaded dynamically

**Why:** `payment_type` has a small, fixed set of values (cash, card, no-charge, dispute, etc.), which is exactly the case partitioning is built for — queries that filter by payment type will only scan the relevant sub-folder.

```sql
CREATE TABLE YellowTrip_Payment_Part (
    VendorID               INT,
    tpep_pickup_datetime   TIMESTAMP,
    tpep_dropoff_datetime  TIMESTAMP,
    passenger_count        INT,
    trip_distance          DOUBLE,
    RatecodeID             INT,
    store_and_fwd_flag     STRING,
    PULocationID           INT,
    DOLocationID           INT,
    fare_amount            DOUBLE,
    extra                  DOUBLE,
    mta_tax                DOUBLE,
    tip_amount             DOUBLE,
    tolls_amount           DOUBLE,
    improvement_surcharge  DOUBLE,
    total_amount           DOUBLE,
    congestion_surcharge   DOUBLE
)
PARTITIONED BY (payment_type INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
```

- Notice `payment_type` is **not** in the main column list — it's declared once, in `PARTITIONED BY`, and Hive treats it as a separate virtual column.

```sql
INSERT OVERWRITE TABLE YellowTrip_Payment_Part PARTITION(payment_type)
SELECT VendorID, tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance,
       RatecodeID, store_and_fwd_flag, PULocationID, DOLocationID, fare_amount, extra, mta_tax,
       tip_amount, tolls_amount, improvement_surcharge, total_amount, congestion_surcharge,
       payment_type
FROM ValidYellowTrip;
```

- `PARTITION(payment_type)` with no `=value` = dynamic partitioning: Hive inspects every row and auto-creates one partition per distinct `payment_type` it finds.
- **Column order rule:** `payment_type` must be the **last** column in the `SELECT` list, because Hive maps `SELECT` columns to table columns positionally for this kind of insert, and it must line up with its position in `PARTITIONED BY`.

Verify:

```sql
SHOW PARTITIONS YellowTrip_Payment_Part;
```

## 1.2 Total trips and total revenue per payment-type partition

```sql
SELECT payment_type, COUNT(*) AS total_trips, SUM(total_amount) AS total_revenue
FROM YellowTrip_Payment_Part
GROUP BY payment_type;
```

- Since `payment_type` is the partition key, Hive can compute this by reading each partition folder's data independently, then combining the per-partition aggregates — no cross-scanning of irrelevant partitions needed the way a non-partitioned table would require.

## 1.3 Create a table partitioned by `VendorID`, loaded dynamically

```sql
CREATE TABLE YellowTrip_Vendor_Part (
    tpep_pickup_datetime   TIMESTAMP,
    tpep_dropoff_datetime  TIMESTAMP,
    passenger_count        INT,
    trip_distance          DOUBLE,
    RatecodeID             INT,
    store_and_fwd_flag     STRING,
    PULocationID           INT,
    DOLocationID           INT,
    payment_type           INT,
    fare_amount            DOUBLE,
    extra                  DOUBLE,
    mta_tax                DOUBLE,
    tip_amount             DOUBLE,
    tolls_amount           DOUBLE,
    improvement_surcharge  DOUBLE,
    total_amount           DOUBLE,
    congestion_surcharge   DOUBLE
)
PARTITIONED BY (VendorID INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

INSERT OVERWRITE TABLE YellowTrip_Vendor_Part PARTITION(VendorID)
SELECT tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance,
       RatecodeID, store_and_fwd_flag, PULocationID, DOLocationID, payment_type, fare_amount,
       extra, mta_tax, tip_amount, tolls_amount, improvement_surcharge, total_amount,
       congestion_surcharge, VendorID
FROM ValidYellowTrip;
```

## 1.4 Pickup location with the highest fare amount, within each Vendor partition

**Why:** this needs "top row per group," which a plain `GROUP BY` can't express on its own — a **window function** ranks rows inside each group so you can pick the #1 row per `VendorID`.

```sql
SELECT VendorID, PULocationID, total_fare
FROM (
    SELECT VendorID, PULocationID,
           SUM(fare_amount) AS total_fare,
           RANK() OVER (PARTITION BY VendorID ORDER BY SUM(fare_amount) DESC) AS rnk
    FROM YellowTrip_Vendor_Part
    GROUP BY VendorID, PULocationID
) ranked
WHERE rnk = 1;
```

- ⚠️ **Don't confuse the two "partition" words here.** `PARTITION BY VendorID` inside `RANK() OVER (...)` is a **SQL window-function clause** — it groups rows in-memory for ranking purposes only. It has nothing to do with the table's physical `PARTITIONED BY VendorID` clause used at `CREATE TABLE` time, even though both happen to use `VendorID` in this example.
- `RANK() OVER (...)` = assigns a rank (1, 2, 3...) to each row within its window; `ORDER BY SUM(fare_amount) DESC` ranks highest total fare first, so `rnk = 1` picks the winning `PULocationID` per vendor.

## 1.5 Multi-level partitioned table by `VendorID` and `payment_type`, then total trip distance per combination

```sql
CREATE TABLE YellowTrip_Vendor_Payment_Part (
    tpep_pickup_datetime   TIMESTAMP,
    tpep_dropoff_datetime  TIMESTAMP,
    passenger_count        INT,
    trip_distance          DOUBLE,
    RatecodeID             INT,
    store_and_fwd_flag     STRING,
    PULocationID           INT,
    DOLocationID           INT,
    fare_amount            DOUBLE,
    extra                  DOUBLE,
    mta_tax                DOUBLE,
    tip_amount             DOUBLE,
    tolls_amount           DOUBLE,
    improvement_surcharge  DOUBLE,
    total_amount           DOUBLE,
    congestion_surcharge   DOUBLE
)
PARTITIONED BY (VendorID INT, payment_type INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

INSERT OVERWRITE TABLE YellowTrip_Vendor_Payment_Part PARTITION(VendorID, payment_type)
SELECT tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance,
       RatecodeID, store_and_fwd_flag, PULocationID, DOLocationID, fare_amount, extra, mta_tax,
       tip_amount, tolls_amount, improvement_surcharge, total_amount, congestion_surcharge,
       VendorID, payment_type
FROM ValidYellowTrip;
```

- Both partition columns go **last**, in the same left-to-right order as `PARTITIONED BY(VendorID, payment_type)` — `VendorID` then `payment_type`.
- Physical layout is nested: `vendorid=1/payment_type=1/`, `vendorid=1/payment_type=2/`, etc.

```sql
SELECT VendorID, payment_type, SUM(trip_distance) AS total_distance
FROM YellowTrip_Vendor_Payment_Part
GROUP BY VendorID, payment_type;
```

---

# Part 2 — Bucketing

## 2.1 Bucketed table on `VendorID` into 4 buckets, loaded from `ValidYellowTrip`

**Why:** `VendorID` here has few distinct values too, but bucketing is shown for comparison — note how, unlike a partition key, the bucketing column **stays** in the regular column list.

```sql
CREATE TABLE YellowTrip_Vendor_Bucket (
    tpep_pickup_datetime   TIMESTAMP,
    tpep_dropoff_datetime  TIMESTAMP,
    passenger_count        INT,
    trip_distance          DOUBLE,
    RatecodeID             INT,
    store_and_fwd_flag     STRING,
    PULocationID           INT,
    DOLocationID           INT,
    payment_type           INT,
    fare_amount             DOUBLE,
    extra                  DOUBLE,
    mta_tax                DOUBLE,
    tip_amount              DOUBLE,
    tolls_amount            DOUBLE,
    improvement_surcharge   DOUBLE,
    total_amount            DOUBLE,
    congestion_surcharge    DOUBLE,
    VendorID                INT
)
CLUSTERED BY (VendorID) INTO 4 BUCKETS
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

INSERT OVERWRITE TABLE YellowTrip_Vendor_Bucket
SELECT tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance,
       RatecodeID, store_and_fwd_flag, PULocationID, DOLocationID, payment_type, fare_amount,
       extra, mta_tax, tip_amount, tolls_amount, improvement_surcharge, total_amount,
       congestion_surcharge, VendorID
FROM ValidYellowTrip;
```

## 2.2 Display `VendorID`, `hash(VendorID)`, and the bucket number via `pmod(hash(VendorID), 4)`

```sql
SELECT DISTINCT VendorID, hash(VendorID), pmod(hash(VendorID), 4) AS bucket_num
FROM ValidYellowTrip;
```

- `pmod(a, b)` = **positive modulo** — same idea as `%`, but always returns a non-negative remainder even if `hash(...)` is negative. Hive's internal bucketing logic uses `pmod`, not plain `%`, so this is the accurate way to predict which bucket a value actually lands in (plain `%` can disagree with Hive's real placement whenever the hash is negative).

## 2.3 Bucketed table on `PULocationID` into 8 buckets — count records per bucket

```sql
CREATE TABLE YellowTrip_PULoc_Bucket (
    tpep_pickup_datetime   TIMESTAMP,
    tpep_dropoff_datetime  TIMESTAMP,
    passenger_count        INT,
    trip_distance           DOUBLE,
    RatecodeID              INT,
    store_and_fwd_flag      STRING,
    DOLocationID             INT,
    payment_type             INT,
    fare_amount              DOUBLE,
    extra                   DOUBLE,
    mta_tax                 DOUBLE,
    tip_amount               DOUBLE,
    tolls_amount             DOUBLE,
    improvement_surcharge    DOUBLE,
    total_amount             DOUBLE,
    congestion_surcharge     DOUBLE,
    VendorID                 INT,
    PULocationID              INT
)
CLUSTERED BY (PULocationID) INTO 8 BUCKETS
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

INSERT OVERWRITE TABLE YellowTrip_PULoc_Bucket
SELECT tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance,
       RatecodeID, store_and_fwd_flag, DOLocationID, payment_type, fare_amount, extra, mta_tax,
       tip_amount, tolls_amount, improvement_surcharge, total_amount, congestion_surcharge,
       VendorID, PULocationID
FROM ValidYellowTrip;
```

Two ways to count records per bucket:

**Option A — group by the physical file each row came from:**

```sql
SELECT INPUT__FILE__NAME AS bucket_file, COUNT(*) AS record_count
FROM YellowTrip_PULoc_Bucket
GROUP BY INPUT__FILE__NAME;
```

- `INPUT__FILE__NAME` = a Hive **virtual column** (available on every table, not declared in `CREATE TABLE`) that returns the physical HDFS file a given row was read from. Since bucketing writes exactly one file per bucket, grouping by this column is equivalent to grouping by bucket number.

**Option B — sample each bucket explicitly and count:**

```sql
SELECT COUNT(*) FROM YellowTrip_PULoc_Bucket TABLESAMPLE(BUCKET 1 OUT OF 8);
-- repeat for BUCKET 2 OUT OF 8, BUCKET 3 OUT OF 8, ... BUCKET 8 OUT OF 8
```

- More explicit and easier to reason about one bucket at a time, but requires 8 separate queries — Option A gets you all 8 counts in a single query.

## 2.4 Total fare amount and average trip distance for each bucket (VendorID bucketing)

```sql
SELECT INPUT__FILE__NAME AS bucket_file,
       SUM(fare_amount)    AS total_fare,
       AVG(trip_distance)  AS avg_trip_distance
FROM YellowTrip_Vendor_Bucket
GROUP BY INPUT__FILE__NAME;
```

## 2.5 Bucketed table on `payment_type` into 4 buckets — which bucket has the most trips?

```sql
CREATE TABLE YellowTrip_Payment_Bucket (
    tpep_pickup_datetime   TIMESTAMP,
    tpep_dropoff_datetime  TIMESTAMP,
    passenger_count        INT,
    trip_distance           DOUBLE,
    RatecodeID               INT,
    store_and_fwd_flag       STRING,
    PULocationID              INT,
    DOLocationID              INT,
    fare_amount               DOUBLE,
    extra                    DOUBLE,
    mta_tax                  DOUBLE,
    tip_amount                DOUBLE,
    tolls_amount              DOUBLE,
    improvement_surcharge     DOUBLE,
    total_amount              DOUBLE,
    congestion_surcharge      DOUBLE,
    VendorID                  INT,
    payment_type               INT
)
CLUSTERED BY (payment_type) INTO 4 BUCKETS
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

INSERT OVERWRITE TABLE YellowTrip_Payment_Bucket
SELECT tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance,
       RatecodeID, store_and_fwd_flag, PULocationID, DOLocationID, fare_amount, extra, mta_tax,
       tip_amount, tolls_amount, improvement_surcharge, total_amount, congestion_surcharge,
       VendorID, payment_type
FROM ValidYellowTrip;

SELECT INPUT__FILE__NAME AS bucket_file, COUNT(*) AS trip_count
FROM YellowTrip_Payment_Bucket
GROUP BY INPUT__FILE__NAME
ORDER BY trip_count DESC
LIMIT 1;
```

---

# Part 3 — Partitioning + Bucketing Combined

## 3.1 Table partitioned by `payment_type`, bucketed by `VendorID` into 4 buckets

**Why:** combine both techniques when you have a low-cardinality "coarse" filter column (good for partitioning) and a "fine-grained" column you want evenly distributed within each partition (good for bucketing). Each `payment_type` partition folder will itself contain 4 bucket files split by `VendorID`.

```sql
CREATE TABLE YellowTrip_Payment_Vendor_PB (
    tpep_pickup_datetime   TIMESTAMP,
    tpep_dropoff_datetime  TIMESTAMP,
    passenger_count        INT,
    trip_distance           DOUBLE,
    RatecodeID               INT,
    store_and_fwd_flag       STRING,
    PULocationID              INT,
    DOLocationID              INT,
    VendorID                  INT,
    fare_amount                DOUBLE,
    extra                     DOUBLE,
    mta_tax                   DOUBLE,
    tip_amount                 DOUBLE,
    tolls_amount                DOUBLE,
    improvement_surcharge       DOUBLE,
    total_amount                DOUBLE,
    congestion_surcharge        DOUBLE
)
PARTITIONED BY (payment_type INT)
CLUSTERED BY (VendorID) INTO 4 BUCKETS
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

INSERT OVERWRITE TABLE YellowTrip_Payment_Vendor_PB PARTITION(payment_type)
SELECT tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance,
       RatecodeID, store_and_fwd_flag, PULocationID, DOLocationID, VendorID, fare_amount,
       extra, mta_tax, tip_amount, tolls_amount, improvement_surcharge, total_amount,
       congestion_surcharge, payment_type
FROM ValidYellowTrip;
```

- `VendorID` (the **bucketing** column) stays in the normal column list.
- `payment_type` (the **partitioning** column) is removed from the column list, declared in `PARTITIONED BY`, and placed last in the `SELECT`.

## 3.2 Total trips and total revenue per Vendor, within each payment-type partition

```sql
SELECT payment_type, VendorID, COUNT(*) AS total_trips, SUM(total_amount) AS total_revenue
FROM YellowTrip_Payment_Vendor_PB
GROUP BY payment_type, VendorID;
```

## 3.3 Bucket assignment of each Vendor, verified with `TABLESAMPLE`

```sql
-- Predicted bucket per VendorID
SELECT DISTINCT VendorID, hash(VendorID), pmod(hash(VendorID), 4) AS predicted_bucket
FROM ValidYellowTrip;

-- Verify by sampling bucket 1 of a specific partition
SELECT VendorID, payment_type
FROM YellowTrip_Payment_Vendor_PB
TABLESAMPLE(BUCKET 1 OUT OF 4 ON VendorID)
WHERE payment_type = 1;
```

- `TABLESAMPLE(BUCKET x OUT OF y ON col)` = the `ON col` form explicitly tells Hive which column's hash to sample by — normally optional when it matches the table's `CLUSTERED BY` column, but stating it here makes the check unambiguous.
- Compare the `VendorID` values returned against your `predicted_bucket = 1` rows from the first query — they should match.

## 3.4 Average trip duration and average fare amount, per Vendor within every payment-type partition

```sql
SELECT payment_type, VendorID,
       AVG((UNIX_TIMESTAMP(tpep_dropoff_datetime) - UNIX_TIMESTAMP(tpep_pickup_datetime)) / 60.0) AS avg_duration_minutes,
       AVG(fare_amount) AS avg_fare_amount
FROM YellowTrip_Payment_Vendor_PB
GROUP BY payment_type, VendorID;
```

## 3.5 Highest-revenue payment-type partition, and the top Vendor within it

**Two-step version (simplest to read):**

```sql
-- Step 1: which payment_type partition has the most revenue?
SELECT payment_type, SUM(total_amount) AS total_revenue
FROM YellowTrip_Payment_Vendor_PB
GROUP BY payment_type
ORDER BY total_revenue DESC
LIMIT 1;

-- Step 2: plug that payment_type value in (e.g. 1) to find its top vendor
SELECT VendorID, SUM(total_amount) AS vendor_revenue
FROM YellowTrip_Payment_Vendor_PB
WHERE payment_type = 1
GROUP BY VendorID
ORDER BY vendor_revenue DESC
LIMIT 1;
```

**Single-query version, using window functions:**

```sql
SELECT payment_type, VendorID, vendor_revenue, partition_revenue
FROM (
    SELECT payment_type, VendorID,
           SUM(total_amount) AS vendor_revenue,
           SUM(SUM(total_amount)) OVER (PARTITION BY payment_type) AS partition_revenue,
           RANK() OVER (PARTITION BY payment_type ORDER BY SUM(total_amount) DESC) AS vendor_rank
    FROM YellowTrip_Payment_Vendor_PB
    GROUP BY payment_type, VendorID
) t
WHERE vendor_rank = 1
ORDER BY partition_revenue DESC
LIMIT 1;
```

- `SUM(SUM(total_amount)) OVER (PARTITION BY payment_type)` = a **nested aggregate**: the inner `SUM(total_amount)` gives per-vendor revenue (from the `GROUP BY`), and wrapping it in a window `SUM(...) OVER (PARTITION BY payment_type)` adds those vendor totals back up to the whole partition's revenue — without collapsing the vendor-level rows.

## 3.6 Top 5 pickup locations by revenue, per payment type — verify only relevant partitions are scanned

```sql
SELECT payment_type, PULocationID, total_revenue
FROM (
    SELECT payment_type, PULocationID,
           SUM(total_amount) AS total_revenue,
           RANK() OVER (PARTITION BY payment_type ORDER BY SUM(total_amount) DESC) AS rnk
    FROM YellowTrip_Payment_Vendor_PB
    GROUP BY payment_type, PULocationID
) ranked
WHERE rnk <= 5;
```

**Verify partition pruning** — confirm a filtered query only touches the relevant partition folder instead of the whole table:

```sql
EXPLAIN
SELECT * FROM YellowTrip_Payment_Vendor_PB WHERE payment_type = 1;
```

- `EXPLAIN` = shows Hive's execution plan without actually running the query. Look for a `TableScan` / `Partition Description` section listing **only** `payment_type=1` — if pruning is working, no other partition folder appears anywhere in the plan.

---

## Quick recap

| Concept                                             | Key point                                                                                                                                                                  |
| --------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Partitioning                                        | Splits data into sub-folders by column value — best for low-cardinality filter columns (`payment_type`, `VendorID`)                                                        |
| Dynamic partitioning                                | `PARTITION(col)` with no value + `nonstrict` mode → Hive infers all partition values from the data in one `INSERT`                                                         |
| Partition column position                           | Must be **last** in the `SELECT` list, matching `PARTITIONED BY` order                                                                                                     |
| Bucketing                                           | Splits data into a **fixed number of files** via `hash(col) % buckets` — best for high-cardinality or evenly-distributed columns                                           |
| Bucketing column                                    | Stays in the normal column list — unlike a partition key, it is not pulled out separately                                                                                  |
| `pmod` vs `%`                                       | `pmod` always returns a non-negative remainder and matches Hive's real internal bucket placement; plain `%` can disagree when the hash is negative                         |
| `INPUT__FILE__NAME`                                 | Virtual column returning the physical file a row was read from — one file per bucket, so grouping by it counts rows per bucket in a single query                           |
| `TABLESAMPLE(BUCKET x OUT OF y [ON col])`           | Reads a single bucket file directly — bucketing's main performance win                                                                                                     |
| Partitioning + Bucketing together                   | Partition by the coarse filter column, bucket by the fine-grained column within each partition, for both partition pruning and even file sizes                             |
| `EXPLAIN`                                           | Shows the execution plan — check the `Partition Description` to confirm only the filtered partition(s) are scanned                                                         |
| Window functions (`RANK() OVER (PARTITION BY ...)`) | "Partition" here is a SQL windowing concept for ranking rows within a group — distinct from a table's physical `PARTITIONED BY` clause, even when they share a column name |
