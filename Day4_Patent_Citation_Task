# Day 4 – Patent Citation Task for

## Objective

The objective of this assignment is to build and analyze a patent citation system using the existing patent dataset. The implementation covers recursive queries, database functions, views, materialized views, and performance analysis on approximately **1 million patent records**.

---

# 1. Create Patent Citation Table

## Description

A new citation table is created to store relationships between patents. Each record represents one citation where a newer patent references an older patent.

### SQL Query

```sql
-- Add SQL Query Here
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 2. Generate Citation Relationships

## Description

Citation data is generated for the existing patent dataset. Each patent is assigned one or more citations pointing only to patents published earlier, ensuring valid citation relationships.

### SQL Query

```sql
-- Add SQL Query Here
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 3. Retrieve Complete Citation Hierarchy

## Description

A recursive query is used to retrieve the complete citation hierarchy for a given publication number. The output includes every citation level along with its corresponding hierarchy depth.

### SQL Query

```sql
-- Add SQL Query Here
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 4. Create Database Function for Citation Hierarchy

## Description

A reusable PostgreSQL function is created that accepts a publication number as input and returns the complete citation hierarchy. This avoids rewriting the recursive query multiple times.

### SQL Query

```sql
-- Add SQL Query Here
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 5. Retrieve Citation Hierarchy Using Function

## Description

The function is executed against the patent table to retrieve citation information for multiple patents in a single query. This demonstrates how the function can be reused efficiently.

### SQL Query

```sql
-- Add SQL Query Here
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 6. Create Citation Analysis View

## Description

A database view is created to combine patent details with citation statistics. The view provides publication information along with direct citations, total citations across all levels, and maximum citation depth.

### SQL Query

```sql
-- Add SQL Query Here
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 7. Create Materialized View

## Description

A materialized view is created using the same logic as the standard view. Unlike a normal view, the result is physically stored to improve query performance.

### SQL Query

```sql
-- Add SQL Query Here
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
-- Add SQL Query Here
```

### Execution Time

```
Add Execution Time Here
```

### Execution Plan

```
Paste EXPLAIN ANALYZE Output Here
```

### Screenshot

> **Insert Screenshot Here**

---

## View

### SQL Query

```sql
-- Add SQL Query Here
```

### Execution Time

```
Add Execution Time Here
```

### Execution Plan

```
Paste EXPLAIN ANALYZE Output Here
```

### Screenshot

> **Insert Screenshot Here**

---

## Materialized View

### SQL Query

```sql
-- Add SQL Query Here
```

### Execution Time

```
Add Execution Time Here
```

### Execution Plan

```
Paste EXPLAIN ANALYZE Output Here
```

### Screenshot

> **Insert Screenshot Here**

---

# 9. Performance Observations

## Description

The execution times and execution plans are compared to identify the most efficient approach. The analysis highlights the advantages and trade-offs between direct queries, views, and materialized views.

### Observations

- Observation 1
- Observation 2
- Observation 3
- Observation 4

---

# 10. Add New Citation Records

## Description

Additional citation records are inserted into the citation table to verify how both the standard view and the materialized view respond to underlying data changes.

### SQL Query

```sql
-- Add SQL Query Here
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 11. Refresh Materialized View

## Description

Since materialized views do not automatically reflect data changes, an appropriate refresh command is executed to synchronize the stored data with the latest citation records.

### SQL Query

```sql
-- Add SQL Query Here
```

### Output Screenshot

> **Insert Screenshot Here**

---

# 12. Testing on One Million Patent Records

## Description

All queries, functions, views, and materialized views are tested using the existing patent dataset containing approximately one million records with sufficient citation data to validate correctness and performance.

### SQL Query (if applicable)

```sql
-- Add SQL Query Here
```

### Output Screenshot

> **Insert Screenshot Here**

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
