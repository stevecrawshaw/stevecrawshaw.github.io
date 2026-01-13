---
layout: post
title: Building a Regional Analytics Local Lakehouse with DuckDB
date: 2026-01-11
categories: [Data Engineering, Geospatial, AI, MCP]
tags:
- DuckDB
- Motherduck
- Data Lakehouse
- Medallion Architecture
- Schema Documentation
- MCP
- Claude Code
- UK Geography
- EPC Data
---

# Building a Regional Analytics Local Lakehouse with DuckDB

Over the past few months, I've been building a standalone analytical platform for UK Local Government environmental data using DuckDB. The project implements a Medallion Architecture (Bronze → Silver → Gold layers) and combines geospatial boundaries and lookups, API streams, and manual datasets into a unified lakehouse database on the local filesystem - not for external publishing.

The end purpose is to enable fast local analysis of key environmental and geographical data using a modern OLAP database. Analysis could include comparisons across other combined authorities, within a specific combined authority, or to support reporting of regional indicators in an outcomes framework. Pre - built queries in the Gold layer will facilitate reproducible anlaytical pipelines (RAP).

## The Database: Key Tables and Architecture

The database (`mca_env_base.duckdb`, ~7GB) consolidates environmental and geographic data for the West of England region across three transformation layers:

**Bronze Layer** - Raw data tables reading directly from landing zone files:

- `raw_domestic_epc_certificates_tbl` / `raw_non_domestic_epc_certificates_tbl` - Energy Performance Certificates (19m UK properties - latest EPCs only)
- `open_uprn_lep_tbl` - Unique Property Reference Numbers with point geometries
- `lsoa_2021_lep_tbl` - Lower Layer Super Output Area boundaries
- `postcode_centroids_tbl` - ONS Postcode Directory with geographic hierarchy lookups
- `eng_lsoa_imd_tbl` - Indices of Multiple Deprivation
- PostGIS federation for corporate boundary data (Combined Authorities, Local Authorities)

**Silver Layer** - Cleaned, standardized views:

- `epc_domestic_vw` / `epc_non_domestic_vw` - Cleaned EPC data with tenure, construction epochs
- `epc_domestic_lep_vw` / `epc_non_domestic_lep_vw` - Spatial subset for republishing as open data
- `ca_la_lookup_inc_ns_vw` - CA/LA boundary lookups with spatial unions and including North Somerset (for WECA)
- All spatial data reprojected to EPSG:27700 (British National Grid)

**Gold Layer** - Analytics-ready views (in development):

- Aggregated energy efficiency by LSOA/LA
- Property stock analysis by construction age band
- Emissions trends with geographic breakdowns

## Three Powerful Features

### 1. Schema Documentation in Comments

The database schema is fully documented using DuckDB's native `COMMENT ON TABLE/COLUMN` syntax. This makes metadata immediately available to AI assistants and BI tools without external documentation files. This is important to help AI agents understand the semantics, for example of ENVIRONMENT_IMPACT_CURRENT - does a high number imply a big environmental impact or a small environmental impact? Encoding this meaning in the comment helps the AI understand natural language queries:

```sql
SELECT column_name, data_type, comment
FROM duckdb_columns()
WHERE table_name = 'raw_domestic_epc_certificates_tbl'
  AND comment IS NOT NULL;
```

**Output:**

```
column_name              | data_type | comment
-------------------------+-----------+------------------------------------------
LMK_KEY                  | VARCHAR   | Individual lodgement identifier. Guaranteed
                         |           | to be unique and can be used to identify a
                         |           | certificate in the downloads and the API.
UPRN                     | BIGINT    | Unique Property Reference Number
CURRENT_ENERGY_RATING    | VARCHAR   | Current energy efficiency rating (A-G scale)
CONSTRUCTION_AGE_BAND    | VARCHAR   | Age band when the property was built
```

### 2. Interactive Schema Comment Editor

