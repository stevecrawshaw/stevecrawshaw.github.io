---
layout: post
title: CRS transformation in Duckdb SPATIAL
date: 2025-09-17
categories: [SQL, spatial, GIS]
tags:
- sql
- duckdb
- CRS
- geometry
- geojson
---

After many frustrating encounters with this issue I have decided to deep dive into it and document my findings. 

The issue is that when importing data from various sources (corporate POSTGIS database, flat files) in British National Grid (EPSG:27700) and then trying to convert to WGS84 (EPSG:4326), the data is often misplaced off the cost of East Africa. The key issue being that the order of co - ordinates is wrong because different systems define the order of co - ordinates differently.

- EPSG:27700 (British National Grid): This is straightforward. The axes are always Easting (X) followed by Northing (Y). No ambiguity here.

- EPSG:4326 (WGS 84): This is the tricky one. The official definition by the standards body specifies the order as Latitude (Y) followed by Longitude (X). However, a huge number of geospatial tools, databases, and formats—most importantly the GeoJSON standard—expect the order to be Longitude (X) followed by Latitude (Y).

DuckDB's ST_Transform function, by default, respects the official CRS definition. When it transforms from EPSG:27700 (X, Y) to EPSG:4326, it produces the officially correct but practically problematic (Y, X) or (Latitude, Longitude) output. So, it's important to note that **inside duckdb the spatial representation is correct**, and all spatial operations can be conducted on the data without flipping the co - ordinates.

When this (lat, lon) data is written to a GeoJSON file, a reader expecting (lon, lat) will interpret the latitude as longitude and vice versa, causing the geometry to be flipped and misplaced. **Essentially this means that the order of co - ordinates needs only to be changed when the data is exported.**

So I did some testing of various options to work this out. The SQL file below attaches to our corporate database and tests some options for CRS (Coordinate Reference System) conversion. Firstly I discovered that using the `always_xy := true` parameter will correctly order the vertex co - ordinates, when exporting to GeoJSON etc. **This is the optimum solution**.

The other way of achieving this is to apply a matrix transformation using ST_Affine() - which is technically correct but less elegant IMO.

Of course if the source data is already in EPSG:4326 and in the correct geoJSON then it's simply a matter of importing using ST_Read() and if you have done `LOAD HTTPFS` you can do it directly from the source URL, in this case the [open data portal](https://opendata.westofengland-ca.gov.uk/explore/dataset/lep-boundary/map/?location=10,51.47564,-2.68359&basemap=jawg.streets).

Finally, and further down the rabbit hole, I learned about the fine - tuning and precise approach to projection conversion achieved by using [OSTN NTv2 grid files](https://www.ordnancesurvey.co.uk/geodesy-positioning/coordinate-transformations/resources) to ensure that the projection to and from WGS84
 and British National Grid is as precise as possible. The difference is typically less than a metre but potentially significant for legal and boundary type issues.

```sql
-- copy the secrets file to the secrets folder (sometimes it needs to be moved - bug)
cp ~/weca_postgres.duckdb_secret ~/.duckdb/stored_secrets

duckdb
LOAD SPATIAL;
LOAD HTTPFS;
-- connect to corp DB VPN on
ATTACH '' AS weca_postgres (TYPE POSTGRES, SECRET weca_postgres);
-- examine schema
SELECT * FROM weca_postgres.information_schema.tables;

-- test various options for transformations of co - ordinates when converting between BNG (EPSG:27700) and
-- WGS84 (EPSG:4326) - lat and long

-- This syntax produces positionally correct geoJSON. Note the walrus operator for the 
-- always_xy parameter
-- https://duckdb.org/docs/stable/core_extensions/spatial/functions#st_transform

-- Ingest the data using the ST_Transform() function to change CRS (no need for always_xy := true here)
CREATE OR REPLACE TABLE lep_boundary_tbl AS
SELECT ST_GeomFromWKB(shape).ST_Transform('EPSG:27700', 'EPSG:4326') geometry
FROM weca_postgres.os.bdline_ua_lep_diss;

-- # but for exporting we need to flip the co - ordinate order
COPY (
    SELECT 
        -- "Transform" to the same CRS just to apply the axis-order flag
        ST_Transform(geom, 'EPSG:4326', 'EPSG:4326', always_xy := true) AS geometry,
        * EXCLUDE geom
    FROM 
        weca_postgres.os.bdline_ua_lep_diss
) 
TO 'data/exported_boundary_correct_order.geojson' 
WITH (FORMAT GDAL, DRIVER 'GeoJSON');

-- Using ST_Affine() applies a matrix transformation to the coordinates, which also has the same effect, ie
-- it swaps the order of the coordinates for each vertex
COPY (
    SELECT 
        -- Apply the affine transformation to swap x and y for each vertex
        ST_Affine(geom, 0, 1, 1, 0, 0, 0) AS geometry,
        -- Select any other columns you need
        * EXCLUDE geom
    FROM 
        weca_postgres.os.bdline_ua_lep_diss
) 
TO 'data/exported_boundary_correct_order.geojson' 
WITH (FORMAT GDAL, DRIVER 'GeoJSON');

-- Reading the geojson export from the Open Data Portal export endpoint
-- https://opendata.westofengland-ca.gov.uk/api/explore/v2.1/catalog/datasets/lep-boundary/exports/geojson?lang=en&timezone=Europe%2FLondon
-- or geoJSON file renders the layer correctly, without the need for transformation.
FROM ST_Read('data/lep-boundary_ods.geojson');


--  this is from the duckdb guidance on ST_Transform() and relates to using grid files for more 
-- precise conversion between CRSs. 
-- need to download, unzip and reference these files
-- https://www.ordnancesurvey.co.uk/documents/resources/OSTN15-NTv2.zip
-- note the full file path is needed
CREATE OR REPLACE TABLE lep_boundary_precise_tbl AS
SELECT ST_GeomFromWKB(shape).ST_Transform('+proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +ellps=airy +units=m +no_defs +nadgrids=C:\\Users\\steve.crawshaw\\OSTN15-NTv2\\OSTN15_NTv2_OSGBtoETRS.gsb +type=crs',
        'EPSG:4326') AS geometry
FROM weca_postgres.os.bdline_ua_lep_diss;


```