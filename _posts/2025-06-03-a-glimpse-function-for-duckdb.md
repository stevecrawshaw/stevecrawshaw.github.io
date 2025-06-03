---
layout: post
title: A Glimpse function for DuckDB
date: 2025-06-03
categories: [SQL]
tags:
- sql
- duckdb
- glimpse
- R
- dplyr
---

I use the [glimpse](https://dplyr.tidyverse.org/reference/glimpse.html) function in the ```dplyr``` package all the time. You can put it at the end of your pipeline to get a quick overview of the data. It is a great way to see the structure of the data, the column names, and the first few rows of data.
I wanted to create a similar function for DuckDB that I could use in SQL queries. The idea is to create a macro that takes a table name as an argument and returns the schema of the table along with a sample of the data.

The SQL below implements this functionality. It uses a Common Table Expression (CTE) to get the schema of the table and another CTE to get a sample of the data. The sample data is then unpivoted to make it easier to read. Finally, the schema and sample data are joined together to produce the final output.


```sql
CREATE OR REPLACE MACRO glimpse(table_name) AS TABLE
WITH TableSchema AS (
    
    SELECT
        cid,  
        name AS column_name,
        type AS column_type
    FROM pragma_table_info(table_name)
),
SampleData AS (
    -- Select the first 5 rows from the target table
    SELECT *
    FROM query_table(table_name)
    LIMIT 5
),
SampleDataUnpivoted AS (
    -- Unpivot the sample data: columns become rows
    UNPIVOT (SELECT list(COLUMNS(*)::VARCHAR) FROM SampleData)
    ON COLUMNS(*)
    INTO
        NAME column_name
        VALUE sample_values_list -- This will be a list of strings
)
-- Final selection joining schema info with sample data
SELECT
    ts.column_name,
    ts.column_type,
    -- Convert the list to string and remove brackets for cleaner display
    regexp_replace(usp.sample_values_list::VARCHAR, '^\[|\]$', '', 'g') AS sample_data
FROM TableSchema ts
JOIN SampleDataUnpivoted usp ON ts.column_name = usp.column_name
ORDER BY ts.cid; 
```

The next step is making this available to use in a SQL script. This is fairly straightforward because you can create the macro in a small duckdb database and then push the database up to GitHub (blob storage). The macro can then be used in any DuckDB database by attaching the database containing the macro.

```sql

ATTACH 'https://github.com/stevecrawshaw/vs-code-setup/raw/refs/heads/main/m.db' AS m;

(SELECT * FROM duckdb_functions() WHERE database_name = 'm' AND function_name = 'glimpse');
-- This is how to call the glimpse function
SELECT * FROM m.glimpse(the_table);

```
