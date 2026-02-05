---
title: "Mapping Broadband Gaps with the Open Data Portal API"
---

Working with large datasets often presents a choice: download everything locally and wrangle it yourself, or use an API to query exactly what you need. In this post, I'll walk through a practical example of using the Open Data Portal API to analyse broadband provision across Lower Layer Super Output Areas (LSOAs) in the West of England, demonstrating why APIs can be a powerful tool for dynamic spatial analysis.

## The Goal

We want to identify which LSOAs have poor broadband provision—specifically, areas where properties have neither current nor future gigabit broadband availability. Rather than downloading hundreds of thousands of records and processing them locally, we'll use the API to aggregate the data server-side and return only what we need.

## Why Use an API?

The BDUK (Broadband Delivery UK) dataset contains approximately 500,000 property records. By using the API's query capabilities, we can:

- Filter records based on specific criteria (no current or future gigabit)
- Aggregate data by LSOA using SQL-like syntax
- Retrieve only the summarised results we need
- Avoid downloading and processing massive files locally
- Enable reproducible, dynamic analysis that always uses current data

## The Setup

I'm using [Positron](https://github.com/posit-dev/positron), a modern IDE from Posit (formerly RStudio), but this workflow works equally well in standard RStudio. We'll need several R libraries:

```r
pacman::p_load(
  tidyverse,
  glue,
  httr2,      # API package
  sf,         # Spatial data
  tmap,       # Mapping
  tmaptools,
  cols4all
)
```

## Building the API Query

The Open Data Portal provides an [API console](https://opendata.westofengland-ca.gov.uk/api/explore/v2.1/console) where you can construct and test queries interactively. For this analysis, we're using the `records` endpoint with SQL-like clauses:

**SELECT**: We want three fields—local authority, LSOA code, and a count of premises:

```sql
SELECT local_authority, lsoa_code, count(uprn) AS premises
```

**WHERE**: Filter for properties without gigabit availability:

```sql
WHERE NOT current_gigabit AND NOT future_gigabit
```

**GROUP BY**: Aggregate the counts by geography:

```sql
GROUP BY local_authority, lsoa_code
```

Testing this in the API console returns a 200 response with JSON containing 491 aggregated records—much more manageable than the full dataset.

## Making the Request in R

The API console generates a request URL, but it's unwieldy to work with directly. The `httr2` package's `url_parse()` function helps us understand its structure:

```r
url_parse(
  "https://westofenglandca.opendatasoft.com/api/explore/v2.1/catalog/datasets/bduk-west-of-england/records?select=local_authority%2Clsoa_code%2Ccount%28uprn%29%20AS%20premises&group_by=local_authority%2Clsoa_code&where=NOT%20current_gigabit%20AND%20NOT%20future_gigabit&limit=20000&offset=0"
)
```

This reveals the components we can work with programmatically. Rather than hardcoding the full URL, we define variables for each part:

```r
# Universal (for all API calls in this script)
hostname <- "https://westofenglandca.opendatasoft.com"
path1 <- "api/explore/v2.1"
path2 <- "catalog/datasets"
limit <- 20000
offset <- 0

# Specific (to the BDUK dataset)
bduk_endpoint <- "records"
bduk_dataset <- "bduk-west-of-england"
bduk_select <- "local_authority,lsoa_code,count(uprn) AS premises"
bduk_groupby <- "local_authority,lsoa_code"
bduk_where <- "NOT current_gigabit AND NOT future_gigabit"
```

Now we can construct the request cleanly:

```r
bduk_response <- request(hostname) |>
  req_url_path_append(path1, path2, bduk_dataset, bduk_endpoint) |>
  req_url_query(
    "select" = bduk_select,
    "group_by" = bduk_groupby,
    "where" = bduk_where,
    "limit" = limit,
    "offset" = offset
  ) |>
  req_perform()
```

The response arrives quickly—the server has aggregated nearly 500,000 records and returned only the summary we need. We can inspect the URL that was constructed:

```r
bduk_response |> pluck("url")
```

## Processing the Response

The API returns JSON, which we need to convert into a data frame:

```r
bduk_no_gigabit_tbl <- bduk_response |>
  resp_body_json() |>
  pluck("results") |>
  bind_rows()
```

Inspecting `bduk_no_gigabit_tbl` shows exactly what we expect: local authority, LSOA code, and premises count for areas with poor broadband provision.

## Adding Spatial Data

To map this data, we need LSOA boundary geometries. The Open Data Portal provides these through the `exports` endpoint. We'll request them in FlatGeobuf (FGB) format, which is more efficient than GeoJSON:

```r
# Specific variables for this API call
lsoa_dataset <- "lep_lsoa_geog"
lsoa_select <- "lsoa21cd,geo_shape"  # Just these 2 fields; geo_shape is the spatial column
lsoa_format <- "fgb"                 # FlatGeoBuf format - efficient spatial representation
lsoa_endpoint <- "exports"           # Getting the file itself now

# Request spatial data
lsoa_sf <- request(hostname) |>
  req_url_path_append(path1, path2, lsoa_dataset, lsoa_endpoint, lsoa_format) |>
  req_url_query(
    "select" = lsoa_select,
    "limit" = limit,
    "offset" = offset
  ) |>
  req_perform() |>
  pluck("url") |>
  st_read()
```

The pipeline extracts the URL from the response and pipes it to `st_read()`, which creates a spatial features object. The `sf` package recognises this as spatial data with a WGS84 coordinate system (latitude/longitude).

## Joining and Mapping

With both datasets in hand, we perform a simple join on the LSOA code:

```r
lsoa_nogigabit_sf <- lsoa_sf |>
  inner_join(bduk_no_gigabit_tbl, by = join_by(lsoa21cd == lsoa_code))
```

Now we can create an interactive map with `tmap`:

```r
tmap_mode("view")
tm_basemap("OpenStreetMap") +
  tm_shape(lsoa_nogigabit_sf) +
  tm_polygons(
    fill = "premises",
    fill.scale = tm_scale_continuous(),
    fill_alpha = 0.7,
    col = "black",
    lwd = 0.5
  )
```

This produces an interactive map showing the distribution of broadband gaps across the region. Areas with darker colours have more premises without current or future gigabit availability—highlighting where investment or programmes might be needed.

## Key Takeaways

1. **APIs enable server-side processing**: Rather than downloading and aggregating 500,000 records locally, we let the server do the work and return only what we need.

2. **SQL-like queries provide flexibility**: The API's support for SELECT, WHERE, and GROUP BY clauses means we can perform complex aggregations without writing local code.

3. **Spatial formats matter**: Using FGB instead of GeoJSON significantly reduces payload size for boundary data.

4. **httr2 simplifies API work**: Breaking URLs into components makes the code more readable and maintainable than working with raw query strings.

5. **Reproducible by design**: This script can be re-run at any time to fetch the latest data, ensuring analysis stays current without manual downloads.

Whilst this example uses R, the same principles apply in Python or any language with HTTP and JSON capabilities. The Open Data Portal API provides a powerful way to work with large datasets dynamically—whether you're building dashboards, conducting ad-hoc analysis, or automating regular reports.

You can watch the full walkthrough on [YouTube](https://youtu.be/gPo831OP8M8) where I demonstrate each step interactively in Positron.
