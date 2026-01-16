## Why Your AI Assistant Needs to Read Column Comments Before Querying Your Database

**TLDR; I asked Claude to find the "worst" environmental impact areas in our database. It confidently returned the BEST areas instead. The fix was a new skill that forces metadata verification before any analytical query. Here's what we learned about semantic errors and how to prevent them.**

## The Setup: When simple queries go wrong

I was working with Claude Code on analyzing Energy Performance Certificate (EPC) data. The task seemed straightforward:

> "Find the 5 LSOAs with the worst mean environmental impact for domestic properties"

Claude dutifully connected to the DuckDB database with [The Answering Service](https://motherduck.com/docs/key-tasks/ai-and-motherduck/mcp-workflows/), joined EPC data with geographic lookups using my [uk-geography skill](https://gist.github.com/stevecrawshaw/32578ef7cb3a7345ede1dd9765741865), aggregated by LSOA, and returned results. The query executed perfectly. The SQL was clean. The joins were correct.

**The results were completely backwards.**

## What Went Wrong: The Assumption Trap

Claude made a natural but fatal assumption: "environmental impact" must mean higher numbers = worse impact, right? More emissions, more impact, higher score.

So it sorted descending and returned the top 5 scores:

- Bath and North East Somerset 013E: 87.99
- South Gloucestershire 019H: 87.07
- South Gloucestershire 034E: 86.92

These were actually the BEST performing areas, not the worst.

## The Hidden Semantic Inversion

When I asked Claude to check the column comment, it queried the metadata:

```sql
SELECT column_name, data_type, comment
FROM duckdb_columns()
WHERE table_name = 'raw_domestic_epc_certificates_tbl'
    AND column_name = 'ENVIRONMENT_IMPACT_CURRENT';
```

The comment revealed:

> "The Environmental Impact Rating. A measure of the property's current impact on the environment in terms of carbon dioxide (CO₂) emissions. **The higher the rating the lower the CO₂ emissions.**"

Classic inverted scale. High score = good (low emissions). Low score = bad (high emissions).

This is exactly like energy efficiency ratings - an A-rated property is more efficient than a G-rated one, even though A comes before G in the alphabet.

## Silent Semantic Errors

Consider the consequences:

1. **Policy decisions based on bad data** - Targeting improvement programs at already-efficient areas
2. **Wasted resources** - Allocating funding to the wrong locations
3. **Missed opportunities** - Overlooking areas that genuinely need intervention
4. **Erosion of trust** - When analysts catch these errors, confidence in AI (and the Analyst!) drops

**The query executed perfectly.** No errors, no warnings, no indication anything was wrong. Just confidently incorrect results.

## The Pattern: When Column Names Lie

This problem appears everywhere:

| Column Name | Intuitive Meaning | Actual Meaning |
|-------------|-------------------|----------------|
| `risk_score` | Higher = more risk? | Could be inverted (higher = safer) |
| `quality_index` | Higher = better? | Could be inverted (higher = more issues) |
| `performance_rating` | Higher = better? | Depends on the rating system |
| `impact_measure` | Higher = worse? | Could measure positive impact |
| `deprivation_decile` | Higher = more deprived? | Often inverted (10 = least deprived) |

You can't tell from the column name alone. You need the metadata.

## The Solution: Metadata-First Workflow

We created a new [Claude Code skill](https://gist.github.com/stevecrawshaw/4c892e9c404714eabf8f683e23d37b24) called `duckdb-query-assistant` that enforces this workflow:

### Before Every Analytical Query

1. **Identify analytical columns** - Anything measuring ratings, scores, impacts, indices
2. **Query metadata first**:

   ```sql
   SELECT column_name, data_type, comment
   FROM duckdb_columns()
   WHERE table_name = 'target_table'
   ```

3. **Parse for semantic indicators**:
   - Scale direction: "higher is better" vs "lower indicates worse"
   - Units: tonnes/year, kWh, percentiles
   - Rating systems: 1-100, A-G, inverted scales
4. **If ambiguous, ASK THE USER** - Don't guess

### The Skill's Critical Rule

**NEVER make assumptions about column semantics from names alone.**

When the user asks for "worst" or "best" or "highest" or "lowest", the skill:

1. Identifies the analytical column
2. Checks its comment
3. If unclear, asks: "Does higher mean better or worse?"
4. Only then constructs the query

## Example: How the Skill Works

**User request:**
> "Find the 5 LSOAs with worst mean environmental impact"

**Skill response:**

```
I'll help find the LSOAs with worst environmental impact. First, let me check
the metadata for ENVIRONMENT_IMPACT_CURRENT to understand the scale direction.

[Queries duckdb_columns() for comment]

The comment shows: "The higher the rating the lower the CO₂ emissions"

This means:
- HIGH score = GOOD (low emissions)
- LOW score = BAD (high emissions)

So "worst" means LOWEST scores. I'll query for the 5 LSOAs with the
lowest mean environmental impact ratings.

[Proceeds with query using ORDER BY ASC]
```

**Enforcement level:** SUGGEST (not blocking)

- Provides guidance and warnings
- Asks clarifying questions
- Doesn't prevent queries, but highlights risks
- Educates about proper metadata workflow

Key features:

- Red flags analytical columns (`*_rating`, `*_impact`, `*_score`, etc.)
- Requires explicit user clarification when ambiguous
- Integrates with schema documentation system
- Shows worked examples of good vs bad patterns

## When to Skip Metadata Verification

The skill is smart about when NOT to require verification:

✅ **Skip for:**

- Identifiers (codes, IDs, keys)
- Dimensional attributes (names, labels, categories)
- Date/time fields (obvious semantic meaning)
- Simple counts (always higher = more)
- Boolean flags (is_active, has_feature)

⚠️ **Require for:**

- Any ratings, scores, or indices
- Impact/effect measures
- Quality/performance metrics
- Anything where "higher" or "lower" is ambiguous

## The Corrected Results

After fixing the sort direction:

| Rank | LSOA | Mean Impact | Properties |
|------|------|-------------|------------|
| 1 | Bath and North East Somerset 010H | **48.30** | 333 |
| 2 | Bath and North East Somerset 022F | **48.37** | 256 |
| 3 | South Gloucestershire 024F | **48.63** | 426 |
| 4 | South Gloucestershire 004C | **50.78** | 392 |
| 5 | North Somerset 013D | **51.15** | 399 |

These are genuinely the worst-performing areas (lowest scores = highest emissions), with adequate sample sizes for reliable statistics.

## Key Takeaways for AI-Assisted Data Analysis

1. **Column names are hints, not specifications** - Never assume semantics from names
2. **Inverted scales are everywhere** - Ratings systems love them
3. **Metadata is documentation** - Comments in `duckdb_columns()` are gold
4. **Silent failures are dangerous** - Syntactically correct ≠ semantically correct (bye bye career)
5. **Skills can enforce good practices** - Automate the metadata-first workflow
6. **When in doubt, ask** - Better to clarify than confidently err

## Use it for your Projects

If you're working with:

- **Government statistics** - Full of inverted indices (IMD deciles, for example)
- **Energy/environmental data** - Ratings vs actual consumption vs impact scores
- **Healthcare metrics** - Risk scores, quality measures, outcome ratings
- **Financial data** - Credit ratings, risk scores, performance indices
- **Any domain with ratings systems** - Usually inverted from intuitive meaning

- Train your AI assistants to check metadata first
- Document your schemas comprehensively
- Create skills/prompts that enforce verification
- All of the above

**A few seconds checking metadata prevents hours debugging wrong results.**

## Next Steps: Making This Standard Practice

We've now built this into our workflow as a reusable skill. Every time we query the database with analytical metrics, the skill activates and ensures metadata verification.

Other options:

- **SQL linters** - Check for ORDER BY on rating columns without comments
- **Query templates** - Require comment queries before aggregations
- **Code review** - Flag analytical queries missing metadata validation
- **Documentation standards** - Mandate clear scale direction in all comments

## Conclusion: Metadata is Not Optional

The difference between a good data analysis and a dangerously wrong one can be a single ORDER BY clause going the wrong direction.

Column names don't tell you enough. Assumptions are dangerous. Metadata is essential. DuckDB is life. Claude is alive!

**Read the comments. Verify the scales. Ask when uncertain.**

Your future self (and your stakeholders) will thank you.

---

## Resources

- [DuckDB System Tables Documentation](https://duckdb.org/docs/sql/information_schema)
- The `duckdb-query-assistant` [skill](https://gist.github.com/stevecrawshaw/4c892e9c404714eabf8f683e23d37b24): Available in `~/.claude/skills/duckdb-query-assistant/`
