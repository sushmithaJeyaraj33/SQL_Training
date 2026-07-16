# PostgreSQL Performance Benchmarking on 10 Million Patent Records

## Project Overview

This project benchmarks different PostgreSQL techniques for loading and querying a large dataset containing approximately **9.46 million US patent records**. The objective is to compare various storage and indexing strategies and identify the most efficient approach for handling large-scale datasets.

---

## Dataset

- **Dataset:** us_patents.csv
- **Total Records:** 9,458,171
- **Source Format:** CSV (converted from Parquet)
- **Database:** PostgreSQL
- **Environment:** Docker Container

---

# Project Objectives

- Compare **LOGGED vs UNLOGGED** tables
- Compare tables **With Index vs Without Index**
- Evaluate **UNIQUE MD5 Functional Index**
- Analyze **Hash Partitioned Tables**
- Measure:
  - Table Creation Time
  - Import Time
  - Query Performance
  - Table Size
  - Index Size
  - Duplicate Prevention

---

# Environment Setup

## Copy dataset into Docker

```bash
docker cp ~/Downloads/us_patents.csv ubuntu_pg2:/tmp/us_patents.csv
```

## Open PostgreSQL container

```bash
docker exec -it ubuntu_pg2 bash
```

---

# Experiment 1 – LOGGED vs UNLOGGED Tables

## Logged Table

- Durable storage
- WAL enabled
- Crash-safe

## Unlogged Table

- Faster data loading
- No WAL logging
- Data lost after crash

### Performance Summary

| Parameter | Logged | Unlogged |
|-----------|---------|-----------|
| Records Imported | 9,458,171 | 9,458,171 |
| Table Creation | 19.147 ms | 27.167 ms |
| Import Time | 1.28 min | 1.15 min |
| Count Retrieval | 19 s | 23 s |
| Table Size | 8558 MB | 8557 MB |
| Index Size | 0 MB | 0 MB |

### Query Performance

#### Primary Lookup

| Logged | Unlogged |
|---------|-----------|
| Planning: 2.74 ms | Planning: 4.58 ms |
| Execution: 11443.09 ms | Execution: 8942.17 ms |

#### UPPER() Function

| Logged | Unlogged |
|---------|-----------|
| 10870 ms | 10719 ms |

#### Concatenation (||)

| Logged | Unlogged |
|---------|-----------|
| 7491 ms | 8885 ms |

---

# Experiment 2 – Without Index vs B-tree Index vs UNIQUE MD5 Index

## Without Index

- Fastest bulk loading
- Slow record lookup

## B-tree Index

- Faster searches
- Slightly slower imports

## UNIQUE MD5 Functional Index

- Prevents duplicate records
- Stores MD5 hash of combined columns
- Slightly larger index
- Slowest import due to uniqueness validation

### Performance Comparison

| Parameter | Without Index | B-tree Index | UNIQUE MD5 |
|-----------|--------------|--------------|-------------|
| Records Imported | 9,458,171 | 9,458,171 | 9,458,171 |
| Import Time | 1.48 min | 2 min | 3.34 min |
| Table Size | 8553 MB | 8553 MB | 8553 MB |
| Index Size | 0 MB | 546 MB | 690 MB |

### Primary Lookup

| Without Index | B-tree |
|---------------|---------|
| 7684.80 ms | 3.89 ms |

### Duplicate Validation

✅ Supported using UNIQUE MD5 Index

---

# Experiment 3 – Hash Partitioned Table

## Objective

Evaluate PostgreSQL hash partitioning for large datasets.

### Partition Key

```
publication_number
```

### Performance

| Parameter | Normal Table | Partitioned Table |
|-----------|--------------|-------------------|
| Records Imported | 9,458,171 | 9,458,171 |
| Import Time | 1.28 min | 1.43 min |
| Count Retrieval | 19 s | 21 s |
| Parent Table Size | 8558 MB | 0 MB |
| Index Size | 0 MB | 0 MB |

### Primary Lookup

| Normal | Partitioned |
|---------|-------------|
| 11443.09 ms | 1567.89 ms |

### UPPER()

| Normal | Partitioned |
|---------|-------------|
| 10870 ms | 20666 ms |

### Concatenation

| Normal | Partitioned |
|---------|-------------|
| 7491 ms | 20701 ms |

---

# Key Observations

## LOGGED vs UNLOGGED

### Advantages of LOGGED

- Crash recovery
- Durable storage
- Better production usage

### Advantages of UNLOGGED

- Faster bulk imports
- Lower write overhead

---

## B-tree Index

### Advantages

- Extremely fast lookups
- Ideal for WHERE clauses
- Best choice for search queries

### Disadvantages

- Slightly slower imports
- Additional storage

---

## UNIQUE MD5 Index

### Advantages

- Prevents duplicate records
- Ensures data integrity
- Useful for ETL pipelines

### Disadvantages

- Slowest import
- Larger index size

---

## Partitioning

### Advantages

- Much faster point lookups
- Better scalability
- Easier maintenance

### Disadvantages

- Slower sequential scans
- Slightly slower imports

---

# Recommended Architecture

For datasets exceeding **10 million records**, the recommended workflow is:

1. Load incoming data into an **UNLOGGED staging table** for maximum loading speed.
2. Validate and move cleaned data into a **LOGGED production table** for durability.
3. Use a **UNIQUE MD5 functional index** to prevent duplicate records.
4. Create **B-tree indexes** on frequently searched columns.
5. Partition large production tables to improve scalability and point lookup performance.

---

# Technologies Used

- PostgreSQL
- Docker
- SQL
- CSV Dataset
- Hash Partitioning
- B-tree Index
- Functional Index (MD5)

---

# Conclusion

This benchmark demonstrates that different PostgreSQL features serve different purposes:

- **UNLOGGED tables** provide the fastest bulk data loading.
- **LOGGED tables** ensure data durability and recovery.
- **B-tree indexes** dramatically improve lookup performance.
- **UNIQUE MD5 indexes** effectively prevent duplicate data.
- **Hash partitioning** significantly improves point lookup performance for very large datasets.

Choosing the right combination of these features depends on the workload, balancing import speed, query performance, storage, and data integrity.
