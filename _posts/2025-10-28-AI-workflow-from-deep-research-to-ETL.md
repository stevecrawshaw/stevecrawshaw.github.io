---
layout: post
title: AI Workflow - from deep research to ETL automation
date: 2025-10-28
categories: [AI, ETL, Python, Automation]
tags:
- polars
- ETL
- energy
- API
- httpx
- Agentic coding
- Google Gemini
- Copilot
---

# AI Workflow - from deep research to ETL automation

Although I have been using AI and agentic coding a little for the last year or so, I have not conducted a full process from deep research to ETL automation until recently. In this post, I will share my experience of using AI to streamline the workflow of researching the issue and then extracting, transforming, and loading (ETL) energy data from an API into a Polars DataFrame.

# Step 1: Deep Research with AI

I have been asked to prepare baseline data and research for the environment plan. Most of the data for this work is publicly accessible open data from government APIs. However, one part of the request was not so straightforward - local generation from fossil fuel sources. This data is not directly available in a granular and recent format.

To address this I used Google Gemini's deep research capabilities to gather [relevant information and insights](https://docs.google.com/document/d/1i10Uhdt-7VwZqHamycNXagK0OZ7wpPZPSpJYha6xCrE/edit?usp=sharing). This actually uncovered the key intelligence I couldn't find previously, which was a free access [API endpoint from Elexon](https://bmrs.elexon.co.uk/api-documentation/endpoint/datasets/B1610). Gemini's deep research mode is really excellent for this kind of task, as it can synthesize information from multiple sources and provide a coherent summary.

I needed to ask a couple of follow up questions about the API about settlement periods and data granularity, but overall this part of the process was very smooth and efficient.

# Step 2: ETL Automation with AI

The next step was to automate the data extraction and transformation process using Python. I have done this for other datasets before, but this time I wanted to leverage AI to help with the coding. I also wanted to use specific libraries like `Polars` for data manipulation and `httpx` for making API requests.

To start, I used the `Build with Agent mode` option in GitHub copilot using Claude Sonnet. In order to give the agent enough context I wrote a [markdown file](https://github.com/stevecrawshaw/environment-plan-evidence/blob/main/seabank-generation.md) which used some of the examples from the Elexon API documentation and explained what I wanted to achieve. I was specific about the libaries I wanted to use and the approach for handling the data - logging, checkpoints etc. Because the httpx library is not as well known as the requests library, I created a httpx-docs.md file from [httpx docs](https://www.python-httpx.org/quickstart/) using Gemini to provide the syntax for this library locally.

I also asked copilot to initially plan, and then ask any questions which weren't clear in my prompt. This was really useful as it helped clarify some of the details about the API parameters and the data structure.

Once approved, the agent started generating code in a single python file. The code looked good, but there were a few things I didn't understand. So again, I asked Gemini to review the code. I copied the code into a new Gemini chat and asked it to identify problems and any potential improvements. Gemini provided a list of errors and potential optimisations. I saved this code review as a [markdown file](https://raw.githubusercontent.com/stevecrawshaw/environment-plan-evidence/refs/heads/main/code-review.md) for reference and asked copilot to respond to the code review, directing it to make specific changes. It made several changes and improvements based on the review.

1. Daylight saving time handling
2. Incorrect implementation of asyncio for httpx requests (this ended up saving 40 minutes of runtime!)
3. Errors in checkpoint implementation
4. Fragile date parsing logic

I think this was a key step and stopped me getting bogged down in debugging.

# Step 3: Running the ETL Script

I didn't want to run the full date range initially, so I modified the script to run for two days. The script ran successfully and produced a Polars DataFrame with the expected data, so I reverted back to the full year and ran the script. Here's the output.

2025-10-28 14:43:23,361 - INFO - Progress: 17568/17568 (100.0%)
2025-10-28 14:43:23,363 - INFO - Data retrieval complete. Retrieved 35136 records
2025-10-28 14:43:23,405 - INFO - Created DataFrame with 35136 rows and 8 columns
2025-10-28 14:43:23,422 - INFO - Data saved to seabank_generation_2024.csv
2025-10-28 14:43:23,422 - INFO - Checkpoint file removed after successful completion
2025-10-28 14:43:23,422 - INFO - Summary statistics:
2025-10-28 14:43:23,423 - INFO -   Total records: 35136
2025-10-28 14:43:23,423 - INFO -   Date range: 2024-01-01 to 2024-12-31  
2025-10-28 14:43:23,424 - INFO -   Units: T_SEAB-1
2025-10-28 14:43:23,424 - INFO -   Total generation (MWh): 3597182.79    
2025-10-28 14:43:23,424 - INFO - **Script completed in 266.71 seconds**

The initial assessment of how long this would take was 44 minutes, so the asyncio optimisation (and fixing its implementation using Gemini's code review) really paid off!

Next steps are to analyse the data. It's a really rich dataset with half hourly volume generation for both generating units at Seabank power station for the whole of 2024.

# Conclusion

In summary, this experience has shown me how AI can be effectively used to streamline both the research and coding aspects of a data engineering workflow. By leveraging Google Gemini for deep research and GitHub Copilot with Claude Sonnet for code generation and review, I was able to efficiently gather information and automate the ETL process using Python, `Polars`, and `httpx`.

# Workflow Summary

1. Research with Google Gemini Deep Research.
2. Write requirements and context in markdown.
3. Share docs as context, especially for lesser known libraries in markdown.
4. Ask the AI to plan and ask questions.
5. Generate code with GitHub Copilot Agent mode.
6. Review initial code with Google Gemini (or another coding assistant).
7. Test in a limited way to prove it works.
8. Run full script and validate output.
