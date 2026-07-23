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
### Screenshot

> <img width="461" height="158" alt="01" src="https://github.com/user-attachments/assets/487f694a-5330-4d04-8382-05e6617ae871" />
### Explanation

This table stores the primary patent information. Each record contains a unique publication number along with inventor details, publication date, patent title, and abstract.

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
> <img width="597" height="579" alt="02" src="https://github.com/user-attachments/assets/32f5703f-48fd-43fd-a525-b9cf1f9eb321" />



---
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

<img width="1365" height="396" alt="03" src="https://github.com/user-attachments/assets/edd36e4d-92d1-46df-bc02-1d96d7ee58dc" />



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

<img width="556" height="137" alt="04" src="https://github.com/user-attachments/assets/ed026625-c88e-49e0-a547-d8ffb3165e10" />


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

<img width="583" height="393" alt="05" src="https://github.com/user-attachments/assets/18001be6-4d0f-45bf-91e2-47f00f516698" />


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
<img width="1124" height="460" alt="06" src="https://github.com/user-attachments/assets/c0856ea8-89cd-412a-af51-6deac4447f75" />


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

# 3. Find Patents by a Specific Inventor

## Objective

Retrieve all patents associated with a **specific inventor** using PostgreSQL ARRAY operators. This demonstrates how ARRAY data can be efficiently searched without normalizing it back into individual rows.

---

## Method 1: Using ARRAY Contains Operator (`@>`)

```sql
SELECT *
FROM patents_1.patent_inventor_array
WHERE inventor_array @> ARRAY['Inventor_22454'];
```

### Explanation

The `@>` operator checks whether the ARRAY on the left contains all the elements of the ARRAY on the right.

In this query, PostgreSQL searches for all patents whose `inventor_array` contains **Inventor_22454**.

For example,

| Publication Number | Inventor Array |
|--------------------|----------------|
| US0000000001 | {Inventor_1002, Inventor_22454, Inventor_7856} |

Since **Inventor_22454** exists in the ARRAY, the patent is returned.

---

### Screenshot

> <img width="438" height="221" alt="07" src="https://github.com/user-attachments/assets/425c196d-2fab-4743-a28d-7419575e7404" />


---

## Method 2: Using the `ANY()` Operator

```sql
SELECT *
FROM patents_1.patent_inventor_array
WHERE 'Inventor_22454' = ANY(inventor_array);
```


### Explanation

The `ANY()` operator checks whether the specified value matches **any element** within the ARRAY.


---

> **Find patents where a specific inventor is associated with the patent.**

Since inventor names are stored inside an ARRAY, PostgreSQL provides specialized ARRAY operators such as `@>` and `ANY()` to search the ARRAY directly, eliminating the need to split the ARRAY back into individual rows.

---

### Difference Between `@>` and `ANY()`

| `@>` | `ANY()` |
|------|---------|
| Checks whether an ARRAY contains another ARRAY. | Checks whether a value exists in an ARRAY. |
| Right side must be an ARRAY. | Right side is an ARRAY, left side is a single value. |
| Suitable for ARRAY containment checks. | Suitable for single-value searches. |

---

# 4. Find Patents Matching At Least One Inventor from a Given List

## Objective

Retrieve patents that are associated with **at least one inventor** from a specified list. This demonstrates PostgreSQL's ability to compare ARRAY values and identify overlapping elements.

---

## Find Patents Using ARRAY Overlap Operator (`&&`)

```sql
SELECT *
FROM patents_1.patent_inventor_array
WHERE inventor_array && ARRAY
[
    'Inventor_5266',
    'Inventor_22454',
    'Inventor_10739'
];
```
> <img width="412" height="297" alt="08" src="https://github.com/user-attachments/assets/c4196741-d5de-4df3-89e8-007109489b9c" />

### Explanation

The `&&` operator checks whether two ARRAYs have **at least one common element**.

In this query, PostgreSQL compares the `inventor_array` of each patent with the given list of inventors. If one or more inventor names match, the corresponding patent is returned.

