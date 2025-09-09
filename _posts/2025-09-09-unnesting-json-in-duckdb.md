---
layout: post
title: Unnesting JSON in duckdb
date: 2025-09-09
categories: [SQL]
tags:
- duckdb
- json
- unnest
- struct
---

It's trivial for a real data analyst, but I always struggle with json, and so here is my aide memoire for how to unnest and rectangle json data in duckdb.

Let's say we've got some wild - caught JSON from our open data portal API:

```python
{
  "total_count": 87,
  "results": [
    {
      "ladnm": "Bath and North East Somerset",
      "current_energy_rating": "A",
      "construction_epoch": "1900 - 1930",
      "count": 3
    },
    {
      "ladnm": "Bath and North East Somerset",
      "current_energy_rating": "A",
      "construction_epoch": "1930 to present",
      "count": 212
    },
    {
      "ladnm": "Bath and North East Somerset",
      "current_energy_rating": "A",
      "construction_epoch": "Unknown",
      "count": 82
    },
    {
      "ladnm": "Bath and North East Somerset",
      "current_energy_rating": "B",
      "construction_epoch": "1900 - 1930",
      "count": 55
    },
    {
      "ladnm": "Bath and North East Somerset",
      "current_energy_rating": "B",
      "construction_epoch": "1930 to present",
      "count": 2614
    }
  ]
}
```
We just want the data, which is in the `results` object. The results object is a struct in duckdb land, i.e. like a dictionary. But we want to "unnest" that object so we get the dictionaries as rows. Then we want to turn those dictionaries into a table where the keys are the column names and the values are records. This uses the [struct functions](https://duckdb.org/docs/stable/sql/functions/struct) in duckdb's SQL. Here's the magic.

```sql
WITH json_cte AS -- A CTE to hold the interim rows
(SELECT unnest(results) r
FROM read_json('data/response.json'))
-- the line below uses the dot notation which serves as an alias for struct_extract() function
SELECT r.ladnm local_authority, r.current_energy_rating current_energy_rating, r.construction_epoch construction_epoch, r.count count
FROM json_cte;

-- If you just want all the columns you can just use the asterisk like r.*
WITH json_cte AS
(SELECT unnest(results) r
FROM read_json('data/response.json'))
SELECT r.*
FROM json_cte;
-- and if you've done 
LOAD HTTPFS;
-- you can read directly from a URL

WITH json_cte AS
(SELECT unnest(results) r
FROM read_json('https://opendata.westofengland-ca.gov.uk/api/explore/v2.1/catalog/datasets/lep-epc-domestic-point/records?select=count%28%2A%29%20AS%20count%2Cladnm%2Ccurrent_energy_rating%2Cconstruction_epoch&group_by=ladnm%2Ccurrent_energy_rating%2Cconstruction_epoch&limit=100&offset=0&timezone=UTC&include_links=false&include_app_metas=false'))
SELECT r.*
FROM json_cte;
```
┌──────────────────────────────┬───────────────────────┬────────────────────┬───────┐
│            ladnm             │ current_energy_rating │ construction_epoch │ count │
│           varchar            │        varchar        │      varchar       │ int64 │
├──────────────────────────────┼───────────────────────┼────────────────────┼───────┤
│ Bath and North East Somerset │ A                     │ 1900 - 1930        │     3 │
│ Bath and North East Somerset │ A                     │ 1930 to present    │   212 │
│ Bath and North East Somerset │ A                     │ Unknown            │    82 │
│ Bath and North East Somerset │ B                     │ 1900 - 1930        │    55 │
│ Bath and North East Somerset │ B                     │ 1930 to present    │  2614 │
│ Bath and North East Somerset │ B                     │ Unknown            │  5591 │
│ Bath and North East Somerset │ C                     │ 1900 - 1930        │  2805 │
│ Bath and North East Somerset │ C                     │ 1930 to present    │ 15447 │
│ South Gloucestershire        │ E                     │ Unknown            │    76 │
│ South Gloucestershire        │ F                     │ 1900 - 1930        │   776 │
│ South Gloucestershire        │ F                     │ 1930 to present    │   972 │
│ South Gloucestershire        │ F                     │ Unknown            │     8 │
│ South Gloucestershire        │ G                     │ 1900 - 1930        │   220 │
│ South Gloucestershire        │ G                     │ 1930 to present    │   140 │
│ South Gloucestershire        │ G                     │ Unknown            │     3 │
├──────────────────────────────┴───────────────────────┴────────────────────┴───────┤
│ 87 rows (40 shown)                                                      4 columns │
└───────────────────────────────────────────────────────────────────────────────────┘
