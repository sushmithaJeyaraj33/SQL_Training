# Day 3 - Complex Data Representation & Analysis
**Project:** Patent Analytics

> **Note:** This README follows the same structure as the Day 2 report. Replace each **Screenshot** placeholder with your GitHub image URL after uploading screenshots.

---

# 1. Create Patent Inventor Array

## Objective

Represent each patent with all associated inventor names in a single ARRAY column.

## SQL

```sql
CREATE TABLE patents_1.patent_inventor_array AS
SELECT
    publication_number,
    ARRAY_AGG(inventor_name ORDER BY inventor_name) AS inventor_array
FROM patents_1.patent_inventors_listdata
GROUP BY publication_number;
```

### Explanation

`ARRAY_AGG()` groups multiple inventor records into one ARRAY for every patent, reducing duplicate rows while preserving one-to-many relationships.

### Screenshot

> Insert screenshot here.

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
