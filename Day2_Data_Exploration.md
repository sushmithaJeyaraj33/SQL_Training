# Day 2 - SQL Data Exploration & Performance Optimization
**Project:** Patent Analytics using PostgreSQL

---

# Dataset Information

| Property | Value |
|----------|-------|
| Database | PostgreSQL |
| Schema | patents_1 |
| Table | patents_synthetic_data |
| Records | 10,000,000 |
| Purpose | SQL Ingestion, Word Parsing & Performance Optimization |

---

# 1. Create Patent Synthetic Data Table

## Table Creation

```sql
DROP TABLE IF EXISTS patents_1.patents_synthetic_data;

CREATE TABLE patents_1.patents_synthetic_data (
    publication_number TEXT,
    inventor_name      TEXT,
    publication_date   DATE,
    title              TEXT,
    abstract           TEXT
);
```

---

### Purpose

This table serves as the primary staging area for bulk data ingestion and analytics testing without initial index overhead constraints.

---

### Screenshot

![Step 1 - Table Creation Screenshot Placeholder](https://placehold.co)

---

# 2. Load Sample Patent Dataset

Populate the table with **10 million synthetic patent records** dynamically from unlogged source configurations.

```sql
INSERT INTO patents_1.patents_synthetic_data 
SELECT 
    src.publication_number,
    'Inventor_' || floor(random() * 30000 + 1)::int AS inventor_name,
    CURRENT_DATE - (random() * 3650)::int AS publication_date,
    src.title,
    src.abstract
FROM patents_1.us_patents_unlogged src;

ALTER TABLE patents_1.patents_synthetic_data ADD PRIMARY KEY (publication_number);
```

---

### Data Generation Highlights

- Unique, indexed primary key constraints on publication numbers.
- Automated generation of 30,000 unique inventor entries.
- Randomized sliding chronological window spanning a decade of publication dates.
- Direct extraction of original unstructured source title and abstract configurations.

---

### Screenshot

![Step 2 - Bulk Data Loading Screenshot Placeholder](https://placehold.co)

---

# 3. Create Inventor Master

## Objective

Normalize inventor profiles into a structured lookup framework to eliminate structural duplicate fields.

---

## Create Table

```sql
CREATE TABLE patents_1.master_inventors (
    inventor_id   SERIAL PRIMARY KEY,
    inventor_name TEXT UNIQUE NOT NULL
);
```

---

## Populate Inventor Master

```sql
INSERT INTO patents_1.master_inventors (inventor_name)
SELECT DISTINCT inventor_name 
FROM patents_1.patents_synthetic_data;
```

---

## Verify Structural Schema

```sql
\d patents_1.master_inventors
```

---

### Expected Result

- Redundant metadata components eliminated from the environment.
- Standardized 4-byte serial identifiers instantiated safely to limit disk footprints.
- Direct unique constraints built automatically to guarantee production consistency.

---

### Screenshot

![Step 3 - Inventor Master Verification Screenshot Placeholder](https://placehold.co+(\d))

---

# 4. Patent Title Analysis

## Objective

Deconstruct text titles down to unique words, filtering structural tokens to establish a top 100 word frequency table.

---

## Create Word Frequency Table

```sql
CREATE TABLE patents_1.title_word_analysis
(
    word TEXT PRIMARY KEY,
    frequency BIGINT,
    rank INT
);
```

---

## Test & Execute Word Extraction

```sql
SELECT
    LOWER(word) AS word,
    COUNT(*) AS frequency,
    RANK() OVER (ORDER BY COUNT(*) DESC) AS rank
FROM patents_1.patents_synthetic_data,
LATERAL regexp_split_to_table(title,'[^A-Za-z0-9]+') AS word
WHERE LENGTH(word) > 3
GROUP BY LOWER(word)
ORDER BY frequency DESC
LIMIT 100;
```

```sql
INSERT INTO patents_1.title_word_analysis (word, frequency, rank)
SELECT
    LOWER(word) AS word,
    COUNT(*) AS frequency,
    RANK() OVER (ORDER BY COUNT(*) DESC) AS rank
FROM patents_1.patents_synthetic_data,
LATERAL regexp_split_to_table(title,'[^A-Za-z0-9]+') AS word
WHERE LENGTH(word) > 3
GROUP BY LOWER(word)
ORDER BY frequency DESC
LIMIT 100;
```

---

## View Top Word Allocations

```sql
SELECT * FROM patents_1.title_word_analysis LIMIT 10;
```

---

### Concepts Covered

- Dynamic string token splitting via `regexp_split_to_table()`.
- Lateral block referencing configurations.
- Window functions alongside case-insensitive string parsing engines.

---

### Screenshot

![Step 4 - Word Tokenization Test Query Screenshot Placeholder](https://placehold.co)

---

# 5. Patent Coverage Analysis

## Objective

Isolate unmapped documents whose title fields completely avoid containing any captured entries from the top 100 frequency index.

---

## Baseline Text Exclusion

```sql
SELECT p.publication_number, p.title
FROM patents_1.patents_synthetic_data p
WHERE p.title IS NOT NULL
  AND NOT EXISTS (
    SELECT 1 
    FROM patents_1.title_word_analysis w
    WHERE p.title ~* ('\y' || w.word || '\y')
  );
```

---

### Concepts Covered

- Correlated Subquery pipelines.
- Negative evaluation exclusions via `NOT EXISTS`.
- Regular expression boundary parsing components using `\y` parameters.

---

### Screenshot

![Step 5 - Baseline Exclusion Query Screenshot Placeholder](https://placehold.co)

---

# 6. Patent Trend Analysis

## Objective

Measure historical data generation density metrics and calculate Year-over-Year volume change vectors.

---

## Core Trend Evaluation Scripts

```sql
-- Query A: Extract single highest production year
SELECT
    EXTRACT(YEAR FROM publication_date) AS patent_year,
    COUNT(*) AS patent_count
FROM patents_1.patents_synthetic_data
GROUP BY patent_year
ORDER BY patent_count DESC
LIMIT 1;

-- Query B: Retrieve top 10 historical blocks
SELECT
    EXTRACT(YEAR FROM publication_date) AS patent_year,
    COUNT(*) AS patent_count
FROM patents_1.patents_synthetic_data
GROUP BY patent_year
ORDER BY patent_count DESC
LIMIT 10;

-- Query C: Calculate Year-over-Year growth percentages
WITH yearly_patents AS
(
    SELECT
        EXTRACT(YEAR FROM publication_date) AS patent_year,
        COUNT(*) AS patent_count
    FROM patents_1.patents_synthetic_data
    GROUP BY patent_year
)
SELECT
    patent_year,
    patent_count,
    LAG(patent_count) OVER (ORDER BY patent_year) AS previous_year_count,
    ROUND(
        (patent_count - LAG(patent_count) OVER (ORDER BY patent_year)) * 100.0 /
        LAG(patent_count) OVER (ORDER BY patent_year), 2
    ) AS growth_percentage
FROM yearly_patents
ORDER BY patent_year;
```

---

### Concepts Covered

- Date extraction and mathematical grouping metrics.
- Common Table Expressions (CTEs) for staging linear sub-allocations.
- Analytic `LAG()` window tracking mechanics for historical evaluations.

---

### Screenshot

![Step 6 - Growth Analytics Output Screenshot Placeholder](https://placehold.co)

---

# 7. Performance Optimization Tuning

## Objective

Profile slow full table execution plans using `EXPLAIN ANALYZE` and apply specialized indexes to shift retrieval costs downward.

---

## Profile Unindexed vs. Expression Index Filtering

```sql
-- Profile Baseline Unindexed Scan
SELECT EXTRACT(YEAR FROM publication_date) AS patent_year, COUNT(*) 
FROM patents_1.patents_synthetic_data
WHERE EXTRACT(YEAR FROM publication_date) = 2023
GROUP BY 1;

-- Implement Expression Index Optimization
CREATE INDEX idx_patents_pub_date_year ON patents_1.patents_synthetic_data ((EXTRACT(YEAR FROM publication_date)));

-- Profile Optimized Index Tree Path Scan
SELECT EXTRACT(YEAR FROM publication_date) AS patent_year, COUNT(*) 
FROM patents_1.patents_synthetic_data
WHERE EXTRACT(YEAR FROM publication_date) = 2023
GROUP BY 1;
```

---

## Profile Baseline Regex Exclusion vs. Full-Text Search GIN Index

```sql
-- Analyze Performance of Baseline Regex Setup
EXPLAIN ANALYZE
SELECT p.publication_number, p.title
FROM patents_1.patents_synthetic_data p
WHERE p.title IS NOT NULL
  AND NOT EXISTS (
    SELECT 1 
    FROM patents_1.title_word_analysis w
    WHERE p.title ~* ('\y' || w.word || '\y')
  );

-- Construct Generalized Inverted Index (GIN) for Full-Text Search
CREATE INDEX idx_lower_title
ON patents_1.patents_synthetic_data
USING gin
(
    to_tsvector('simple', lower(title))
);

-- Refresh Planner Statistics
ANALYZE patents_1.patents_synthetic_data;

-- Analyze Performance of Optimized Full-Text Search Pipeline
EXPLAIN ANALYZE
SELECT p.publication_number, p.title
FROM patents_1.patents_synthetic_data p
WHERE p.title IS NOT NULL
  AND NOT (
    to_tsvector('simple', lower(p.title)) @@ 
    (SELECT to_tsquery('simple', string_agg(word, ' | ')) FROM patents_1.title_word_analysis)
  );
```

---

### Optimization Takeaways

- **Algorithmic Transformation**: Converting a `Nested Loop Anti Join` into an isolated `InitPlan` cluster eliminates row-by-row regex comparisons.
- **Planner Cost Reductions**: Cost estimates fall significantly from **16,749,511.03** down to **3,079,449.34** following GIN tuning steps.
- **Hardware Acceleration**: GIN indexing converts heavy sentence string comparisons into sub-second token matching lookups.

---

### Benchmarking Screenshots

![Step 7A - Baseline Regex Plan Screenshot Placeholder](https://placehold.co)