I built a Rich TUI-based schema documentation system that intelligently infers column descriptions using pattern matching and allows interactive review and editing:

**Pattern matching examples:**

- Suffix patterns: `_cd` → "code", `_nm` → "name", `_dt` → "date"
- Prefix patterns: `current_` → "Current value of", `total_` → "Total"
- Exact matches: `uprn` → "Unique Property Reference Number", `lsoa` → "Lower Layer Super Output Area code"

The workflow combines:

1. **XML schemas** - Import canonical metadata from curated sources
2. **Pattern inference** - Auto-generate descriptions with confidence scoring
3. **Interactive editing** - Review and refine in a terminal UI
4. **View mapping** - Auto-propagate comments from base tables to dependent views
5. **SQL generation** - Output `COMMENT ON` statements

```bash
# Launch interactive editor
uv run python -m src.tools.schema_documenter edit-comments \
    -d data_lake/mca_env_base.duckdb

# Auto-saves progress to .schema_review_session.json
# Exports to manual_overrides.xml (highest priority)
```

### 3. Incremental EPC Updates

EPC certificates are updated monthly. The EPC API incremental update system automatically fetches new certificates daily using cursor-based pagination:

```bash
# Update both domestic and non-domestic certificates
uv run python -m src.extractors.epc_incremental_update all -v
```

**How it works:**

1. Query database for latest `LODGEMENT_DATE` in target table
2. Fetch certificates from EPC API since that date (5,000 records/page)
3. Normalize columns (lowercase → UPPERCASE) and deduplicate by UPRN
4. Stage to CSV (`data_lake/staging/epc_domestic_incremental_{date}.csv`)
5. Atomic upsert using `MERGE INTO` (insert new, update existing)

**Typical output:**

```
Date range: 2025-01-02 to 2025-01-07
Fetched 4523 records
Deduplicated to 4501 unique UPRNs (removed 22 duplicates)
MERGE completed: 4489 inserted, 12 updated
```

## The UK Geography Skill

