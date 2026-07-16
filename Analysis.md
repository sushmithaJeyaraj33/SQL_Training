# PostgreSQL Performance Benchmark Report

## Dataset Information

- **Dataset:** `us_patents.csv`
- **Dataset Size:** ~10 Million Records

---

# Converting Parquet File to CSV

> **Screenshot**

---

# Copy CSV File from Local Machine to Docker Container

```bash
docker cp ~/Downloads/us_patents.csv ubuntu_pg2:/tmp/us_patents.csv
docker exec -it ubuntu_pg2 bash
```

> **Screenshot**

---

# Report 1: Logged vs Unlogged Tables

## Create Logged Table

```sql
CREATE TABLE patents_1.us_patents_logged (
    publication_number TEXT,
    title TEXT,
    abstract TEXT
);
```

> **Screenshot**

<img width="468" height="107" alt="image_3" src="https://github.com/user-attachments/assets/67a8661a-564b-4996-a528-9545ad34de3e" />

---

## Import Data into Logged Table

```sql
COPY patents_1.us_patents_logged
FROM '/tmp/us_patents.csv'
DELIMITER ','
CSV HEADER;
```

> **Screenshot**

---

## Create Unlogged Table

```sql
CREATE UNLOGGED TABLE patents_1.us_patents_unlogged (
    publication_number TEXT,
    title TEXT,
    abstract TEXT
);
```

> **Screenshot**

---

## Import Data into Unlogged Table

```sql
COPY patents_1.us_patents_unlogged
FROM '/tmp/us_patents.csv'
DELIMITER ','
CSV HEADER;
```

> **Screenshot**

---

## Measure Logged Table Size

```sql
SELECT pg_size_pretty(
       pg_total_relation_size('patents_1.us_patents_logged'));
```

> **Screenshot**

---

## Measure Unlogged Table Size

```sql
SELECT pg_size_pretty(
       pg_total_relation_size('patents_1.us_patents_unlogged'));
```

> **Screenshot**

---

## Primary Key Lookup (Unlogged)

```sql
EXPLAIN ANALYZE
SELECT *
FROM patents_1.us_patents_unlogged
WHERE publication_number='US-4081864-A';
```

> **Screenshot**

---

## Primary Key Lookup (Logged)

```sql
EXPLAIN ANALYZE
SELECT *
FROM patents_1.us_patents_logged
WHERE publication_number='US-4081864-A';
```

> **Screenshot**

---

## UPPER() Function (Unlogged)

```sql
EXPLAIN ANALYZE
SELECT UPPER(title) AS new_title
FROM patents_1.us_patents_unlogged;
```

> **Screenshot**

---

## UPPER() Function (Logged)

```sql
EXPLAIN ANALYZE
SELECT UPPER(title) AS new_title
FROM patents_1.us_patents_logged;
```

> **Screenshot**

---

## String Concatenation (Unlogged)

```sql
EXPLAIN ANALYZE
SELECT publication_number || ' - ' || title AS patent_identity
FROM patents_1.us_patents_unlogged;
```

> **Screenshot**

---

## String Concatenation (Logged)

```sql
EXPLAIN ANALYZE
SELECT publication_number || ' - ' || title AS patent_identity
FROM patents_1.us_patents_logged;
```

> **Screenshot**

---

## Performance Comparison

| Comparison Parameter | LOGGED Table | UNLOGGED Table |
|----------------------|-------------|---------------|
| Number of Records Imported | 9,458,171 | 9,458,171 |
| Table Creation Time | 19.147 ms | 27.167 ms |
| Import Time | 1.28 s | 1.15 s |
| Count Retrieval | 19 s | 23 s |
| Table Size | 8558 MB | 8557 MB |
| Index Size | 0 MB | 0 MB |
| Primary Lookup | Planning: 2.74 ms<br>Execution: 11443.09 ms | Planning: 4.58 ms<br>Execution: 8942.17 ms |
| UPPER() Function | Planning: 0.52 ms<br>Execution:10870 ms | Planning:0.92 ms<br>Execution:10719 ms |
| String Concatenation | Planning:0.339 ms<br>Execution:7491 ms | Planning:0.382 ms<br>Execution:8885 ms |

---

# Report 2: Without Index

## Create Table

```sql
CREATE TABLE patents_1.patents_no_index (
    publication_number TEXT,
    title TEXT,
    abstract TEXT
);
```

> **Screenshot**

---

## Verify No Index Exists

