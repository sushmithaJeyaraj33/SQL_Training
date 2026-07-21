# Day 3 - Complex Data Representation & Analysis
**Project:** Patent Analytics

# 1. Create Mock Patent Dataset

## Objective

Create a synthetic patent dataset containing **1,000,000 patent records** with randomly generated publication numbers, inventor names, publication dates, titles, and abstracts. This dataset serves as the foundation for performing complex data representation and analysis in PostgreSQL.

---

## Create Patent Table

```sql
CREATE TABLE patents_1.patents_mockdata1
(
    publication_number TEXT PRIMARY KEY,
    inventor_name      TEXT,
    publication_date   DATE,
    title              TEXT,
    abstract           TEXT
);
```

### Explanation

This table stores the primary patent information. Each record contains a unique publication number along with inventor details, publication date, patent title, and abstract.

---

### Screenshot

> Insert screenshot here.

---

## Populate Mock Patent Data

```sql
INSERT INTO patents_1.patents_mockdata1
(
    publication_number,
    inventor_name,
    publication_date,
    title,
    abstract
)
SELECT
    'US' || LPAD(gs::text, 10, '0'),

    'Inventor_' || LPAD((1 + floor(random() * 10000))::int::text, 5, '0'),

    DATE '2010-01-01' + (random() * 5843)::int,

    concat_ws(
        ' ',
        adjectives[(random()*19+1)::int],
        technologies[(random()*19+1)::int],
        actions[(random()*19+1)::int],
        'for',
        domains[(random()*19+1)::int],
        'using',
        technologies[(random()*19+1)::int],
        connectors[(random()*19+1)::int],
        features[(random()*19+1)::int],
        architectures[(random()*19+1)::int],
        'with',
        benefits[(random()*19+1)::int],
        qualities[(random()*19+1)::int],
        'performance',
        'and',
        security[(random()*19+1)::int],
        'based',
        'system',
        'architecture',
        versions[(random()*19+1)::int]
    ),

    concat_ws(
        ' ',
        'This invention provides',
        adjectives[(random()*19+1)::int],
        technologies[(random()*19+1)::int],
        'for',
        domains[(random()*19+1)::int],
        'using',
        architectures[(random()*19+1)::int],
        'to improve',
        benefits[(random()*19+1)::int],
        'performance, scalability, availability and security.'
    )

FROM generate_series(1,1000000) gs

CROSS JOIN
(
SELECT

ARRAY[
'Machine','Cloud','Distributed','Neural','Artificial',
'Quantum','Blockchain','Battery','Electric','Medical',
'Image','Wireless','Sensor','Edge','Cyber',
'Speech','Network','Autonomous','Virtual','Digital'
] technologies,

ARRAY[
'System','Method','Framework','Architecture','Platform',
'Engine','Application','Solution','Algorithm','Model',
'Protocol','Mechanism','Workflow','Service','Module',
'Controller','Interface','Process','Device','Analytics'
] actions,

ARRAY[
'Healthcare','Manufacturing','Finance','Retail',
'Agriculture','Education','Automotive','IoT',
'Cloud','Robotics','Satellite','Security',
'Telecommunication','Energy','Payments',
'Logistics','SupplyChain','Medical','SmartCity','Defence'
] domains,

ARRAY[
'advanced','intelligent','scalable','distributed',
'secure','adaptive','dynamic','predictive',
'robust','optimized','highspeed','cloudnative',
'autonomous','wireless','efficient','reliable',
'virtualized','parallel','automated','flexible'
] adjectives,

ARRAY[
'through','via','leveraging','utilizing',
'combining','integrating','supporting','enabling',
'optimizing','processing','executing','monitoring',
'controlling','managing','analyzing','predicting',
'detecting','improving','accelerating','transforming'
] connectors,

ARRAY[
'realtime','streaming','analytics','monitoring',
'optimization','automation','processing','storage',
'routing','prediction','classification','compression',
'authentication','authorization','encryption','synchronization',
'visualization','coordination','diagnostics','recovery'
] features,

ARRAY[
'framework','architecture','pipeline','database',
'cluster','engine','platform','gateway',
'controller','scheduler','processor','interface',
'network','application','repository','cache',
'queue','service','module','workflow'
] architectures,

ARRAY[
'high','better','enhanced','maximum',
'greater','improved','efficient','fast',
'optimized','reliable','stable','consistent',
'predictable','secure','scalable','flexible',
'faulttolerant','continuous','intelligent','dynamic'
] benefits,

ARRAY[
'overall','operational','computational','system',
'business','enterprise','network','application',
'resource','processing','storage','service',
'cloud','database','transaction','runtime',
'deployment','production','execution','workflow'
] qualities,

ARRAY[
'protection','encryption','authentication','validation',
'authorization','monitoring','verification','integrity',
'privacy','compliance','governance','availability',
'confidentiality','resilience','isolation','backup',
'auditing','detection','prevention','recovery'
] security,

ARRAY[
'v1','v2','v3','generation',
'nextgeneration','release','edition','model',
'series','prototype','variant','revision',
'phase1','phase2','phase3','alpha',
'beta','gamma','enterprise','premium'
] versions

) x;
```

