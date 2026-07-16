# PostgreSQL Performance Benchmark Report

## Dataset Information

- **Dataset:** `us_patents.csv`
- **Dataset Size:** ~10 Million Records

---

# Converting Parquet File to CSV

<img width="936" height="61" alt="image_1" src="https://github.com/user-attachments/assets/49aea970-3931-479c-9799-4f225908a7bf" />


---

# Copy CSV File from Local Machine to Docker Container

```bash
docker cp ~/Downloads/us_patents.csv ubuntu_pg2:/tmp/us_patents.csv
docker exec -it ubuntu_pg2 bash
```


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

> <img width="718" height="151" alt="image_4" src="https://github.com/user-attachments/assets/019e5e7b-76ad-46cd-90ab-e68b559c748d" />


---

## Create Unlogged Table

```sql
CREATE UNLOGGED TABLE patents_1.us_patents_unlogged (
    publication_number TEXT,
    title TEXT,
    abstract TEXT
);
```

> <img width="558" height="104" alt="image_5" src="https://github.com/user-attachments/assets/428dec4b-2d94-4e10-b59f-78a4cd04445d" />


---

## Import Data into Unlogged Table

```sql
COPY patents_1.us_patents_unlogged
FROM '/tmp/us_patents.csv'
DELIMITER ','
CSV HEADER;
```

> <img width="754" height="62" alt="image_6" src="https://github.com/user-attachments/assets/252c71fd-7ac9-4135-b95e-f74150e9e0fa" />


---

## Measure Logged Table Size

```sql
SELECT pg_size_pretty(
       pg_total_relation_size('patents_1.us_patents_logged'));
```

> <img width="709" height="214" alt="image_8" src="https://github.com/user-attachments/assets/e06d3332-76ec-49a0-80b9-61b13db73b1c" />


---

## Measure Unlogged Table Size

```sql
SELECT pg_size_pretty(
       pg_total_relation_size('patents_1.us_patents_unlogged'));
```

> <img width="780" height="210" alt="image_9" src="https://github.com/user-attachments/assets/7380701f-d739-4885-9776-f1a110b65cdf" />


---

## Primary Key Lookup (Unlogged)

```sql
EXPLAIN ANALYZE
SELECT *
FROM patents_1.us_patents_unlogged
WHERE publication_number='US-4081864-A';
```

> <img width="1012" height="275" alt="image_10" src="https://github.com/user-attachments/assets/b21030f9-a5e6-4476-b9eb-fe7e3707586b" />


---

## Primary Key Lookup (Logged)

```sql
EXPLAIN ANALYZE
SELECT *
FROM patents_1.us_patents_logged
WHERE publication_number='US-4081864-A';
```

> <img width="1045" height="278" alt="image_11" src="https://github.com/user-attachments/assets/ad2ff1e9-8263-4e7a-9f74-b90d7d050718" />


---

## UPPER() Function (Unlogged)

```sql
EXPLAIN ANALYZE
SELECT UPPER(title) AS new_title
FROM patents_1.us_patents_unlogged;
```

> <img width="1044" height="198" alt="image_12" src="https://github.com/user-attachments/assets/81c1d1b6-359e-4786-9ed1-1a188978e753" />


---

## UPPER() Function (Logged)

```sql
EXPLAIN ANALYZE
SELECT UPPER(title) AS new_title
FROM patents_1.us_patents_logged;
```

> <img width="992" height="201" alt="image_13" src="https://github.com/user-attachments/assets/1680c962-5183-4e5e-84b0-25bddf4d1c5f" />


---

## String Concatenation (Unlogged)

```sql
EXPLAIN ANALYZE
SELECT publication_number || ' - ' || title AS patent_identity
FROM patents_1.us_patents_unlogged;
```

> <img width="973" height="202" alt="image_14" src="https://github.com/user-attachments/assets/89b5b05a-9f81-4918-b7f1-726ec5ec0cfe" />


---

## String Concatenation (Logged)

```sql
EXPLAIN ANALYZE
SELECT publication_number || ' - ' || title AS patent_identity
FROM patents_1.us_patents_logged;
```

> <img width="2000" height="400" alt="image_15" src="https://github.com/user-attachments/assets/c1b87602-ffc6-4fab-ab1b-64f72cbd7169" />


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

> <img width="446" height="120" alt="image_16" src="https://github.com/user-attachments/assets/e29047f0-3302-453d-87e6-236f12d4dba6" />


---

## Verify No Index Exists

```sql
SELECT indexname
FROM pg_indexes
WHERE tablename='patents_no_index';
```

> <img width="742" height="102" alt="image_17" src="https://github.com/user-attachments/assets/20f5b8e9-a158-4305-b23c-0ffa4156fcd4" />


---

## Import Data

```sql
COPY patents_1.patents_no_index
FROM '/tmp/us_patents.csv'
DELIMITER ','
CSV HEADER;
```

> <img width="766" height="79" alt="image_18" src="https://github.com/user-attachments/assets/2af991bd-ab0c-46a1-ad1b-97a7268376be" />


---

## Record Count

```sql
SELECT COUNT(*)
FROM patents_1.patents_no_index;
```

> <img width="502" height="109" alt="image_19" src="https://github.com/user-attachments/assets/90a14647-3a40-4410-b6f2-28b27815c28b" />


---

## Table Size

```sql
SELECT pg_size_pretty(
pg_relation_size('patents_1.patents_no_index'));
```

> <img width="714" height="108" alt="image_20" src="https://github.com/user-attachments/assets/4929f901-32ef-41bb-ad82-94c51590bbfe" />


---

## Index Size

```sql
SELECT pg_size_pretty(
pg_indexes_size('patents_1.patents_no_index'));
```

> <img width="664" height="108" alt="image_21" src="https://github.com/user-attachments/assets/54c03649-7bda-4222-9651-d9d3e24b9ce2" />


---

## Primary Lookup

```sql
EXPLAIN ANALYZE
SELECT *
FROM patents_1.patents_no_index
WHERE publication_number='US-4081864-A';
```

> <img width="1059" height="276" alt="image_22" src="https://github.com/user-attachments/assets/76ed1284-1626-4348-a616-ea1aa988689c" />


---

## UPPER()

```sql
EXPLAIN ANALYZE
SELECT UPPER(title)
FROM patents_1.patents_no_index;
```

> <img width="1011" height="204" alt="image_23" src="https://github.com/user-attachments/assets/6ebcbe90-b4bc-434e-9105-fd1a2bdc2351" />


---

## Concatenation

```sql
EXPLAIN ANALYZE
SELECT publication_number || ' - ' || title
FROM patents_1.patents_no_index;
```

> <img width="966" height="198" alt="image_24" src="https://github.com/user-attachments/assets/66d93c8b-3b07-4593-bb41-c172f7fd36d2" />


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