```sql
SELECT indexname
FROM pg_indexes
WHERE tablename='patents_no_index';
```

> **Screenshot**

---

## Import Data

```sql
COPY patents_1.patents_no_index
FROM '/tmp/us_patents.csv'
DELIMITER ','
CSV HEADER;
```

> **Screenshot**

---

## Record Count

```sql
SELECT COUNT(*)
FROM patents_1.patents_no_index;
```

> **Screenshot**

---

## Table Size

```sql
SELECT pg_size_pretty(
pg_relation_size('patents_1.patents_no_index'));
```

> **Screenshot**

---

## Index Size

```sql
SELECT pg_size_pretty(
pg_indexes_size('patents_1.patents_no_index'));
```

> **Screenshot**

---

## Primary Lookup

```sql
EXPLAIN ANALYZE
SELECT *
FROM patents_1.patents_no_index
WHERE publication_number='US-4081864-A';
```

> **Screenshot**

---

## UPPER()

```sql
EXPLAIN ANALYZE
SELECT UPPER(title)
FROM patents_1.patents_no_index;
```

> **Screenshot**

---

## Concatenation

```sql
EXPLAIN ANALYZE
SELECT publication_number || ' - ' || title
FROM patents_1.patents_no_index;
```

> **Screenshot**

---

# Report 3: With B-Tree Index

## Create Table

```sql
CREATE TABLE patents_1.patents_with_index (
    publication_number TEXT,
    title TEXT,
    abstract TEXT
);
```

> **Screenshot**

---

## Create Index

```sql
CREATE INDEX idx_patents_btree
ON patents_1.patents_with_index(publication_number);
```

> **Screenshot**

---

## Verify Index

```sql
SELECT indexname
FROM pg_indexes
WHERE tablename='patents_with_index';
```

> **Screenshot**

---

## Import Data

```sql
COPY patents_1.patents_with_index
FROM '/tmp/us_patents.csv'
DELIMITER ','
CSV HEADER;
```

> **Screenshot**

---

## Record Count

```sql
SELECT COUNT(*)
FROM patents_1.patents_with_index;
```

> **Screenshot**

---

## Table Size

```sql
SELECT pg_size_pretty(
pg_relation_size('patents_1.patents_with_index'));
```

> **Screenshot**

---

## Index Size

```sql
SELECT pg_size_pretty(
pg_indexes_size('patents_1.patents_with_index'));
```

> **Screenshot**

---

## Primary Lookup

```sql
EXPLAIN ANALYZE
SELECT *
FROM patents_1.patents_with_index
WHERE publication_number='US-4081864-A';
```

> **Screenshot**

---

## UPPER()

```sql
EXPLAIN ANALYZE
SELECT UPPER(title)
FROM patents_1.patents_with_index;
```

> **Screenshot**

---

## Concatenation

```sql
EXPLAIN ANALYZE
SELECT publication_number || ' - ' || title
FROM patents_1.patents_with_index;
```

> **Screenshot**

---

# Report 4: UNIQUE MD5 Index

## Create Table

```sql
CREATE TABLE patents_1.patents_md5 (
    publication_number TEXT,
    title TEXT,
    abstract TEXT
);
```

> **Screenshot**

---

## Create UNIQUE MD5 Index

```sql
CREATE UNIQUE INDEX idx_patents_md5
ON patents_1.patents_md5
(MD5(publication_number || title || abstract));
```

> **Screenshot**

---

## Verify Index

```sql
SELECT schemaname,
       tablename,
       indexname
FROM pg_indexes
WHERE schemaname='patents_1'
AND tablename='patents_md5';
```

> **Screenshot**

---

## Import Data

```sql
COPY patents_1.patents_md5
FROM '/tmp/us_patents.csv'
DELIMITER ','
CSV HEADER;
```

> **Screenshot**

---

## Record Count

```sql
SELECT COUNT(*)
FROM patents_1.patents_md5;
```

> **Screenshot**

---

## Table Size

```sql
SELECT pg_size_pretty(
pg_relation_size('patents_1.patents_md5'));
```

> **Screenshot**

---

## Index Size

```sql
SELECT pg_size_pretty(
pg_indexes_size('patents_1.patents_md5'));
```

> **Screenshot**

---

## Duplicate Prevention Test

```sql
INSERT INTO patents_1.patents_md5
SELECT *
FROM patents_1.patents_md5
LIMIT 1;
```

> **Screenshot**