### Explanation

The `INSERT` statement generates **1 million synthetic patent records** using `generate_series()`. Arrays containing technologies, domains, actions, and descriptive words are randomly combined with `concat_ws()` to produce realistic patent titles and abstracts, while inventor names and publication dates are randomly generated to simulate production-scale patent data.

---

### Data Generation Highlights

- Generated **1,000,000** synthetic patent records.
- Created unique publication numbers with the **US0000000001** format.
- Randomly assigned **10,000 inventor names**.
- Generated publication dates between **2010 and 2025**.
- Created realistic patent titles using randomly selected technology and domain keywords.
- Generated meaningful patent abstracts for testing full-text search and JSON operations.

---

### Screenshot

> Insert screenshot here.

---

---

## Objective

Create a mapping table to establish a **one-to-many relationship** between patents and inventors. Since the original patent dataset contains only one inventor per patent, this table is created to associate multiple inventors with a single patent, enabling complex ARRAY operations in later tasks.

---

## Create Patent-Inventor Mapping Table

```sql
CREATE TABLE patents_1.patent_inventors_listdata
(
    publication_number TEXT,
    inventor_name      TEXT,
    inventor_order     SMALLINT,
    PRIMARY KEY(publication_number, inventor_order)
);
```

### Explanation

The `patent_inventors_listdata` table stores inventor information separately from the patent table, allowing multiple inventors to be linked to a single patent. The `inventor_order` column preserves the sequence of inventors for each patent and, together with `publication_number`, forms the primary key to ensure uniqueness.

---

### Screenshot

> Insert screenshot here.

---

## Populate Patent-Inventor Mapping

```sql
INSERT INTO patents_1.patent_inventors_listdata
(
    publication_number,
    inventor_name,
    inventor_order
)
SELECT
    p.publication_number,
    x.inventor_name,
    x.rn
FROM patents_1.patents_mockdata1 p
CROSS JOIN LATERAL
(
    SELECT
        inventor_name,
        ROW_NUMBER() OVER () + 1 AS rn
    FROM
    (
        SELECT inventor_name
        FROM patents_1.master_inventors
        WHERE inventor_name <> p.inventor_name
        ORDER BY random()
        LIMIT (floor(random()*4)+1)::int
    ) t
) x;
```

### Explanation

This query assigns **1 to 4 additional random inventors** to every patent using `CROSS JOIN LATERAL`. The lateral join executes the subquery for each patent individually, ensuring that every patent receives a unique set of inventors. The `ROW_NUMBER()` function generates the inventor sequence, while excluding the patent's original inventor avoids duplicate assignments.

---

# 2. Create Patent Inventor ARRAY

## Objective

Represent each patent with **all its associated inventor names stored in a single ARRAY column**. This satisfies the requirement of grouping multiple inventors into one field, making ARRAY-based querying and analysis efficient.

---

## Create Patent Inventor ARRAY Table

```sql
CREATE TABLE patents_1.patent_inventor_array AS

SELECT
    publication_number,
    ARRAY_AGG(inventor_name ORDER BY inventor_name) AS inventor_array
FROM patents_1.patent_inventors_listdata
GROUP BY publication_number;
```

### Explanation

The `ARRAY_AGG()` aggregate function groups all inventor names belonging to the same patent into a PostgreSQL ARRAY. Instead of storing multiple rows for a patent, all associated inventors are stored together in a single column.

For example:

**Before Grouping**

| Publication Number | Inventor Name |
|--------------------|---------------|
| US0000000001 | Inventor_1023 |
| US0000000001 | Inventor_5432 |
| US0000000001 | Inventor_8765 |

**After Grouping**

| Publication Number | Inventor Array |
|--------------------|----------------|
| US0000000001 | {Inventor_1023, Inventor_5432, Inventor_8765} |

---

### Why This Step?

The assignment requires:

> **Create a representation where each patent contains all its associated inventor names together in a single field.**

Using an ARRAY allows PostgreSQL to store multiple inventor names in one column while preserving their relationship with the patent. This representation also enables efficient ARRAY operations such as searching, matching, and comparison in the following tasks.