For example,

**Inventor List**

```text
{Inventor_5266, Inventor_22454, Inventor_10739}
```

**Patent Inventor Array**

```text
{Inventor_1045, Inventor_22454, Inventor_8912}
```

Since **Inventor_22454** appears in both arrays, the patent satisfies the condition and is included in the result.

---

> **Find patents where at least one inventor from a given list is associated with the patent.**

Instead of executing multiple search conditions using `OR`, PostgreSQL's ARRAY overlap operator (`&&`) performs this comparison efficiently in a single operation.

This approach becomes especially useful when searching against large lists of inventors in production databases.

---

### Advantages of Using `&&`

- Checks multiple inventor names in a single query.
- Simplifies SQL by avoiding multiple `OR` conditions.
- Optimized for ARRAY data types.
- Can leverage a **GIN Index** for faster searches on large datasets.

---
# 5. Find Two Patents Having at Least One Inventor in Common

## Objective

Identify pairs of patents that share **one or more common inventors**. This demonstrates how PostgreSQL ARRAY operators can be used to compare data between rows and discover relationships among patents.

---

## Find Patents Sharing Common Inventors

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

### Explanation

This query performs a **self-join** on the `patent_inventor_array` table, comparing the inventor arrays of two different patents.

The `&&` (ARRAY Overlap) operator checks whether both patents have **at least one inventor in common**. If a common inventor exists, the pair of patents is returned.



> **Find two patents that have at least one inventor in common.**

This type of analysis helps identify collaboration between inventors and discover patents connected through shared contributors.

In real-world patent databases, this technique is commonly used for:

- Inventor collaboration analysis
- Patent relationship discovery
- Research network analysis
- Innovation trend analysis

---

### Advantages of This Approach

- Efficiently compares inventor arrays without converting them into individual rows.
- Uses PostgreSQL's built-in ARRAY overlap operator (`&&`).
- Can be significantly accelerated using a **GIN Index** on the ARRAY column.
- Scales well for large patent datasets.

---
### Screenshot

> <img width="418" height="325" alt="09" src="https://github.com/user-attachments/assets/d62683e7-4b81-4b44-9d89-e5fcc416f31c" />


---

# 6. Convert the Grouped Inventor Information Back into Individual Rows

## Objective

Convert the inventor ARRAY back into individual rows and verify that the reconstructed data matches the original patent-to-inventor relationships. This demonstrates that the ARRAY representation preserves all inventor information without any data loss.

---

## Convert ARRAY into Individual Rows

```sql
SELECT
    publication_number,
    UNNEST(inventor_array) AS inventor_name
FROM patents_1.patent_inventor_array;
```

### Explanation

The `UNNEST()` function expands each element of an ARRAY into a separate row.

Since the inventors for each patent were previously grouped using `ARRAY_AGG()`, this query reverses that operation by converting the inventor ARRAY back into the original row-based format.

For example,

**Before UNNEST**

| Publication Number | Inventor Array |
|--------------------|----------------|
| US0000000001 | {Inventor_1023, Inventor_5432, Inventor_8765} |

**After UNNEST**

| Publication Number | Inventor Name |
|--------------------|---------------|
| US0000000001 | Inventor_1023 |
| US0000000001 | Inventor_5432 |
| US0000000001 | Inventor_8765 |

---

### Screenshot

> <img width="420" height="299" alt="10" src="https://github.com/user-attachments/assets/f0344393-dd98-49d0-adc5-acd2196a8081" />


---

## Verify the Original Patent-Inventor Relationships

### Count Records in the Original Mapping Table

```sql
SELECT COUNT(*)
FROM patents_1.patent_inventors_listdata;
```

### Count Records After Applying UNNEST

```sql
SELECT COUNT(*)
FROM
(
    SELECT
        publication_number,
        UNNEST(inventor_array)
    FROM patents_1.patent_inventor_array
) t;
```

