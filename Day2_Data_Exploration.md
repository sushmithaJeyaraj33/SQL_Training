# Day 2 - SQL Data Exploration & Performance Optimization
**Project:** Patent Analytics using PostgreSQL

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

This table serves as the primary staging area for bulk data ingestion and analytics testing.

---

### Screenshot

<img width="627" height="140" alt="Screenshot 2026-07-19 at 5 17 12 PM" src="https://github.com/user-attachments/assets/74682550-4c7e-4ff5-893e-ad1eed782440" />




---

# 2. Load Sample Patent Dataset

Populate the table with **10 million synthetic patent records** 

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
- Randomized date spanning a decade of publication dates.
  

---

### Screenshot

<img width="833" height="316" alt="Screenshot 2026-07-19 at 5 17 42 PM" src="https://github.com/user-attachments/assets/88000d8d-4bbe-498a-acb4-6ad5efce59b2" />



---

# 3. Create Inventor Master

## Objective

Creating a inventor master with inventor_id and name details.

---

## Create Table

```sql
CREATE TABLE patents_1.master_inventors (
    inventor_id   SERIAL PRIMARY KEY,
    inventor_name TEXT UNIQUE NOT NULL
);
```

## Screenshot

<img width="461" height="109" alt="image_3" src="https://github.com/user-attachments/assets/856d5720-7383-45c0-bc2a-cec8a118875a" />


---

## Populate Inventor Master

```sql
INSERT INTO patents_1.master_inventors (inventor_name)
SELECT DISTINCT inventor_name 
FROM patents_1.patents_synthetic_data;
```

## Screenshot

<img width="558" height="94" alt="image_4" src="https://github.com/user-attachments/assets/8f3bf9a9-732d-476c-93be-dae182c6a0b2" />

---

## Verify Structural Schema

```sql
\d patents_1.master_inventors
```

---

### Screenshot

<img width="888" height="154" alt="image_5" src="https://github.com/user-attachments/assets/c6edc0aa-56fa-4573-9502-e87ca8788d09" />


---

# 4. Patent Title Analysis

## Objective

Split patent titles into individual words using spaces and special characters as delimiters.
Ignoring words with 3 or fewer characters.
Convert words to a lower case
Finding the Top 100 most frequently occurring words.
Store the results in a new table.

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

### Screenshot

<img width="507" height="136" alt="image_6" src="https://github.com/user-attachments/assets/a57f511d-0bb6-44f5-b949-5a323864159d" />


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

## Screenshot

<img width="514" height="352" alt="image_7" src="https://github.com/user-attachments/assets/3e8fbfa9-9637-4f9e-8662-e9075f73635a" />


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
### Screenshot

<img width="689" height="220" alt="image_8" src="https://github.com/user-attachments/assets/a3dd8aab-f1e1-4eaf-b5fb-34dd6beb5c51" />


---

## View Top Word Allocations

```sql
SELECT * FROM patents_1.title_word_analysis LIMIT 10;
```

### Screenshot

<img width="634" height="213" alt="image_9" src="https://github.com/user-attachments/assets/bc6ef73e-51ec-4a2f-bb8f-c5b5b1067aef" />


---

### Concepts Covered

- string token splitting via `regexp_split_to_table()`.
- Window functions.

---


# 5. Patent Coverage Analysis

## Objective

Finding all patents whose titles do not contain any of the Top 100 words.
Explaining the SQL approach used.

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

### Concepts Covered

- Correlated Subquery pipelines.
- Negative evaluation exclusions via `NOT EXISTS`.
- Regular expression boundary parsing components using `\y` parameters.

---
### Screenshot

<img width="836" height="436" alt="image_10" src="https://github.com/user-attachments/assets/564e4384-6959-45ae-b350-51726171e641" />


### SQL Approach

NOT EXISTS uses a Correlated Subquery. For every patent row, it checks the 100-word table. The moment it finds even one matching word in the title, it immediately stops looking and discards that patent from the results. It never wastes time checking the remaining 99 words for that row. 

~*: This is PostgreSQL's operator for a case-insensitive regular expression match. 

\y: This represents a word boundary anchor in PostgreSQL regex. By sandwiching the word between them (\yword\y), the engine ensures it only matches the exact, standalone word. It will match "system" or "System", but it will completely ignore partial sub-string matches like "ecosystem" or "microsystems". 

---

# 6. Patent Trend Analysis

## Objective

Finding the year with the highest number of patents.
Displaying the Top 10 years by patent count.
Calculating the year-over-year growth percentage using window functions.

---

## Trend Evaluation Scripts

### Query A: Extract single highest production year
```sql
SELECT
    EXTRACT(YEAR FROM publication_date) AS patent_year,
    COUNT(*) AS patent_count
FROM patents_1.patents_synthetic_data
GROUP BY patent_year
ORDER BY patent_count DESC
LIMIT 1;
```

#### Query A Execution Result
Here is the screenshot confirming the single year with highest patent volume

<img width="498" height="203" alt="image_11" src="https://github.com/user-attachments/assets/1e6f1467-1982-493a-b714-c8d133585cef" />


