---
title: "Building an Interactive ERD for a MotherDuck Database with Claude MCP"
---

**TLDR; By wiring up the MotherDuck MCP server to Claude Code, I was able to introspect a whole database schema in seconds and auto-generate an interactive schema explorer — complete with a relationship diagram, column browser, and SQL snippet generator — without writing a line of code manually.**

## The Setup

Our `mca_data` database on [MotherDuck](https://motherduck.com) holds the core data assets for the West of England Combined Authority (WECA): EPC certificates, greenhouse gas emissions, deprivation indices, housing tenure, boundary geometries, and the geography lookup tables that tie them all together. It has grown organically and contains 17 tables and 7 views across 5 thematic domains.

I always struggle to remember precisely what's in it and what the tables and views are called. There was no ERD, no central schema reference. Just column comments and the knowledge in people's heads.

I wanted to fix that without spending an afternoon building a documentation site.

## The MotherDuck MCP Server

[Model Context Protocol (MCP)](https://docs.anthropic.com/en/docs/agents-and-tools/mcp) is Anthropic's open standard for connecting AI assistants to external tools and data sources. MotherDuck ships an MCP server that exposes your cloud DuckDB databases directly to Claude — no copy-pasting connection strings, no exporting schemas to text files.

With the server configured in `~/.claude/claude_desktop_config.json`, Claude gains a set of database tools it can call autonomously:

| Tool | What it does |
|------|-------------|
| `list_databases` | Enumerate all databases in your account |
| `list_tables` | List tables and views (with comments) for a database |
| `list_columns` | Get column names, types, and comments for a table |
| `query` | Execute read-only SQL against MotherDuck |
| `search_catalog` | Fuzzy-search across the full object catalog |

## The Workflow

The session followed a straightforward pattern that would have taken hours manually (contrast to the [workflow](https://stevecrawshaw.github.io/sql/2025/05/30/Workflow-for-diagrams-with-ai.html) I used previously):

**One shot prompt to make an interactive ERD**

```
use the motherduck MCP to connect to mca_data database.
Introspect the database schema and create a playground to
help me explore the tables and views and their relationships.
```

Claude called `list_tables` once, then fanned out to 24 parallel `list_columns` calls — one for each table and view — collecting types, comments, and key column information simultaneously.

**2. Relationship inference from shared key columns**

With all the column metadata in hand, Claude identified join relationships by matching key column names across tables:

- `lsoa21cd` / `lsoa21_code` — connecting boundary tables, IMD data, tenure data, postcode lookups
- `ladcd` / `local_authority_code` — linking the CA-to-LA hierarchy to GHG emissions tables
- `CAUTH25CD` / `cauthcd` — joining authority boundary tables to the lookup views
- `UPRN` — connecting EPC certificate tables to the property reference dataset
- `POSTCODE` — threading EPC data through postcode and boundary lookups

View derivation chains were also mapped: `raw_domestic_epc_certificates_tbl` → `epc_domestic_vw` → `epc_domestic_lep_vw`, and so on.

**3. Auto-generated playground**

All of that schema knowledge was compiled into a single self-contained HTML playground (no external dependencies, no build step) using the [playground skill](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/playground). The result is embedded below.

## The Schema Explorer

Explore the full `mca_data` schema interactively:

<div style="border: 1px solid #2e3248; border-radius: 8px; overflow: hidden; margin: 1.5rem 0;">
  <iframe
    src="/demos/mca_schema_explorer.html"
    width="100%"
    height="700"
    frameborder="0"
    style="display: block;"
    title="mca_data Schema Explorer">
  </iframe>
</div>

*Can't see the embed? [Open the full explorer](/demos/mca_schema_explorer.html) in a new tab.*

The playground has two main sections:

**Schema Browser**
- All 24 objects listed in the left sidebar, grouped by theme (Boundaries, Geography/Lookup, EPC/Energy Performance, GHG Emissions, Deprivation/Demographics)
- Views are distinguished from base tables with a teal badge
- Clicking any item shows its description, column count, and join keys in the right panel
- The **Columns tab** shows a filterable, searchable column list — join key columns are highlighted in gold
- The **SQL Snippet tab** auto-generates a `SELECT` query with configurable `LIMIT` and a "key columns only" toggle

**Relationship Diagram**
- A draggable SVG node graph with all 24 objects
- Solid lines = join relationships, labelled with the connecting key column
- Dashed lines = view derivation / composition relationships
- Legend lets you toggle theme groups on/off to reduce clutter
- Click a node to select it and sync back to the Schema Browser

## Database Structure at a Glance

Here's the full object inventory:

### Boundaries (6 tables, 1 view)

| Object | Type | Description |
|--------|------|-------------|
| `bdline_ua_lep_diss_tbl` | table | WECA LEP dissolved boundary |
| `bdline_ua_lep_tbl` | table | LEP UA unitary authority boundaries |
| `bdline_ua_weca_diss_tbl` | table | WECA dissolved boundary |
| `bdline_ward_lep_tbl` | table | Ward boundaries for WECA LEP |
| `ca_boundaries_bgc_tbl` | table | Combined authority boundaries — all England |
| `lsoa_2021_lep_tbl` | table | LSOA 2021 boundaries for WECA LEP |
| `ca_boundaries_inc_ns_vw` | view | CA boundaries including North Somerset |

### Geography / Lookup (5 tables, 2 views)

| Object | Type | Description |
|--------|------|-------------|
| `boundary_lookup_tbl` | table | Postcode → OA / LSOA / MSOA / LAD hierarchy |
| `ca_la_lookup_tbl` | table | Combined authority → local authority |
| `codepoint_open_lep_tbl` | table | Codepoint Open postcodes for LEP |
| `postcode_centroids_tbl` | table | Full postcode centroids (60 columns) |
| `open_uprn_lep_tbl` | table | UPRN coordinates for WECA LEP |
| `ca_la_lookup_inc_ns_vw` | view | CA→LA lookup including North Somerset |
| `weca_lep_la_vw` | view | WECA LEP local authorities |

### EPC / Energy Performance (2 tables, 3 views)

| Object | Type | Description |
|--------|------|-------------|
| `raw_domestic_epc_certificates_tbl` | table | Domestic EPC register (93 columns) |
| `raw_non_domestic_epc_certificates_tbl` | table | Non-domestic EPC register (41 columns) |
| `epc_domestic_vw` | view | Domestic EPC + computed fields |
| `epc_domestic_lep_vw` | view | Domestic EPC filtered to WECA LEP |
| `epc_non_domestic_lep_vw` | view | Non-domestic EPC filtered to WECA LEP |

### GHG Emissions (2 tables, 1 view)

| Object | Type | Description |
|--------|------|-------------|
| `la_ghg_emissions_tbl` | table | LA GHG emissions — long/tidy format |
| `la_ghg_emissions_wide_tbl` | table | LA GHG emissions — wide format |
| `ca_la_ghg_emissions_sub_sector_ods_vw` | view | Emissions by Combined Authority |

### Deprivation / Demographics (2 tables)

| Object | Type | Description |
|--------|------|-------------|
| `eng_lsoa_imd_tbl` | table | IMD 2019 — deciles, ranks, scores for all domains |
| `uk_lsoa_tenure_tbl` | table | LSOA housing tenure breakdown |

## Key Join Patterns

For anyone querying this database, these are the joins you'll reach for most often:

```sql
-- Enrich LSOA-level analysis with deprivation data
SELECT
  b.lsoa21cd,
  b.ladnm,
  imd.imd_decile,
  imd.employment_decile
FROM boundary_lookup_tbl b
JOIN eng_lsoa_imd_tbl imd ON b.lsoa21cd = imd.lsoa21_code;

-- EPC certificates with Combined Authority context
SELECT
  epc.CURRENT_ENERGY_RATING,
  epc.CONSTRUCTION_AGE_BAND,
  ca.cauthnm
FROM epc_domestic_lep_vw epc
JOIN boundary_lookup_tbl b ON epc.POSTCODE = b.pcds
JOIN ca_la_lookup_inc_ns_vw ca ON b.ladcd = ca.ladcd;

-- GHG emissions summarised by Combined Authority
SELECT
  cauthnm,
  calendar_year,
  la_ghg_sector,
  SUM(territorial_emissions_kt_co2e) AS total_kt_co2e
FROM ca_la_ghg_emissions_sub_sector_ods_vw
GROUP BY ALL
ORDER BY calendar_year, total_kt_co2e DESC;
```

## What This Demonstrates

A few things stood out from this session:

**MCP makes database introspection trivial.** The entire schema — 24 objects, ~300 columns — was fetched in a few seconds with parallel tool calls. No manual `SHOW TABLES` sessions, no schema dump files to maintain.

**Column comments pay dividends.** Because the `mca_data` tables have well-maintained column comments in DuckDB (set via `COMMENT ON COLUMN`), Claude could surface not just names and types but actual meaning — distinguishing, for example, that `lsoa21_code` in `eng_lsoa_imd_tbl` is the same key as `lsoa21cd` in `boundary_lookup_tbl` even though the names differ.

**The playground skill is a force multiplier.** Rather than describing the schema in prose, the entire introspection output was compiled into an interactive tool that anyone on the team can use. One session → living documentation.

**Self-contained HTML is underrated for internal tooling.** No server, no build pipeline, no dependencies. The explorer is a single file that can be emailed, committed to a repo, or hosted on GitHub Pages.

## Resources

- [MotherDuck MCP documentation](https://motherduck.com/docs/key-tasks/ai-and-motherduck/mcp-workflows/)
- [Model Context Protocol](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)
- [DuckDB column comments](https://duckdb.org/docs/sql/statements/comment_on.html)
- [Full schema explorer](/demos/mca_schema_explorer.html)