> **Convert the grouped inventor information back into individual rows and verify that the result matches the original inventor-to-patent relationships.**

To verify this, the total number of inventor records in the original mapping table is compared with the number of rows produced after applying `UNNEST()`.

If both counts are identical, it confirms that:

- Every inventor has been successfully reconstructed.
- No inventor information was lost during the `ARRAY_AGG()` operation.
- The ARRAY representation accurately preserves the original one-to-many relationship.

This validation step ensures the integrity of the transformed data.

---

Using `UNNEST()` allows PostgreSQL to:

- Convert ARRAY values back into relational rows.
- Validate data consistency.
- Support downstream reporting and ETL processes.
- Interoperate with applications expecting normalized tables.

---

# 6. Create Patent Metadata using JSONB

Use `jsonb_build_object()` to store country, category, status, technology and filing date inside a single JSONB column.

Include your SQL from the project here.

### Screenshot

> <img width="546" height="300" alt="11" src="https://github.com/user-attachments/assets/d88c9f44-b762-4714-83b8-97c65281ffc3" />


---

# 7. Create Structured Patent Metadata Using JSONB

## Objective

Store multiple patent attributes such as **country, category, status, technology area, and filing date** within a single **JSONB** column. This provides a flexible way to manage semi-structured patent metadata while allowing individual attributes to be queried and updated efficiently.

---

## Create Patent Metadata Table

```sql
CREATE TABLE patents_1.patent_metadata AS

SELECT
    publication_number,

    jsonb_build_object(

        'country',
        (ARRAY['US','IN','JP','CN','DE'])[floor(random()*5+1)],

        'category',
        (ARRAY['Mechanical','Software','Medical','Chemical','Electronics'])[floor(random()*5+1)],

        'status',
        (ARRAY['Granted','Pending','Expired'])[floor(random()*3+1)],

        'technology',
        (ARRAY['AI','Cloud','Blockchain','IoT','Cyber Security'])[floor(random()*5+1)],

        'filing_date',
        publication_date

    ) AS metadata

FROM patents_1.patents_mockdata1;
```

### Explanation

The `jsonb_build_object()` function creates a **JSONB document** by combining multiple patent attributes into a single structured column named `metadata`.

Instead of storing separate columns for each attribute, all related information is organized as key-value pairs within one JSON object.

For example,

```json
{
  "country": "US",
  "category": "Software",
  "status": "Granted",
  "technology": "AI",
  "filing_date": "2018-06-14"
}
```

This approach is ideal for storing semi-structured information where attributes may evolve over time without requiring changes to the table schema.

---

## View Sample Metadata

```sql
SELECT *
FROM patents_1.patent_metadata
LIMIT 5;
```

### Explanation

This query displays a sample of the generated metadata records to verify that the JSONB objects have been created successfully.

Each row contains:

- Publication Number
- JSONB Metadata

allowing multiple patent attributes to be viewed as a single structured document.

---

### Screenshot

> <img width="777" height="396" alt="12" src="https://github.com/user-attachments/assets/c5732b4a-132a-4767-8496-45b59984b945" />

<img width="1068" height="148" alt="13" src="https://github.com/user-attachments/assets/52e0aa8a-94d0-4b7a-ae10-0e5100856ac3" />



---

### Advantages of Using JSONB

- Stores multiple related attributes in one column.
- Supports flexible and evolving data structures.
- Allows efficient querying of individual JSON fields.
- Supports partial updates without replacing the entire document.
- Can be optimized using GIN indexes for fast searches.

---
# 8. Query and Filter Patent Metadata Using JSONB

## Objective

Retrieve patents based on individual attributes stored inside the **JSONB metadata** column. This demonstrates how PostgreSQL allows direct access to JSON fields without storing them as separate table columns.

---

## Find Patents by Country

```sql
SELECT COUNT(*)
FROM patents_1.patent_metadata
WHERE metadata->>'country' = 'US';
```

### Explanation

The `->>` operator extracts the value of a JSON key as text.

