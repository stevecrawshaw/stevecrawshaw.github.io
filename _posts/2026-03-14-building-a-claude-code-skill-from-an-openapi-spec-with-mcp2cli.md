---
title: "Building a Claude Code skill from an OpenAPI spec with mcp2cli"
date: 2026-03-14
categories: [AI]
tags:
- claude
- mcp
- open-data
- cli
- skills
---

**TLDR; I turned WECA's Open Data portal API into a Claude Code skill using [mcp2cli](https://github.com/knowsuchagency/mcp2cli), scored a Grade D on the first attempt, and iterated until it actually worked properly. The skill lets Claude query 122 public datasets from the command line without an MCP server running.**

## The problem

WECA publishes [122 datasets](https://opendata.westofengland-ca.gov.uk) on air quality, greenhouse gas emissions, deprivation indices, biodiversity priorities, and more. The portal runs on OpenDataSoft and has a full [Explore v2.1 API](https://opendata.westofengland-ca.gov.uk/api/explore/v2.1/swagger.json) with a Swagger spec.

I wanted Claude Code to query this portal directly. The obvious route is an MCP server, but that means running a process, managing connections, and stuffing tool schemas into the system prompt on every turn whether the model uses them or not. I did actually build this a few months ago using a [rest-to-mcp](https://github.com/stevecrawshaw/rest-to-mcp-adapter) adapter, but I noticed the mcp2cli library recently so decided to experiment with this approach instead.

## Why CLI instead of MCP?

[mcp2cli](https://github.com/knowsuchagency/mcp2cli) does something simpler. It gives the LLM a CLI it can shell out to. The model runs `--list` to see what's available, `--help` to check parameters, and then executes commands with flags. Same way you or I would use any CLI.

The token difference is hard to ignore. A 30-tool MCP server burns ~3,600 tokens per turn just on schema overhead. mcp2cli's system prompt is 67 tokens. Over a 15-turn conversation using 5 tools, that's 96% fewer tokens. The mcp2cli README has [detailed benchmarks](https://github.com/knowsuchagency/mcp2cli#the-numbers-how-much-context-do-you-actually-save) if you want the full breakdown.

Token use is abviously important, but the overhead of setting up MCP on a per - project basis, or remembering to disable a global implementation is not ideal. A skill and CLI appproach is just more lightweight.

Beyond tokens: no server process to keep alive, works with any LLM that can run shell commands, and when the API adds new endpoints they just appear on the next call. No rebuild, no codegen.

## Generating the skill

mcp2cli can generate skills from API specs. One command:

```
mcp2cli create a skill for https://opendata.westofengland-ca.gov.uk/api/explore/v2.1/swagger.json
```

Out came a SKILL.md with the base command, endpoints, parameters, and example workflows. Functional, but generic. It listed everything in the Swagger spec without knowing which bits actually matter for this API. Like an auto-generated man page: technically complete, but not foolproof.

## The skill-judge verdict: Grade D

I ran `/skill-judge` against it. This skill evaluates Claude Code skills on eight dimensions including knowledge delta (does it teach the model something it can't get from `--help`?), anti-pattern documentation, and progressive disclosure.

Score: 72/120. Grade D.

| Dimension | Score | What was wrong |
|-----------|-------|----------------|
| Knowledge Delta | 9/20 | Most content was just command syntax, already in `--help` |
| Anti-Pattern Quality | 3/15 | Zero gotchas documented |
| Mindset + Procedures | 8/15 | Decent workflow order, no decision framework |
| Practical Usability | 13/15 | The commands did work |

The most useful finding came from live testing during the evaluation, not from the rubric itself. The API quietly types fields called `year` and `calendar_year` as `date`, not `text`. Filter with a plain string and you get a cryptic error:

```bash
# Fails: IncompatibleTypesInComparisonFilter
--where "year = '2022'"

# What you actually need
--where "year = date'2022'"
```

This trips up every first-time user. The original skill said nothing about it.

## What I changed

I stripped out 30 lines of reference tables (Available Commands, Key Parameters) that duplicated what `--help` already provides. Then added the stuff that's actually hard to figure out on your own:

A "Before Querying Any Dataset" checklist. Four steps: find the dataset ID, inspect the schema (and check field types, because of the date thing), decide sample vs full export, check whether the data is WECA-only or national scope. I keep coming back to this last one. The GHG emissions dataset contains every English local authority, not just the West of England. Without a filter on `cauthnm = 'West of England'`, you get Birmingham and Leeds mixed in. Nothing in the API tells you this.

A "NEVER / Gotchas" section with six anti-patterns: the date type mismatch, field name guessing (dataset naming is wildly inconsistent), CSV semicolon defaults, national scope assumptions, invalid `--select` on catalogue queries, and the missing `--raw` flag.

A table of eight commonly used datasets with scope annotations, so the model doesn't have to run a discovery query every session.

And an export command comparison table, because `export-records-csv` supports `--delimiter` but not `--where`, while `export-records` supports `--where` but not `--delimiter`. There's no single command that does filtered, comma-separated CSV export. Annoying, but at least it's documented now.

The skill shrank from 160 to 148 lines while the ratio of genuinely useful knowledge went up.

## Testing it for real

I ran five tests against live data:

| Test | What it checked | Result |
|------|-----------------|--------|
| 1 | Dataset discovery and metadata inspection | Pass |
| 2 | Date-filtered query with CSV export | Partial pass |
| 3 | Aggregation with group by | Pass |
| 4 | Sample data as formatted table | Pass |
| 5 | Facet browsing and filtered query | Pass |

The partial pass on Test 2 is the `export-records-csv` vs `export-records` gap I mentioned. No single command covers both filtering and delimiter control. I documented the workaround rather than pretending the problem doesn't exist.

Test 4 turned up something else I hadn't expected: datasets with `geo_shape` polygon fields produce about 200 KB per record. Ask for 5 rows without `--select` and you get a megabyte of GeoJSON polygon coordinates. Not a bug, but worth warning about.

Here's what a real query looks like. GHG emissions by sector for the West of England, 2021:

```bash
uvx mcp2cli --spec https://opendata.westofengland-ca.gov.uk/api/explore/v2.1/swagger.json \
  get-records --dataset-id "ca_la_ghg_emissions_sub_sector_ods_vw" \
  --select "la_ghg_sector, sum(territorial_emissions_kt_co2e) as total_kt" \
  --group-by "la_ghg_sector" \
  --where "calendar_year = date'2021' AND cauthnm = 'West of England'" \
  --order-by "total_kt desc" --limit 20 --pretty
```

Transport came back at 2,172 kt CO2e, Domestic at 1,472, and LULUCF at -40 (carbon sequestration, so negative). Public API, no auth, no server process.

## The skill

Full SKILL.md is on [GitHub as a Gist](https://gist.github.com/stevecrawshaw/ff29a73158d08aa8c037b648117833d6). To use it, copy to `~/.claude/skills/weca-opendata/SKILL.md` and invoke with `/weca-opendata` in Claude Code. You could of course modify it to work for any other [Huwise open data portal](https://www.huwise.com/en/data-marketplace-solution/) quite simply - and Huwise have their own [MCP and AII agents](https://www.huwise.com/en/mcp-ai-agents/) offering.

It covers dataset discovery, schema inspection, the "before querying" checklist, six anti-patterns, record queries with filtering and aggregation, facet browsing, export workflows for CSV and Parquet, ODSQL syntax, and eight commonly used datasets.

## What I'd do differently

The auto-generated skill was a reasonable starting point but I spent more time fixing it than I would have spent writing it from scratch. The value of generation is the structure and the boilerplate, not the content. Next time I'd generate the skeleton and immediately start layering in domain knowledge rather than treating the output as near-finished.

The skill-judge evaluation was worth doing. It caught the missing anti-patterns and the knowledge-delta problem (too much reference, not enough expertise). But the operational testing caught different things: the export command gap, the geo_shape bloat, Windows encoding issues. Both were needed. Desk review and live testing find different classes of problem.

There's an ongoing [debate](https://www.reddit.com/r/mcp/comments/1rrviz4/perplexity_drops_mcp_cloudflare_explains_why_mcp/) about the value of the MCP approach vs. more lightweiight approaches to agent tools - don't know whether CLI-based tool access will hold up as APIs get more complex. For a public read-only portal with no auth, it's ideal. I'd be less confident about APIs that need OAuth flows or streaming responses. But for the analytical query use case, the trade-off (slightly more latency on first call, no streaming) is easy to accept.

## Resources

- [mcp2cli](https://github.com/knowsuchagency/mcp2cli)
- [WECA Open Data portal](https://opendata.westofengland-ca.gov.uk)
- [weca-opendata SKILL.md](https://gist.github.com/stevecrawshaw/ff29a73158d08aa8c037b648117833d6)
- [OpenDataSoft Explore API v2.1](https://help.opendatasoft.com/apis/ods-explore-v2/)
- [Claude Code skills documentation](https://docs.anthropic.com/en/docs/claude-code/skills)
