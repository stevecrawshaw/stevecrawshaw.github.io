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

```r

blob_to_geopoint <- function(blob_col) {

#' Convert a column of WKB blobs to a vector of geopoint strings
#' @param blob_col A column of WKB blobs, typically from a `sf` object or geoparquet file
#' @return A character vector of geopoint strings in the format "{lat,long}"  as required for the  ODS rendering of points
#' @details This function uses `rlang::enquo` to capture the column name and `rlang::eval_tidy` to evaluate it.
#' The function is designed to be used within a `mutate` call in a `dplyr` pipeline.
#' to be used within a mutate function call
  bc <- enquo(blob_col)
# Convert the WKB blobs to simple features geometries
# The wkb blobs are represented as vectors in R
  geom_list <- map(rlang::eval_tidy(bc),
      ~as.raw(.x) |> 
        st_as_sfc(wkb = TRUE))
# set the length of the vector to the length of the geom_list
  gp_vec <- vector(mode = "character", length = length(geom_list))
# Iterate over the list to construct the geopoint strings
  gp_vec = map(geom_list,
               ~paste0("{",
                       st_coordinates(.x)[2],
                       ",",
                       st_coordinates(.x)[1],
                       "}"))
# Return the vector of geopoint strings
  unlist(gp_vec)
  
}

```r