This query counts all patents where the **country** stored inside the JSONB metadata is **US**.

For example,

```json
{
    "country":"US",
    "category":"Software",
    "status":"Granted",
    "technology":"AI"
}
```

Only patents whose `country` value is **US** are included in the result.

---

### Screenshot

> <img width="327" height="112" alt="14" src="https://github.com/user-attachments/assets/5b51b235-914a-42e6-898f-655deca050d5" />


---

## Find Patents by Technology

```sql
SELECT *
FROM patents_1.patent_metadata
WHERE metadata->>'technology' = 'AI';
```

### Explanation

This query retrieves all patents where the **technology** attribute inside the JSON metadata is **AI**.

Rather than searching a normal relational column, PostgreSQL extracts the value from the JSONB document using the `->>` operator and compares it with the specified value.

---

### Screenshot

> <img width="1017" height="232" alt="15" src="https://github.com/user-attachments/assets/d107c935-4afb-478c-8052-cbc63dbf14a6" />


---

## Find Granted Patents

```sql
SELECT *
FROM patents_1.patent_metadata
WHERE metadata->>'status' = 'Granted'
LIMIT 5;
```

### Explanation

This query filters patents whose **status** is **Granted**.

---

### Screenshot

> <img width="1011" height="189" alt="16" src="https://github.com/user-attachments/assets/f1ba574e-342a-4c6a-b2ea-ebc166ccaa06" />


---
# 9. Modify Individual JSON Attributes Using `jsonb_set()`

## Objective

Update a specific attribute within the **JSONB metadata** without replacing the entire JSON document. This demonstrates PostgreSQL's ability to perform partial updates on structured data efficiently.

---

## Update Patent Status

```sql
UPDATE patents_1.patent_metadata
SET metadata =
jsonb_set
(
    metadata,
    '{status}',
    '"Expired"'
)
WHERE publication_number = 'US0000000002';
```

### Explanation

The `jsonb_set()` function updates only the specified key within the JSONB document.

In this example, the **status** attribute of patent **US0000000002** is changed to **Expired**, while all other metadata fields remain unchanged.

**Before Update**

```json
{
  "country": "US",
  "category": "Software",
  "status": "Granted",
  "technology": "AI",
  "filing_date": "2018-06-14"
}
```

**After Update**

```json
{
  "country": "US",
  "category": "Software",
  "status": "Expired",
  "technology": "AI",
  "filing_date": "2018-06-14"
}
```

Only the **status** field is modified, preserving the rest of the JSON document.

---

### Verify the Updated Metadata (Optional)

```sql
SELECT *
FROM patents_1.patent_metadata
WHERE publication_number = 'US0000000002';
```

### Explanation

This query retrieves the updated patent record to verify that only the **status** field has been modified successfully.

---

### Screenshot

> <img width="479" height="148" alt="17" src="https://github.com/user-attachments/assets/9728bf3c-ee59-41c3-90ea-01150243e130" />


---

## 11. Generate a single structured output for each patent containing the patent details and all associated inventors.

```
SELECT jsonb_build_object(
    'publication_number', p.publication_number,
    'title', p.title,
    'inventors', jsonb_agg(pi.inventor_name)
) AS patent_document
FROM patents_1.patents_mockdata1 p
JOIN patents_1.patent_inventors_listdata pi
USING (publication_number)
GROUP BY
    p.publication_number,
    p.title
LIMIT 5;

```
### Screenshot

> 


## 11. Generate a Hierarchical JSON Structure

## Objective

Generate a hierarchical JSON document representing the patent data in the following structure:

```text
Year
 ├── Patents
      ├── Patent Details
      └── Inventors
```

---

## Generate Hierarchical JSON

