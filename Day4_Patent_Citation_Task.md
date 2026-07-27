# Day 4 – Patent Citation Task for

## Objective

The objective of this assignment is to build and analyze a patent citation system using the existing patent dataset. The implementation covers recursive queries, database functions, views, materialized views, and performance analysis on approximately **1 million patent records**.

---

# 1. Create Patent Citation Table

## Description

A new citation table is created to store relationships between patents. Each record represents one citation where a newer patent references an older patent.

### SQL Query

```sql
CREATE TABLE patent_citations (
  citing_publication_number TEXT NOT NULL, 
  cited_publication_number TEXT NOT NULL, 
  PRIMARY KEY (
    citing_publication_number, cited_publication_number
  ), 
  FOREIGN KEY (citing_publication_number) REFERENCES patents_1.patents_mockdata1(publication_number), 
  FOREIGN KEY (cited_publication_number) REFERENCES patents_1.patents_mockdata1(publication_number)
);
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 2. Generate Citation Relationships

## Description

Citation data is generated for the existing patent dataset. Each patent is assigned one or more citations pointing only to patents published earlier, ensuring valid citation relationships.

### SQL Query

```sql
CREATE TEMP TABLE ordered_patents AS 
SELECT 
  publication_number, 
  publication_date, 
  ROW_NUMBER() OVER (
    ORDER BY 
      publication_date, 
      publication_number
  ) AS rn 
FROM patents_1.patents_mockdata1;

INSERT INTO patent_citations
(
    citing_publication_number,
    cited_publication_number
)
SELECT
    p.publication_number,
    c.publication_number
FROM ordered_patents p
CROSS JOIN LATERAL
(
    SELECT DISTINCT
        (
            GREATEST(1, p.rn - 1000) +
            floor(random() * LEAST(1000, p.rn - 1))
        )::bigint AS random_rn
    FROM generate_series(1,10)
    WHERE p.rn > 1
    LIMIT (floor(random() * 3) + 1)::int
) r
JOIN ordered_patents c
  ON c.rn = r.random_rn
WHERE p.rn > 1;
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 3. Retrieve Complete Citation Hierarchy

## Description

A recursive query is used to retrieve the complete citation hierarchy for a given publication number. The output includes every citation level along with its corresponding hierarchy depth.

### SQL Query

```sql
WITH RECURSIVE hierarchy AS
(
    -- Start with multiple patents
    SELECT
        p.publication_number AS root_patent,
        pc.cited_publication_number,
        1 AS depth,
        p.publication_number || ' -> ' || pc.cited_publication_number AS path
    FROM
    (
        SELECT publication_number
        FROM patents_1.patents_mockdata1
        ORDER BY publication_number
        LIMIT 3
    ) p
    JOIN patent_citations pc
      ON pc.citing_publication_number = 'US0000392642'

    UNION ALL

    -- Continue recursion
    SELECT
        h.root_patent,
        pc.cited_publication_number,
        h.depth + 1,
        h.path || ' -> ' || pc.cited_publication_number
    FROM hierarchy h
    JOIN patent_citations pc
      ON pc.citing_publication_number = h.cited_publication_number
    WHERE h.depth < 3
)
SELECT
    root_patent,
    depth,
    path
FROM hierarchy
ORDER BY root_patent, depth, path;
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 4. Create Database Function for Citation Hierarchy

## Description

A reusable PostgreSQL function is created that accepts a publication number as input and returns the complete citation hierarchy. This avoids rewriting the recursive query multiple times.

### SQL Query

```sql
CREATE OR REPLACE FUNCTION get_patent_citation_paths(patent_no TEXT)
RETURNS TABLE
(
    path TEXT,
    depth INT
)
LANGUAGE SQL
AS
$$
WITH RECURSIVE hierarchy AS
(
    SELECT
        cited_publication_number,
        1 AS depth,
        patent_no || ' -> ' || cited_publication_number AS path
    FROM patent_citations
    WHERE citing_publication_number = patent_no

    UNION ALL

    SELECT
        pc.cited_publication_number,
        h.depth + 1,
        h.path || ' -> ' || pc.cited_publication_number
    FROM hierarchy h
    JOIN patent_citations pc
      ON pc.citing_publication_number = h.cited_publication_number
    WHERE h.depth < 5
)
SELECT
    path,
    depth
FROM hierarchy;
$$;

```

### Output Screenshot

> **Insert Screenshot Here**

---

# 5. Retrieve Citation Hierarchy Using Function

## Description

The function is executed against the patent table to retrieve citation information for multiple patents in a single query. This demonstrates how the function can be reused efficiently.

### SQL Query

```sql
SELECT
    p.publication_number,
    c.depth,
    c.path