To make geographic analysis easier, I created a comprehensive `uk-geography` [skill for Claude Code](https://gist.githubusercontent.com/stevecrawshaw/32578ef7cb3a7345ede1dd9765741865/raw/c095514c90d3f59bc7e0a825f9d6cd2251685455/SKILL.md). This skill documents:

- **Statistical geographies**: OAs, LSOAs, MSOAs with population thresholds
- **GSS codes**: 9-character Government Statistical Service codes
- **Geographic hierarchies**: How areas nest (OA → LSOA → MSOA → LAD → Combined Authority)
- **UPRN/USRN**: Unique Property/Street Reference Numbers
- **Lookup patterns**: Common joins (Postcode → OA → LSOA → LAD)
- **WECA-specific geography**: West of England Combined Authority boundaries

**Key reference data:**

```
Output Area (OA)           ~100-625 people
  ↓ nest into
Lower Layer Super Output Area (LSOA)   ~1,500 people
  ↓ nest into
Middle Layer Super Output Area (MSOA)  ~7,500 people
  ↓ nest into
Local Authority District (LAD)
```

## AI-Assisted SQL with MotherDuck MCP + UK Geography Skill

Here's where it gets powerful. By combining:

1. **[MotherDuck MCP server](https://github.com/motherduckdb/mcp-server-motherduck)** - Database introspection and query execution
2. **[UK Geography skill]** - Geographic hierarchy knowledge for Claude code
3. **Schema comments** - Self-documenting database metadata

Claude Code can write complex spatial SQL queries from natural language prompts.

**Example prompt:**
> "Create a view that adds LSOA codes to the EPC domestic data using spatial joins on UPRN locations"

Claude:

1. Calls the `uk-geography` skill to understand LSOA codes and UPRN-to-LSOA lookup patterns
2. Uses MotherDuck MCP to introspect available tables and columns
3. Identifies the join strategy: EPC → UPRN geometries → LSOA polygons via `ST_Within()`
4. Writes and tests the SQL

**Generated SQL** (`lsoa_epc_domestic_vw.sql`):

```sql
-- View: lsoa_epc_domestic_vw
-- Purpose: Extends epc_domestic_lep_vw with LSOA 2021 codes via spatial join
-- Dependencies:
--   - epc_domestic_lep_vw (EPC certificates with UPRN)
--   - open_uprn_lep_tbl (UPRN point geometries)
--   - lsoa_2021_lep_tbl (LSOA 2021 polygon geometries)

CREATE OR REPLACE VIEW lsoa_epc_domestic_vw AS
SELECT
    e.*,
    l.lsoa21cd AS LSOA21CD,
    l.lsoa21nm AS LSOA21NM
FROM epc_domestic_lep_vw e
LEFT JOIN open_uprn_lep_tbl u ON e.UPRN = u.uprn
LEFT JOIN lsoa_2021_lep_tbl l ON ST_Within(u.shape::GEOMETRY, l.shape::GEOMETRY)
WHERE e.UPRN IS NOT NULL;
```

**Verification query:**

```sql
-- Aggregate EPC energy efficiency by LSOA
SELECT
    LSOA21CD,
    LSOA21NM,
    COUNT(*) as property_count,
    ROUND(AVG(CURRENT_ENERGY_EFFICIENCY), 1) as avg_energy_efficiency,
    ROUND(AVG(TOTAL_FLOOR_AREA), 1) as avg_floor_area_m2
FROM lsoa_epc_domestic_vw
GROUP BY LSOA21CD, LSOA21NM
ORDER BY property_count DESC
LIMIT 10;
```

**Results:**

```
LSOA21CD  | LSOA21NM                      | property_count | avg_energy_efficiency
----------+--------------------------------+----------------+----------------------
E01015014 | South Gloucestershire 006A    |      2102      |        81.3
E01032515 | Bristol 053F                  |      1436      |        75.5
E01035216 | North Somerset 027G           |      1215      |        83.2
E01014370 | Bath and North East Somerset  |      1204      |        64.3
```

**Match rate: 99.2%** - Out of 372,734 unique UPRNs, 369,748 successfully joined to LSOA codes using the spatial join.

I was impressed that this query was zero - shotted and anticipate that more advanced queries would be possible with the right prompting approaches.

## Why This Matters

This approach demonstrates how AI-assisted development can accelerate geospatial data engineering:

1. **Self-documenting schema** - Comments travel with the data, accessible to both humans and AI
2. **Domain knowledge as skills** - Complex geographic concepts encoded once, reused forever
3. **MCP integration** - Database becomes an interactive environment for exploration
4. **Spatial reasoning** - AI understands joins like "point-in-polygon" and suggests appropriate DuckDB Spatial (PostGIS) functions.

The combination eliminates the need to:

- Look up ONS geography codes manually
- Remember GSS code structures
- Consult PostGIS documentation for spatial predicates
- Navigate complex geographic hierarchies

Instead, I can ask questions like "Which LSOAs have the worst energy efficiency ratings?" and Claude translates this into working SQL using the documented schema, geographic knowledge, and spatial functions.

## Tech Stack

- **DuckDB 1.4.0+** with SPATIAL extension
- **Python 3.13+** with `uv` package manager
- **Click** + **Rich** for CLI/TUI
- **Pydantic** for data validation
- **lxml** for XML schema parsing
- **httpx** for EPC API client

## Next Steps

Phase 4 (Gold Layer) will migrate analytics views from legacy SQL into the orchestrated transformation system, completing the medallion architecture with aggregated metrics ready for BI tools, open data exports and corporate reporting functions.

The full project is available at: [github.com/stevecrawshaw/data-lake](https://github.com/stevecrawshaw/data-lake)

---

*This post demonstrates how modern AI tools (Claude Code), database design (self-documenting schemas), and domain-specific skills (UK geography) combine to accelerate analytical data platform development.*