---

### Advantages of Using ARRAY

- Eliminates duplicate patent rows.
- Stores multiple inventor names in a single column.
- Simplifies one-to-many data representation.
- Supports efficient ARRAY operators like `@>`, `ANY`, and `&&`.
- Improves readability and simplifies JSON document generation.

---



# 2. Find Patents by a Specific Inventor

## SQL

```sql
SELECT *
FROM patents_1.patent_inventor_array
WHERE inventor_array @> ARRAY['Inventor_22454'];

SELECT *
FROM patents_1.patent_inventor_array
WHERE 'Inventor_22454' = ANY(inventor_array);
```

### Concepts Covered

- ARRAY
- @>
- ANY

### Screenshot

> Insert screenshot here.

---

# 3. Find Patents Matching Any Inventor in a List

```sql
SELECT *
FROM patents_1.patent_inventor_array
WHERE inventor_array && ARRAY[
'Inventor_5266',
'Inventor_22454',
'Inventor_10739'
];
```

### Explanation

The overlap operator (`&&`) returns patents containing at least one inventor from the supplied list.

### Screenshot

> Insert screenshot here.

---

# 4. Find Two Patents Sharing Common Inventors

```sql
SELECT
p1.publication_number,
p2.publication_number
FROM patents_1.patent_inventor_array p1
JOIN patents_1.patent_inventor_array p2
ON p1.inventor_array && p2.inventor_array
AND p1.publication_number < p2.publication_number
LIMIT 20;
```

### Screenshot

> Insert screenshot here.

---

# 5. Convert ARRAY back into Rows

```sql
SELECT
publication_number,
UNNEST(inventor_array) AS inventor_name
FROM patents_1.patent_inventor_array;
```

Verify using COUNT() against the original mapping table.

### Screenshot

> Insert screenshot here.

---

# 6. Create Patent Metadata using JSONB

Use `jsonb_build_object()` to store country, category, status, technology and filing date inside a single JSONB column.

Include your SQL from the project here.

### Screenshot

> Insert screenshot here.

---

# 7. Query JSONB Metadata

Examples

- Filter by country
- Filter by technology
- Filter by status

Use `metadata->>'country'`, `metadata->>'technology'`, and `metadata->>'status'`.

### Screenshot

> Insert screenshot here.

---

# 8. Update Individual JSON Attribute

Use `jsonb_set()` to update only the `status` field without replacing the complete JSON document.

### Screenshot

> Insert screenshot here.

---

# 9. Generate Patent Document

Use `json_build_object()` to generate a single JSON document containing:

- Patent Details
- Metadata
- Inventors

### Screenshot

> Insert screenshot here.

---

# 10. Generate Hierarchical JSON

Create nested JSON in the following hierarchy:

Year
- Patents
  - Patent Details
    - Inventors

### Screenshot

> Insert screenshot here.

---

# 11. Performance Optimization

## ARRAY Index

```sql
CREATE INDEX idx_inventors
ON patents_1.patent_inventor_array
USING GIN(inventor_array);
```

Compare `EXPLAIN ANALYZE` before and after indexing.

## JSONB Index

```sql
CREATE INDEX idx_metadata
ON patents_1.patent_metadata
USING GIN(metadata);
```

Again compare execution plans before and after indexing.

### Screenshot

> Insert screenshots here.

---

# 12. Additional PostgreSQL ARRAY Operations

- array_position()
- array_append()
- array_remove()
- Array slicing

### Screenshot

> Insert screenshot here.

---

# 13. Additional PostgreSQL JSONB Operations

- ?
- ?&
- jsonb_pretty()

### Screenshot

> Insert screenshot here.

---

# Performance Comparison

| Feature | Before Index | After Index |
|----------|--------------|-------------|
| ARRAY Search | Sequential Scan | GIN Index Scan |
| JSONB Search | Sequential Scan | GIN Index Scan |

---

# Production Use Cases

## ARRAY

Suitable for storing:
- Multiple inventors
- Tags
- Skills

Not suitable for frequent element updates.

## JSONB

Suitable for:
- Flexible metadata
- API payloads
- Dynamic attributes

Not suitable when strict relational constraints are required.

---

# Technologies Used

- PostgreSQL
- SQL
- ARRAY
- JSONB
- GIN Index
- EXPLAIN ANALYZE

---

# Conclusion

This project demonstrates PostgreSQL's advanced support for complex data types using ARRAY and JSONB. It also shows how GIN indexing significantly improves query performance while enabling flexible storage and efficient querying of semi-structured data.