FROM
(
    SELECT publication_number
    FROM patents_1.patents_mockdata1
    ORDER BY publication_number
    LIMIT 3
) p
CROSS JOIN LATERAL
    get_patent_citation_paths(p.publication_number) c
ORDER BY p.publication_number, c.depth
LIMIT 150 ;
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 6. Create Citation Analysis View

## Description

A database view is created to combine patent details with citation statistics. The view provides publication information along with direct citations, total citations across all levels, and maximum citation depth.

### SQL Query

```sql
CREATE OR REPLACE VIEW patent_citation_summary_demo AS
SELECT
    p.publication_number,
    p.inventor_name,
    p.publication_date,
    (
        SELECT COUNT(*)
        FROM patent_citations pc
        WHERE pc.citing_publication_number = p.publication_number
    ) AS direct_citation_count,
    (
        SELECT COUNT(*)
        FROM get_patent_citation_paths(p.publication_number)
    ) AS total_citation_count,
    (
        SELECT MAX(depth)
        FROM get_patent_citation_paths(p.publication_number)
    ) AS max_depth
FROM
(
    SELECT *
    FROM patents_1.patents_mockdata1
    ORDER BY publication_number
) p;
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 7. Create Materialized View

## Description

A materialized view is created using the same logic as the standard view. Unlike a normal view, the result is physically stored to improve query performance.

### SQL Query

```sql
CREATE MATERIALIZED VIEW patent_citation_summary_mv AS
SELECT *
FROM patent_citation_summary_demo; 
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 8. Performance Comparison

## Description

Performance is measured by executing the same query using a direct query, a standard view, and a materialized view. Execution plans and execution times are compared to evaluate performance improvements.

## Direct Query

### SQL Query

```sql
EXPLAIN ANALYZE
SELECT
    p.publication_number,
    c.depth,
    c.path
FROM
(
    SELECT publication_number
    FROM patents_1.patents_mockdata1
    ORDER BY publication_number
    LIMIT 3
) p
CROSS JOIN LATERAL
    get_patent_citation_paths(p.publication_number) c
ORDER BY p.publication_number, c.depth
LIMIT 100 ;
```

### Screenshot

> **Insert Screenshot Here**

---

## View

### SQL Query

```sql
EXPLAIN ANALYZE
SELECT *
FROM patent_citation_summary_demo
limit 100;
```


### Screenshot

> **Insert Screenshot Here**

---

## Materialized View

### SQL Query

```sql
EXPLAIN ANALYZE
SELECT *
FROM patent_citation_summary_mv
limit 100;
```

### Screenshot

> **Insert Screenshot Here**

---

# 9. Performance Observations

## Description

The execution times and execution plans are compared to identify the most efficient approach. The analysis highlights the advantages and trade-offs between direct queries, views, and materialized views.


# 10. Add New Citation Records

## Description

Additional citation records are inserted into the citation table to verify how both the standard view and the materialized view respond to underlying data changes.

### SQL Query

```sql
INSERT INTO patent_citations
(
    citing_publication_number,
    cited_publication_number
)
VALUES
(
    'US1234589012',
    'US1234589112'
);
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 11. Refresh Materialized View

## Description

Since materialized views do not automatically reflect data changes, an appropriate refresh command is executed to synchronize the stored data with the latest citation records.

### SQL Query

```sql
REFRESH MATERIALIZED VIEW patent_citation_summary_mv;
```

### Output Screenshot

> **Before Refresh
> **After Refresh

---


# 13. Overall Observations

- Citation relationships were successfully generated for the existing patent dataset.
- Recursive queries accurately retrieved citation hierarchies across multiple levels.
- The database function simplified repeated hierarchy retrieval.
- Views provided real-time results based on the latest data.
- Materialized views significantly reduced query execution time for repeated analysis.
- Materialized views required manual refresh after inserting new citation records.
- Performance testing confirmed the benefits of storing precomputed results for analytical workloads.
- Execution plans were analyzed to compare optimizer behavior for each implementation.

---

# 14. Conclusion

The patent citation system was successfully implemented using PostgreSQL recursive queries, functions, views, and materialized views. Performance testing demonstrated that materialized views provide the best read performance for large analytical datasets, while standard views always reflect the latest data. The complete implementation was validated on approximately **1 million patent records**, and all SQL scripts, execution plans, observations, and screenshots were documented for version control in GitHub.

---
