---
layout: post
title: Vibe coding a CRUD application with Claude Code
date: 2025-11-03
categories: [AI, Vibe coding, Python, Database]
tags:
- polars
- CRUD
- LNRS
- Claude Code
- DuckDB
- Agentic coding
- Streamlit
---

# Vibe coding a CRUD application with Claude Code

In this post, I'll explain how I used Claude Code to build a simple CRUD (Create, Read, Update, Delete) [application](https://github.com/stevecrawshaw/lnrs-db-app) using Vibe (AI supported) coding. The application uses Polars for data manipulation and DuckDB as the database backend. DuckDB's relational python API was also used for database interactions. Finally, I'll demonstrate how to deploy the application using Streamlit.

**CAVEAT I'm not an experienced dev - just learning as I go..**

## Application Overview

The Local Nature Recovery Strategy is a new duty and framework for local authorities in England to plan and deliver nature recovery. The West of England Combined Authority was the first local authority in England to publish its LNRS in November 2024. A key component of the LNRS was the development of an "[LNRS toolkit](https://opendata.westofengland-ca.gov.uk/pages/lnrs-application/?headless=true)" - a web application to help farmers and landowners identify opportunities for nature recovery on their land. The toolkit is underpinned by a wide variety of [open datasets](https://opendata.westofengland-ca.gov.uk/explore/?sort=modified&q=LNRS) published on the West of England's open data portal. The open data approach is critical for transparency and trust, allowing users to understand how the toolkit works and access the data it uses.

The datasets used in the LNRS toolkit include:

- Areas (polygons) of different habitat and landscape types
- Biodiversity priorities grouped by theme
- Measures (actions) to deliver the biodiversity priorities
- Grants available to fund the measures
- Priority species - which could benefit from measures taken

These datasets are all interrelated. For example, certain measures may be more appropriate for specific habitat types or biodiversity priorities. The application allows users to explore these relationships and identify the most suitable actions for their land using a simple map-driven interface.

The source data originate in a large and complex spreadsheet manually created by the lead officer for LNRS. The original ingestion process involved significant data cleaning and transformation using a monolithic R script to get the data into a usable format. **This is where the CRUD application comes in - it allows for easy management and updating of the underlying datasets as new information becomes available or corrections are needed, thereby dispensing with the need for complex and fragile scripts.**

## Data Model

The first step in building the CRUD application was to define the data model. The LNRS toolkit is driven by a single "one big table" approach, where all the relevant information is combined into a single table for ease of querying and analysis. However, for the CRUD application, I decided to break this down into several related tables to better represent the relationships between the different entities. This was quite challenging to do manually, so I used Perplexity to help generate the initial schema based on the original spreadsheet structure.

I implemented the data model in DuckDB, developing the schema iteratively as I refined my understanding of the relationships between the entities. The schema includes bridge tables to capture the many-to-many relationships between measures, habitats, biodiversity priorities, and grants. All the relevant data is included in a single database file with a size of ~21MB. **This relational structure supports more efficient querying and data manipulation and is critical to help Claude understand the relationships between tables.**

## Design Decisions

### Front - end Framework

I did a fair bit of research into possible CRUD application frameworks, using coding approaches and low - code \ no - code applications. The low - code solutions looked OK, but I was concerned about the lack of flexibility and control over the code, as well as vendor lock - in.

I had played with Streamlit before, and it seemed like a good fit for this application. It has a simple and intuitive API for building web applications, and it integrates well with Python libraries like Polars and DuckDB. I also liked the fact that Streamlit applications can be easily deployed to the web using services like Streamlit Cloud.

### Back - end Framework

For the back - end, I decided to use DuckDB as the database backend. I have been using it reasonably intensively for the last year or so for data analysis and ETL tasks, and I like its simplicity and performance. DuckDB's relational python API makes it easy to interact with the database from Python code, and it supports a wide range of SQL features.

I initially attempted to build the CRUD application with Gemini CLI, and using a cloud instance of DuckDB [Motherduck](https://Motherduck.com/). This proved to be slow going and I realised that the communication with the cloud instance and authentication etc would prove to be a bottleneck and complication during development. So I switched to using a local DuckDB instance, which made development much faster and smoother [but see note later about write access](#improvements-during-development).

### AI Coding Model

I've been exploring various AI coding models over the last year or so. GitHub copilot's integration with VS Code is excellent, and I use it all the time when writing analysis code. I have a Gemini Pro subscription and I like the deep research capabilities and its all round capability, but have found performance with the Gemini CLI sub optimal. A short window of free access to Claude Pro encouraged me to try out Claude Code for this project.

Work laptops are quite limited for installing and running non - standard software, so I used my personal Linux workstation for this project. I installed [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) and used it from within the VS Code IDE. None of the data are personal or sensitive.

## Environment Configuration

1. Clone the existing messy LNRS repo and delete irrelevant files
2. Create /data folder and move in relevant CSV files and the lnrs_3nf_o1.duckdb database file
3. rm /.git -rf and git init
4. uv init and uv add streamlit polars duckdb ipykernel
5. gh repo create lnrs-db-app
6. git commit and push

## Priming the AI Coding Model with Prompts and Planning Documents

This is a critical step. I researched several web resources and implemented practice from them. Here are some useful ones.

- [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Getting started with Claude Code](https://vincent.codes.finance/posts/claude-code-data-analysis/)
- [Claude Code optimisation](https://github.com/JoeInnsp23/claude-code-optimisation?tab=readme-ov-file)
- [Claude code everything you need to know](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know)
- [How I use Claude Code](https://bagerbach.com/blog/how-i-use-claude-code)

My tips and learning from these:

- Write a comprehensive README.md file in the root of your repo describing the project. This should capture the key functionality of the project.
- Prime Claude with your database structure by giving it an [XML schema](https://Motherduck.com/blog/vibe-coding-sql-cursor/). [Python script](https://github.com/stevecrawshaw/lnrs-db-app/blob/main/get_schema.py) for duckdb.
- Similarly any other files that might be useful for it to understand project context - but be concise and don't overload it
- Set up mcp servers for [context7](https://github.com/upstash/context7) (access docs) and in the case of this project [DuckDB](https://github.com/Motherduckdb/mcp-server-Motherduck) so that Claude can access the database
- In Claude Code use `/init` to create the CLAUDE.md file that will drive the model's work.
- **Review and improve the CLAUDE.md to ensure it accurately reflects the work you want it to do**
- be explicit about environment management - e.g. "always use uv add or uv sync rather than venv or editing the pyproject.toml file directly".
- Make sure tasks are broken down into small sub tasks
- Ask claude in the CLAUDE.md to use the mcp servers for specific tasks - be explicit.
- Ask Claude to create an implementation plan and document its actions in markdown files.
- Once you've got a good plan, with clear phases, ask Claude to start development - but ask it to test at key phases
- Git commit regularly and at key points
- when it has completed tasks or phases, ask it to update e.g. PHASE_2_IMPLEMENTATION.md
- If you need to break and resume, remind it where you were by adding the context of the most recent doc e.g. @PHASE_2_IMPLEMENTATION.md
- You can tell it to Think hard, Think harder, Ultrathink for tasks that you anticipate are challenging
- Use `/compact` at the completion of key tasks to avoid unpredictable behaviour.

## Progress and Experience of Vibe Coding

I wasn't confident to run this process in "YOLO" (You Only Live Once) mode, where you leave the model to automatically proceed at each step. So I closely monitored progress and tried to verify the steps it was taking. What helped was regular testing of the Streamlit app from a separate bash terminal `uv run streamlit run app.py`. This highlighted a recurring problem linked to the design of the database, which was the decision to use the grant identifier like "CAB12" as the unique ID  - primary key of the record in the grants table. All the other tables used incrementing integers as the primary key. This threw the model off and needed to be fixed. Easily done by pasting the error message from streamlit into the prompt. Learning point - keep the design of the database tables consistent!

### Improvements During Development

When most of the implementation was complete, I started looking into deployment. I researched this with Google Gemini. A potential stumbling block emerged, in that the popular (free) deployment options do not allow write access to the database. This is critical as the Create, Update and Delete components of CRUD require write access. I had originally stated this project thinking that a Motherduck instance could be a viable data storage solution; now it was time to revisit that approach. When attempting to copy the local database to Motherduck with ```sql CREATE OR REPLACE DATABASE lnrs_weca FROM lnrs;``` I ran into a foreign key constraint error. On fully researching this error it transpired that there was no foreign key error in the database itself, it was an error related to the order in which duckDB processes constraints in the copying function. So the rather long - winded approach is to manually recreate the schema on Motherduck and copy the data from the local database to Motherduck manually. This did work, but the foreign key error seems like a bug.

Anyway, remote database implementation solved the ephemeral file system issue and Claude did well in implementing this solution, migrating Motherduck token from the .env file to streamlit secret.

On testing the Motherduck connection option, while this did work, the performance seemed laggy. I asked Claude to resolve, perhaps with a caching strategy. Implementing caching in streamlit for the dashboard component seemed to improve performance adequately. It may be necessary to implement more components of the caching strategy when the app is deployed to the cloud.

## Results

I completed five hours of coding before the daily limit kicked in. Claude completed an impressive ~3000 lines of code in that time, and around 80% of the app. All CRUD operations were completed and I was able to test all operations successfully. The default UI it proposed was fine. This is not an application which will be exposed to outside \ public users, so the interface just needs to be functional. But it still looks nice. In any case I think streamlit would limit the tweaking that is possible for the UI.

The app itself is not public, as I don't want randoms changing our database. Here's a screenshot.

<img src="https://github.com/stevecrawshaw/stevecrawshaw.github.io/blob/5510e1986a3fd9fdd11df03090f4c6d2d6327fd5/images/lnrs-db-app-crud.streamlit.app.jpeg">