```sql
SELECT jsonb_build_object
(
    'years',

    jsonb_agg(year_data ORDER BY patent_year)
)
FROM
(
    SELECT
        patent_year,

        jsonb_build_object
        (
            'year', patent_year,

            'patents',

            jsonb_agg
            (
                jsonb_build_object
                (
                    'patent_details',

                    jsonb_build_object
                    (
                        'publication_number', publication_number,
                        'title', title,
                        'publication_date', publication_date,
                        'abstract', abstract
                    ),

                    'inventors',

                    (
                        SELECT jsonb_agg(pi.inventor_name ORDER BY pi.inventor_name)
                        FROM patents_1.patent_inventors_listdata pi
                        WHERE pi.publication_number = p.publication_number
                    )
                )
            )
        ) AS year_data

    FROM
    (
        SELECT *,
               EXTRACT(YEAR FROM publication_date) AS patent_year
        FROM patents_1.patents_mockdata1
    ) p

    GROUP BY patent_year
) y;
```

### Explanation

Using `jsonb_build_object()` and `jsonb_agg()`, PostgreSQL builds a hierarchical JSON structure where:

- The top level represents the **publication year**.
- Each year contains a collection of **patents**.
- Every patent contains its **publication number**, **title**, and an array of **inventors**
---

Instead of returning flat relational tables, PostgreSQL generates a nested JSON hierarchy that groups patents by publication year and includes inventor information within each patent.

This type of output is commonly used in dashboards, reporting tools, and web applications where hierarchical data is easier to consume.

---

### Advantages of Hierarchical JSON

- Eliminates additional data transformation in application code.
- Represents parent-child relationships naturally.
- Simplifies integration with frontend applications.
- Reduces network payload by returning structured data in a single query.

---

### Screenshot



---

# 11. Performance Optimization

# 12. Performance Optimization Using GIN Indexes

## Objective

Investigate how indexing improves the performance of ARRAY and JSONB queries by comparing execution plans before and after creating **GIN (Generalized Inverted Index)** indexes.

---

# 12.1 Performance Analysis for ARRAY Data

## Query Performance Before Indexing

```sql
EXPLAIN ANALYZE

SELECT *
FROM patents_1.patent_inventor_array
WHERE inventor_array @> ARRAY['Inventor_309'];
```

### Explanation

Before creating an index, PostgreSQL performs a **Sequential Scan**, meaning every row in the table is examined to determine whether the inventor exists in the ARRAY.

For large datasets, this approach becomes slower because the database must scan the complete table.

---

### Screenshot

> <img width="1146" height="239" alt="20" src="https://github.com/user-attachments/assets/3773ba5b-5b3b-4bad-ba00-eb1c26ff112f" />


---

## Create GIN Index on ARRAY Column

```sql
CREATE INDEX idx_inventors
ON patents_1.patent_inventor_array
USING GIN(inventor_array);
```

### Explanation

A **GIN (Generalized Inverted Index)** is specifically designed for searching complex data types such as **ARRAY** and **JSONB**.

Instead of scanning every row, PostgreSQL creates an index for each inventor stored inside the ARRAY, allowing much faster searches.

---

### Screenshot

> <img width="420" height="67" alt="21" src="https://github.com/user-attachments/assets/b3a794cf-4355-42a5-a041-ca57ba479f7f" />


---

## Query Performance After Indexing

```sql
EXPLAIN ANALYZE

SELECT *
FROM patents_1.patent_inventor_array
WHERE inventor_array @> ARRAY['Inventor_309'];
```

### Explanation

After creating the GIN index, PostgreSQL can directly locate matching inventor values using the index instead of scanning the entire table.

This significantly reduces query execution time, especially for large datasets containing millions of records.

---

### Screenshot

> <img width="1013" height="227" alt="22" src="https://github.com/user-attachments/assets/c34d4aa5-3f3d-4818-861c-56aab439ecd3" />


---

# 12.2 Performance Analysis for JSONB Data

## Query Performance Before Indexing

```sql
EXPLAIN ANALYZE

SELECT *
FROM patents_1.patent_metadata
WHERE metadata->>'technology' = 'AI';
```

### Explanation

