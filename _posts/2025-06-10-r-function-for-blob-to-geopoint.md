---
layout: post
title: Converting from WKB to Geopoint in R
date: 2025-06-10
categories: [R]
tags:
- R
- spatial
- sf
- opendatasoft
- geo
---

I always struggle to convert between binary formats in spatial data wrangling, and so here is an R function that converts a WKB (Well-Known Binary) representation of a geometry into a geopoint using the `sf` package. This is particularly useful when working with data from sources like OpenDataSoft, which often provides geometries in WKB format (from geoparquet export) and also expects point geometries in the form of `{lat,long}` when uploading data.

The tricks here are to convert the WKB blobs to "raw" first - they are stored as character vectors in R. Then they can be converted to simple features geometries using the `st_as_sfc()` function from the `sf` package, with the wkb = TRUE argument. Finally, we extract the coordinates and format them as geopoint with curly braces.

The function is designed to be used within a `mutate` call in a `dplyr` pipeline, allowing for easy integration into data manipulation workflows.

<img src="/images/wkb_geopoint.png">