---

## Index Performance Comparison

| Parameter | Without Index | B-Tree Index | UNIQUE MD5 |
|------------|--------------|-------------|------------|
| Records | 9,458,171 | 9,458,171 | 9,458,171 |
| Table Creation | 76.848 ms | 51.610 ms | 27.167 ms |
| Import Time | 1.48 min | 2 min | 3.34 min |
| Count Retrieval | 11 s | 13 s | 14 s |
| Table Size | 8553 MB | 8553 MB | 8553 MB |
| Index Size | 0 MB | 546 MB | 690 MB |
| Primary Lookup | 7684.80 ms | 3.89 ms | Duplicate validation successful |
| UPPER() | 11020 ms | 12598 ms | - |
| Concatenation | 6863 ms | 7310 ms | - |

---

# Report 5: Partitioned Table Analysis

## Create Parent Table

```sql
CREATE TABLE patents_1.patents_partitioned(
    publication_number TEXT,
    title TEXT,
    abstract TEXT
)
PARTITION BY HASH(publication_number);
```

> **Screenshot**

---

## Create Child Partition

```sql
CREATE TABLE patents_1.patents_p0
PARTITION OF patents_1.patents_partitioned
FOR VALUES WITH (MODULUS 5, REMAINDER 0);
```

> **Screenshot**

---

## Verify Partitions

```sql
SELECT inhrelid::regclass AS partition_name
FROM pg_inherits
WHERE inhparent='patents_1.patents_partitioned'::regclass;
```

> **Screenshot**

---

## Import Data

```sql
COPY patents_1.patents_partitioned
FROM '/tmp/us_patents.csv'
DELIMITER ','
CSV HEADER;
```

> **Screenshot**

---

## CPU Usage During Import

> **Screenshot**

---

## Total Record Count

```sql
SELECT COUNT(*)
FROM patents_1.patents_partitioned;
```

> **Screenshot**

---

## Records Per Partition

```sql
SELECT tableoid::regclass AS partition_name,
COUNT(*) AS records
FROM patents_1.patents_partitioned
GROUP BY tableoid
ORDER BY partition_name;
```

> **Screenshot**

---

## Parent Table Size

```sql
SELECT pg_size_pretty(
pg_relation_size('patents_1.patents_partitioned'));
```

> **Screenshot**

---

## Index Size

```sql
SELECT pg_size_pretty(
pg_indexes_size('patents_1.patents_partitioned'));
```

> **Screenshot**

---

## Query 1 - Primary Lookup

```sql
EXPLAIN ANALYZE
SELECT *
FROM patents_1.patents_partitioned
WHERE publication_number='US-4081864-A';
```

> **Screenshot**

---

## Query 2 - UPPER()

```sql
EXPLAIN ANALYZE
SELECT UPPER(title)
FROM patents_1.patents_partitioned;
```

> **Screenshot**

---

## Query 3 - Concatenation

```sql
EXPLAIN ANALYZE
SELECT publication_number || ' - ' || title
FROM patents_1.patents_partitioned;
```

> **Screenshot**

---

## Normal vs Partitioned Table Comparison

| Parameter | Normal Table | Partitioned Table |
|------------|-------------|------------------|
| Records Imported | 9,458,171 | 9,458,171 |
| Table Creation | 19.147 ms | Parent:47.70 ms<br>Child:15 ms |
| Import Time | 1.28 min | 1.43 min |
| Count Retrieval | 19 s | 21 s |
| Table Size | 8558 MB | Parent:0 MB |
| Index Size | 0 MB | 0 MB |
| Primary Lookup | Planning:2.74 ms<br>Execution:11443.09 ms | Planning:0.852 ms<br>Execution:1567.89 ms |
| UPPER() | Planning:0.52 ms<br>Execution:10870 ms | Planning:0.76 ms<br>Execution:20666 ms |
| Concatenation | Planning:0.339 ms<br>Execution:7491 ms | Planning:6.120 ms<br>Execution:20701 ms |

---

# Conclusion

For datasets exceeding **10 million records**, the recommended workflow is:

1. Load incoming data into an **UNLOGGED staging table** for faster bulk imports.
2. Move validated data into a **LOGGED production table** for durability.
3. Use a **UNIQUE MD5 Index** to prevent duplicate records.
4. Create **B-Tree indexes** on frequently searched columns.
5. Use **Partitioned Tables** to improve lookup performance, simplify maintenance, and support scalability.