Without an index, PostgreSQL performs a Sequential Scan and evaluates the JSON attribute for every row in the table.

Although JSONB supports flexible storage, searching without an index becomes expensive as data volume increases.

---

### Screenshot

> <img width="1040" height="237" alt="23" src="https://github.com/user-attachments/assets/424c7b17-b0e6-4128-9089-d8785d344512" />


---

## Create GIN Index on JSONB Column

```sql
CREATE INDEX idx_metadata
ON patents_1.patent_metadata
USING GIN(metadata);
```

### Explanation

This GIN index stores searchable entries for the JSONB document.

It enables PostgreSQL to locate matching JSON keys and values efficiently instead of scanning every metadata record.

---

### Screenshot

> <img width="426" height="79" alt="24" src="https://github.com/user-attachments/assets/3c0ca3dc-fb57-47b7-80fb-748b8fcca3a9" />


---

## Query Performance After Indexing

```sql
EXPLAIN ANALYZE

SELECT *
FROM patents_1.patent_metadata
WHERE metadata->>'technology' = 'AI';
```

### Explanation

After indexing, PostgreSQL can use the GIN index to locate patents containing the required technology.

Compared to a Sequential Scan, indexed searches require significantly fewer disk reads and improve overall query performance.

---

### Screenshot

> <img width="992" height="241" alt="25" src="https://github.com/user-attachments/assets/3847b140-d599-4e52-8a1e-94b67858058f" />


---

# Why GIN Index?

GIN (Generalized Inverted Index) is optimized for **multi-valued data types** such as:

- ARRAY
- JSONB

Unlike B-Tree indexes, GIN indexes store individual elements contained inside complex data structures, making membership and containment searches much faster.

---

# Performance Comparison

| Query | Before Index | After Index |
|---------|--------------|-------------|
| ARRAY Search (`@>`) | Sequential Scan | GIN Index Scan |
| JSONB Search (`->>`) | Sequential Scan | GIN Index Scan |

> **Note:** Replace this table with the actual execution times obtained from your `EXPLAIN ANALYZE` output.

Example:

| Query | Before Index | After Index |
|---------|--------------|-------------|
| ARRAY Search | 145 ms | 12 ms |
| JSONB Search | 178 ms | 15 ms |

---

# Benefits of Indexing

- Eliminates full table scans.
- Reduces query execution time.
- Improves performance on large datasets.
- Optimizes ARRAY and JSONB searches.


---

# 13. Additional PostgreSQL Operations for Complex Data

## Objective

Explore additional PostgreSQL operations supported for **ARRAY** and **JSONB** data types. These functions provide powerful capabilities for manipulating, querying, and formatting complex data structures beyond the basic operations implemented in the previous tasks.

The assignment requires:

> **Research and demonstrate at least three additional operations that PostgreSQL supports for these types of complex data.**

---

# 13.1 Find the Position of an Inventor in an ARRAY

## SQL

```sql
SELECT
    publication_number,
    inventor_array,
    array_position(inventor_array,'Inventor_309') AS inventor_position
FROM patents_1.patent_inventor_array
WHERE inventor_array @> ARRAY['Inventor_309']
LIMIT 5;
```

### Explanation

The `array_position()` function returns the position of a specified element within an ARRAY.

If the inventor exists, PostgreSQL returns its index; otherwise, it returns `NULL`.

This function is useful when the order of elements inside an ARRAY is important.

---

### Screenshot

> <img width="642" height="412" alt="26" src="https://github.com/user-attachments/assets/01683a6d-bdc9-4c69-adf5-7dbaed5347e7" />


---

# 13.2 Append a New Inventor to an ARRAY

## SQL

```sql
SELECT
    publication_number,
    inventor_array,
    array_append(inventor_array,'Inventor_309') AS updated_inventors
FROM patents_1.patent_inventor_array
WHERE publication_number='US0000001133';
```

### Explanation

The `array_append()` function adds a new element to the end of an existing ARRAY without modifying the original stored value.

