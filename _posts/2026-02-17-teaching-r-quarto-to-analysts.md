---
title: "Designing an R and Quarto Course for Analysts"
---

I've built a 5-session [training course](https://r-quarto-learning.vercel.app/) to move our analysis team at WECA from Excel and PowerPoint to R, Quarto, and Git. The goal is specific: each analyst needs to independently produce indicator chapters for our regional priorities report using a code-first workflow.

Here's how the course is structured and the design decisions behind it.

## Rationale

Our analysts produce good work in Excel, but the workflow has problems that compound over time. Charts are manually pasted into PowerPoint. Updates mean rebuilding everything from scratch. There's no version history, no reproducibility, and no easy way to review each other's work.

R and Quarto solve these problems directly. Write the analysis once, render it to HTML or PDF, track changes in Git, and review through pull requests. The challenge is getting people from zero to productive without losing them along the way.

I've instigated various other learning opportunities at WECA including "Code Club", presentations on code - first data analysis at our data group meeting and lunch and learn sessions on generative AI. Feedback from analysts has been that they want to "learn with intent". This project, which has a clear business need and benefit aims to deliver the learning opportunity and the business outcome.

## Course Structure

I settled on five 4-hour sessions, each building on the last:

1. **The Whole Game** — Run the entire workflow end-to-end before learning any syntax
2. **R Fundamentals** — Tidyverse pipes, dplyr verbs, ggplot2 basics
3. **Data Wrangling** — Import, clean, reshape, join real datasets
4. **Quarto Mastery** — Markdown, code chunks, tables, cross-references
5. **Git Collaboration** — Branching, pull requests, code review, merge conflicts

The total is 20 hours of contact time, split roughly 30% demonstration and 70% hands-on practise.

## Start With the Whole Game

The decision I'm most confident about is Session 1. Instead of starting with "what is a variable?", analysts run a complete workflow on day one: open the project, execute a code chunk that reads data, produces a chart, and renders a full indicator chapter to HTML.

They won't understand most of what they're doing. That's the point. Seeing the finished product first — and knowing they just made it happen — should give them a reason to care about the details that follow.

This approach comes from learning science research on "whole game" teaching. You don't teach football by spending three weeks on passing drills before anyone sees a match. You play a simplified version of the full game first, then build skills in context.

## Tidyverse First, Base R Later

I made a deliberate choice to teach tidyverse exclusively and skip base R. Some R instructors would disagree. My reasoning:

```r
# This is what analysts see on day 2
transport_data |>
  filter(local_authority == "Bristol") |>
  group_by(year) |>
  summarise(total_trips = sum(trips, na.rm = TRUE))
```

Compare that to the base R equivalent and the readability gap is obvious. For analysts coming from Excel, the tidyverse pipe syntax maps more naturally to how they already think about data operations: take this, then filter it, then group it, then summarise it.

I want people productive on real work, not fluent in computer science. Tidyverse-first serves that goal.

## Helper Functions to Hide Complexity

I wrote custom WECA helper functions that wrap ggplot2 theming:

```r
ggplot(data, aes(x = year, y = value)) +
  geom_line(colour = get_weca_color("purple")) +
  theme_weca()
```

`theme_weca()` handles all the brand styling — fonts, colours, grid lines, spacing. `get_weca_color()` returns hex codes from the WECA palette by name. Analysts get branded charts from session one without needing to understand the dozen-odd ggplot2 theme parameters underneath.

This is a calculated trade-off. It hides complexity, which makes the learning curve manageable, but it also means analysts won't immediately learn how ggplot2 theming works. They can peel back the abstraction later when they're ready.

## The Freeze Workflow

The trickiest part of using Quarto collaboratively is rendering. Our indicators project has multiple chapters written by different analysts, each using different datasets. If one analyst tries to render the whole book, it fails because they don't have everyone else's data.

Quarto's freeze feature solves this. When an analyst renders their chapter, the code execution results get cached in a `_freeze/` directory. That cache gets committed to Git. When someone else pulls the repository, they get the cached results without needing to re-execute the code.

```yaml
# _quarto.yml
execute:
  freeze: auto
```

I expect this will be conceptually the hardest thing to teach. "Your code ran, but the results are saved so others don't need to run it" requires understanding both Quarto's rendering pipeline and Git's role in sharing state. I've allocated extra time in Session 4 for it.

## Progressive Data Complexity

I've staged the difficulty of the practise datasets carefully:

- **Sessions 1-2**: Clean CSVs with consistent column names, no missing values, correct types. The focus is on learning R syntax, not fighting data quality.
- **Session 3**: Intentionally messy data — mixed case column names, wide format that needs pivoting, duplicates, missing values. This is where they learn `janitor::clean_names()`, `pivot_longer()`, and `drop_na()`.
- **Sessions 4-5**: Real indicator data with full real-world complexity.

Getting this gradient right matters. If session 1 data is messy, people get stuck on data problems before they've learned the tools to fix them. If session 3 data is too clean, they're unprepared for their actual work.

## Git: The Necessary Pain

Session 5 on Git is the one I'm most worried about. Version control is genuinely hard to learn, and the mental model (staging area, commits, branches, remotes) has no equivalent in most analysts' experience.

The approach is to keep the workflow simple and consistent. Every analyst follows the same pattern:

```bash
git checkout -b firstname/chapter-name
# do work
git add specific-files.qmd
git commit -m "Add transport indicator draft"
git push -u origin firstname/chapter-name
# create pull request on GitHub
```

There's a branch naming convention (`firstname/feature`) that's easy to remember and makes it obvious who's working on what. The session practises the full cycle — branch, commit, push, PR, review, merge — multiple times.

Merge conflicts get a dedicated exercise. I've set up a scenario where two people edit the same file, and we walk through resolving it together. Better to encounter that in a safe environment than for the first time on a deadline.

## Open questions

A few things I'm still weighing up.

Four hours on Git might not be enough. The concepts need repetition before they stick, and I may end up extending Session 5 or splitting Git across two sessions.

I've included pair programming as an option in later sessions, but I'm tempted to start it from Session 2. Working in pairs builds confidence and catches errors that would otherwise fester. Whether the analysts will go for it is another matter.

I'm also considering producing a cheat sheet — a single-page reference card with the most common dplyr verbs, ggplot2 geom types, and Git commands. Something to pin next to the monitor.

## Post-course support

The course itself is only the start. I'm planning weekly office hours for four weeks afterwards, a dedicated chat channel for questions, and I'll review each analyst's first five pull requests personally. The gap between "I completed the training" and "I can do this independently" is real, and filling it requires ongoing support.

## What success looks like

The measure is straightforward: can each analyst independently produce their indicator chapter using the R/Quarto/Git workflow? If they can do that — even with questions, even with the occasional `git push` to the wrong branch — the course has done its job.

The course materials are themselves a [Quarto book project](https://quarto.org/docs/books/), rendered to a browsable HTML site the analysts can refer back to after the sessions are done.

## Principles behind the design

Show people the destination before teaching them to drive. Use real data, not toy examples — toy examples don't transfer. Wrap complexity in helper functions early on; people can unwrap them later. And budget serious time for post-course support, because the training is where learning starts, not where it ends.

I'll write a follow-up once the course has run.

[GitHub Repo](https://github.com/stevecrawshaw/r-quarto-learning)
