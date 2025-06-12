---
layout: post
title: ETL for ODS analytics data
date: 2025-06-12
categories: [SQL]
tags:
- nushell
- duckdb
- ETL
- API
---

*TLDR;*
ETL process to extract analytics data from the OpenDataSoft API and load it into a DuckDB (motherduck) database using SQL.

I administer an open data platform provided by [OpenDataSoft](https://opendata.westofengland-ca.gov.uk/pages/homepage/). In order to monitor and evaluate the operation of the platform I need to extract analytics data from the platform's API and load it into a database for analysis. I have chosen to use [DuckDB](https://duckdb.org/) as it is lightweight, fast, and supports SQL queries. Specifically I am using a cloud hosted version of DuckDB provided by (motherduck)[https://app.motherduck.com/].

The on - platform dataset that holds the analytics data only holds data for three months on the plan that we have. I wanted to set up an ETL process that could be scheduled to run regularly to extract the data so that we don't get into a situation where we lose data. This post documents the process I developed and explains the reasoning.

Obviously it is possible to write this process in Python. However I wanted to see if it was possible to do this using only the command line and DuckDB's SQL. The outline of the process is as follows:
1. Use the OpenDataSoft API to extract the initial load of analytics data in parquet format.
2. Create a database on Motherduck to hold this initial data.
3. Write a [SQL script](https://github.com/stevecrawshaw/ods-analytics-etl/blob/3f230a61d5a116855cfaa1aa63a066d347534ddf/ods-analytics.sql) to load periodic downloads of the analytics data into the database.

This post will document the process in step 3.

I'm writing my SQL in a .sql file in VS code and have set up VS code so you can send commands direct to the terminal with `Ctrl + Enter`. This is a great way to work with DuckDB as you can run the SQL commands directly in the terminal and see the results immediately. You can also send terminal commands to bash, or as below nushell.

## Extracting the data
The analytics data can only be accessed with the right permissions. So an API key is required. I have my API keys in a config.yml (yaml) file in my home folder. DuckDB can't read yaml files directly, so I use [Nushell](https://www.nushell.sh/) to read the yaml file and convert to json (could probably do this with bash and jq). It has a nice concise syntax so you can just write something like this. 

`nu`
`cat ../config.yml | from yaml | to json | save ../config.json`

Once the json file is created the API key can be accessed in the SQL script and assigned to a variable for later use.

```sql
SET VARIABLE ods_apikey = (SELECT ods.apikey FROM read_json('../config.json'));
SELECT getvariable('ods_apikey') as ods_apikey;
```
To efficiently extract just the new data we need to know the date of the last data we already have. This is stored in a table called `ods_api_monitoring_tbl` in the DuckDB database. We can use this to filter the API request to only get data that is newer than the last date we have.

```sql
SET VARIABLE max_date = (SELECT max(timestamp) AS last_date FROM ods_api_monitoring_tbl);
SELECT getvariable('max_date') AS last_date; -- just to verify the date
```
We also need to create a URL to access the API. The URL is constructed using the base URL, the API key, and the last date. The API returns data in parquet format, so we can use the `read_parquet` function to read the data directly into DuckDB. This relies on the `HTTPFS` extension being installed in DuckDB.

This is pretty ugly and doesn't offer the fine - grained control that you would get with a Python script, but it works. The URL is constructed using the `getvariable` function to access the variables we have set.

```sql
SET VARIABLE url = (SELECT 'https://opendata.westofengland-ca.gov.uk/api/explore/v2.1/monitoring/datasets/ods-api-monitoring/exports/parquet?where=timestamp+%3E+date%27' || getvariable('max_date').strftime('%Y-%m-%d') || '%27&use_labels=false&apikey=' || getvariable('ods_apikey') AS url);
SELECT getvariable('url') AS url; -- just to verify the URL
```
Once we have the URL we can read the data directly into DuckDB using the `read_parquet` function. This will create a temporary table called `update_tbl` that holds the data.

```sql
CREATE OR REPLACE TEMP TABLE update_tbl AS
SELECT *
FROM read_parquet(
    getvariable('url')
);
```
From this point it is trivial to just insert the data into the main table. The `ods_api_monitoring_tbl` table is used to store the data, and the `timestamp` column is used to filter out any duplicate data.

```sql
INSERT INTO ods_api_monitoring_tbl BY NAME
SELECT * FROM update_tbl;
```
## Next steps
The next step is to schedule this SQL script to run regularly. I am thinking this could be run with GitHub Actions or a cron job. The script can be run with the `duckdb` command line tool, which will execute the SQL commands in the script file.