This operation is useful for previewing or generating updated ARRAY values before performing an actual database update.

---

### Screenshot

> <img width="1172" height="167" alt="27" src="https://github.com/user-attachments/assets/7d5fd25d-06ba-44eb-b31b-2f0794f735f5" />


---

# 13.3 Remove an Inventor from an ARRAY

## SQL

```sql
SELECT
    publication_number,
    inventor_array,
    array_remove(inventor_array,'Inventor_16292') AS updated_inventors
FROM patents_1.patent_inventor_array
WHERE publication_number='US0000001136';
```

### Explanation

The `array_remove()` function removes all occurrences of the specified element from an ARRAY.

It is commonly used when an inventor, tag, or category needs to be excluded from a multi-valued column.

---

### Screenshot

> <img width="681" height="159" alt="28" src="https://github.com/user-attachments/assets/9a3a0a4b-e2b8-4f8d-b438-d0453db4a25b" />


---

# 13.4 Retrieve a Portion of an ARRAY (Array Slicing)

## SQL

```sql
SELECT
    publication_number,
    inventor_array[1:2] AS first_two_inventors
FROM patents_1.patent_inventor_array
WHERE cardinality(inventor_array) >= 2
LIMIT 10;
```

### Explanation

Array slicing retrieves only a specified range of elements from an ARRAY.

In this example, only the **first two inventors** are returned for patents containing at least two inventors.

This is useful when displaying a subset of large ARRAY values.

---

### Screenshot

> <img width="566" height="307" alt="29" src="https://github.com/user-attachments/assets/133932bc-308b-4800-af79-2a78f7fdc3d1" />


---

# 13.5 Check Whether a JSON Key Exists

## SQL

```sql
SELECT *
FROM patents_1.patent_metadata
WHERE metadata ? 'technology'
LIMIT 20;
```

### Explanation

The `?` operator checks whether a specified key exists in a JSONB document.

Only patents whose metadata contains the **technology** key are returned.

This operation is useful when working with optional or dynamic JSON attributes.

---

### Screenshot

> <img width="1061" height="269" alt="30" src="https://github.com/user-attachments/assets/c5b3bd0e-1bff-4947-8d99-4a5efe77d797" />


---

# 13.6 Check Whether Multiple JSON Keys Exist

## SQL

```sql
SELECT *
FROM patents_1.patent_metadata
WHERE metadata ?& ARRAY['country','technology']
LIMIT 20;
```

### Explanation

The `?&` operator verifies that **all specified keys** are present in the JSONB document.

In this example, only records containing both **country** and **technology** keys are returned.

This is particularly useful for validating JSON document structure.

---

### Screenshot

> <img width="1095" height="305" alt="31" src="https://github.com/user-attachments/assets/0252a1bf-aeb6-423f-a861-1a954a3f01cf" />


---

# 13.7 Display Formatted JSON Output

## SQL

```sql
SELECT jsonb_pretty(metadata)
FROM patents_1.patent_metadata
LIMIT 5;
```

### Explanation

The `jsonb_pretty()` function formats JSONB data into an easy-to-read structure.

Although it does not modify the stored data, it improves readability during debugging, demonstrations, and documentation.

---

### Screenshot

> <img width="589" height="631" alt="32" src="https://github.com/user-attachments/assets/7655437f-168d-4269-9877-99454673a5d4" />


---

# Conclusion

This project demonstrates PostgreSQL's advanced support for **ARRAY** and **JSONB** data types to represent, query, and manipulate complex data efficiently.

Key accomplishments include:

- Grouped multiple inventors into ARRAY columns.
- Queried ARRAY data using PostgreSQL operators.
- Converted ARRAY data back into relational rows.
- Stored flexible patent metadata using JSONB.
- Queried and updated individual JSON attributes.
- Generated structured and hierarchical JSON documents.
- Optimized ARRAY and JSONB queries using GIN indexes.
- Explored additional ARRAY and JSONB operations for advanced data manipulation.



