---
layout: post
title: DuckDB - a couple of learnings
date: 2025-08-06
categories: [SQL, duckDB]
tags:
- SQL
- over()
- csv
---

I've been updating some slides for emissions in the West of England using data from [DESNZ](https://www.gov.uk/government/statistics/uk-local-authority-and-regional-greenhouse-gas-emissions-statistics-2005-to-2023).

R and RStudio are unwell so I resolved to use duckdb to derive the proportions of emissions by sector for the latest year and the 10 year change in emissions. Nothing dramatic. I did learn a couple of useful things.

1. Renaming all columns in a table to snake case doesn't seem that easy in SQL compared to the wonderful `janitor::clean_names()` function in R. However there is a nice workaround if you are importing from a csv. Simply use the `normalize_names = true` parameter in the read_csv() function. From the [docs](https://duckdb.org/docs/stable/data/csv/overview): *Normalize column names. This removes any non-alphanumeric characters from them. Column names that are reserved SQL keywords are prefixed with an underscore character (_).*

2. Calculating the proportions of values compared to the totals of a column. When I wanted to work out the proportion of emissions by sector in the same table as the percentage change over 10 years I got a bit stuck as I didn't want to do a GROUP BY. The solution is to use the OVER() clause in the column calculation like this:
```sql
SELECT *,
    ("2023_LEP" - "2014_LEP") * 100 / "2014_LEP"  lep_pc_change,
    ("2023_UK" - "2014_UK") * 100 / "2014_UK"  uk_pc_change,
    "2023_LEP" * 100 / SUM("2023_LEP")  OVER() lep_pc_total,
    "2023_UK" * 100 / SUM("2023_UK")  OVER() uk_pc_total
FROM year_pivot_tbl
WHERE la_ghg_sector != 'LULUCF';
```
The OVER () clause turns the aggregation into a window function. An empty () specifies that the window for the sum is the entire result set.