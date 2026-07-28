# Day 4 – Patent Citation Task

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

> <img width="769" height="177" alt="image_1" src="https://github.com/user-attachments/assets/42313625-0cc6-4ee2-a535-da25176cc321" />


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

> <img width="506" height="189" alt="image_3" src="https://github.com/user-attachments/assets/347b0479-dc60-4bd9-bfbf-b7b8024be31a" />

<img width="595" height="412" alt="image_5" src="https://github.com/user-attachments/assets/4c8661ed-5ffd-46a4-82f0-953ed7b97b9d" />

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

> <img width="732" height="637" alt="Screenshot 2026-07-28 at 8 52 27 AM" src="https://github.com/user-attachments/assets/764c04a9-3cf6-4f64-9ef9-3edd237fb956" />


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

> <img width="797" height="516" alt="image_10" src="https://github.com/user-attachments/assets/dd752965-4323-4e3a-9478-6eef5de18a4f" />


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

> <img width="929" height="599" alt="image_11" src="https://github.com/user-attachments/assets/ca7004b4-1734-4b99-8222-8c53dbdbc016" />


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

> <img width="682" height="446" alt="image_12" src="https://github.com/user-attachments/assets/204c48fc-e03d-4756-8f2c-be814c0064f6" />


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

> <img width="605" height="119" alt="image_13" src="https://github.com/user-attachments/assets/c4627d4c-3fc7-42bf-b537-3cc0f6ae7aca" />


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

> <img width="1334" height="506" alt="image_14" src="https://github.com/user-attachments/assets/e3e907ae-85e5-47d2-9c7f-cdc588413f3b" />


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

> <img width="1244" height="368" alt="image_15" src="https://github.com/user-attachments/assets/07fc946b-8613-47c6-bd9e-bc3517a76755" />


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

> <img width="1098" height="175" alt="image_16" src="https://github.com/user-attachments/assets/c356f7fc-7657-4425-add6-bd4d2aefc54c" />


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

> <img width="806" height="260" alt="image_18" src="https://github.com/user-attachments/assets/03893ae9-752b-4519-863f-71a0d49a67ff" />


<img width="876" height="90" alt="image_20" src="https://github.com/user-attachments/assets/8e8a4a00-de3d-49f3-88c8-fa0f05781499" />

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
<img width="876" height="90" alt="image_20" src="https://github.com/user-attachments/assets/955cc5fe-ccf8-44ba-9a9b-17a71e06722d" />


> **After Refresh
<img width="610" height="64" alt="image_21" src="https://github.com/user-attachments/assets/27448e7c-b143-4cc2-b2ae-a6403da3d472" />

<img width="1014" height="113" alt="Screenshot 2026-07-28 at 9 03 55 AM" src="https://github.com/user-attachments/assets/4771ab27-8347-47e2-808d-9c7e617eb79f" />


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