---

### Query B: Retrieve top 10 historical blocks
```sql
SELECT
    EXTRACT(YEAR FROM publication_date) AS patent_year,
    COUNT(*) AS patent_count
FROM patents_1.patents_synthetic_data
GROUP BY patent_year
ORDER BY patent_count DESC
LIMIT 10;
```

#### Query B Execution Result
Below is the screenshot showing the distribution across the top 10 years

<img width="529" height="315" alt="image_12" src="https://github.com/user-attachments/assets/81194c28-dc09-4e49-92ec-d207a250ece8" />


---

### Query C: Calculate Year-over-Year growth percentages
```sql
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

#### Query C Execution Result
The screenshot below displays the final calculated Year-over-Year percentage trends:

<img width="584" height="670" alt="image_13" src="https://github.com/user-attachments/assets/c1a2ba71-6eec-4a47-9b28-426da2a7e5d8" />


---

### Concepts Covered

- Common Table Expressions (CTEs).
- Analytic `LAG()` window tracking historical evaluations.

---


---

# 7. Optimization

## Objective

Improving query performance using Indexes

---

### 1. Profile Baseline Unindexed Scan
Run a baseline check filtering records down to a single specific year (2023) before optimizing.
```sql
SELECT EXTRACT(YEAR FROM publication_date) AS patent_year, COUNT(*) 
FROM patents_1.patents_synthetic_data
WHERE EXTRACT(YEAR FROM publication_date) = 2023
GROUP BY 1;
```

#### Unindexed Scan Execution Result
Below is the execution output screenshot showing a slow full table sequential scan:

<img width="1125" height="352" alt="image_14" src="https://github.com/user-attachments/assets/23f80714-79e9-44e2-a3ab-d8c67c9a5d68" />


---

### 2. Implement Expression Index Optimization
Construct a functional index on the year expression so the database engine pre-calculates and caches these values.
```sql
CREATE INDEX idx_patents_pub_date_year ON patents_1.patents_synthetic_data ((EXTRACT(YEAR FROM publication_date)));
```

#### Index Creation Success Verification
Below is the terminal verification screenshot showing successful index creation

<img width="952" height="69" alt="image_15" src="https://github.com/user-attachments/assets/aa41d737-7f1d-40d1-ac45-7ca7ad277627" />


---

### 3. Profile Optimized Index Tree Path Scan
Re-run the year-specific query to observe performance improvements with the new index path active.
```sql
SELECT EXTRACT(YEAR FROM publication_date) AS patent_year, COUNT(*) 
FROM patents_1.patents_synthetic_data
WHERE EXTRACT(YEAR FROM publication_date) = 2023
GROUP BY 1;
```

#### Indexed Scan Execution Result


<img width="1112" height="332" alt="image_16" src="https://github.com/user-attachments/assets/4cf4005e-2b47-4ade-9523-70102795d208" />


---

## Profile Baseline Regex Exclusion vs. Full-Text Search GIN Index

### 1. Analyze Performance of Regex Setup

```sql
EXPLAIN ANALYZE
SELECT p.publication_number, p.title
FROM patents_1.patents_synthetic_data p
WHERE p.title IS NOT NULL
  AND NOT EXISTS (
    SELECT 1 
    FROM patents_1.title_word_analysis w
    WHERE p.title ~* ('\y' || w.word || '\y')
  );
```

#### Baseline Regex Query Plan Result

<img width="1187" height="439" alt="image_17" src="https://github.com/user-attachments/assets/c17b1a24-0b5f-400c-b115-32e12d992ad1" />


---

### 2. Construct Generalized Inverted Index (GIN) for Full-Text Search
Build a GIN index on the lowercase `to_tsvector` representation of patent titles to map individual words directly to row identifiers.
```sql
CREATE INDEX idx_lower_title
ON patents_1.patents_synthetic_data
USING gin
(
    to_tsvector('simple', lower(title))
);
```

#### GIN Index Creation Success Verification

<img width="500" height="139" alt="image_18" src="https://github.com/user-attachments/assets/ae8e8dc1-6b0a-43a1-9845-a57e5b740eab" />


---

### 3. Refresh Planner Statistics
Force the database to update its internal structural logs so that the query optimizer knows the new index path exists.
```sql
ANALYZE patents_1.patents_synthetic_data;
```

### 4. Analyze Performance of Optimized Full-Text Search Pipeline
Re-run the session on the rewritten full-text query using the `@@` match operator to observe performance changes.
```sql
EXPLAIN ANALYZE
SELECT p.publication_number, p.title
FROM patents_1.patents_synthetic_data p
WHERE p.title IS NOT NULL
  AND NOT (
    to_tsvector('simple', lower(p.title)) @@ 
    (SELECT to_tsquery('simple', string_agg(word, ' | ')) FROM patents_1.title_word_analysis)
  );
```

#### Optimized GIN Query Plan Result


<img width="1114" height="447" alt="image_19" src="https://github.com/user-attachments/assets/156b2bac-ed5f-4342-ab98-48ebebc327da" />


---






