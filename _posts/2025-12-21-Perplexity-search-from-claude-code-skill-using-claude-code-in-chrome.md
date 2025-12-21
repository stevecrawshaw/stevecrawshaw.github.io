---
name: perplexity-search
description: Search Perplexity.ai and retrieve comprehensive AI-powered search results
version: 1.0.0
author: Steve Crawshaw
tags: [search, perplexity, browser, research]
---

# Perplexity Search Skill

This skill enables automated searching on Perplexity.ai using Claude in Chrome browser automation. It handles the complete workflow from browser initialization to result retrieval.

## When to Use This Skill

Use this skill when you need to:
- Search for information using Perplexity.ai's AI-powered search
- Get comprehensive answers with multiple sources
- Research topics with structured, cited results
- Leverage Perplexity's synthesis capabilities for complex queries

## Arguments

The skill accepts a search query as an argument:
- **query** (required): The search query to execute on Perplexity.ai

## Usage Examples

/perplexity-search "Tesla's tower at Wardenclyffe"
/perplexity-search "latest developments in quantum computing"
/perplexity-search "how does CRISPR gene editing work"

## What This Skill Does

<skill_implementation>

You are executing the Perplexity Search skill to search Perplexity.ai and retrieve results.

<workflow>
Follow these steps in order to complete the search:

<step id="1">
<name>Initialize Browser Context</name>
<instruction>
Call mcp__claude-in-chrome__tabs_context_mcp with createIfEmpty set to true.
Extract the tabId from the available tabs in the result.
Store this tabId for use in all subsequent steps.
</instruction>
</step>

<step id="2">
<name>Navigate to Perplexity</name>
<instruction>
Call mcp__claude-in-chrome__navigate with:
- tabId: the captured tabId from step 1
- url: https://perplexity.ai

Wait for navigation to complete.
</instruction>
</step>

<step id="3">
<name>Locate Search Input</name>
<instruction>
Call mcp__claude-in-chrome__find with:
- tabId: the captured tabId
- query: "search input box"

The search input will have helper text "Ask anything. Type @ for mentions and / for shortcuts."
Extract the ref ID from the matching element (this will be your search_input_ref).
</instruction>
</step>

<step id="4">
<name>Focus Search Input</name>
<instruction>
Call mcp__claude-in-chrome__computer with:
- action: left_click
- ref: the search_input_ref from step 3
- tabId: the captured tabId

This focuses the search input field.
</instruction>
</step>

<step id="5">
<name>Enter Search Query</name>
<instruction>
Call mcp__claude-in-chrome__computer with:
- action: type
- text: the search query provided by the user
- tabId: the captured tabId

This types the query into the focused search field.
</instruction>
</step>

<step id="6">
<name>Submit Search</name>
<instruction>
Call mcp__claude-in-chrome__computer with:
- action: key
- text: Return
- tabId: the captured tabId

This submits the search by pressing Enter.
The URL will change to a pattern like https://www.perplexity.ai/search/*
</instruction>
</step>

<step id="7">
<name>Wait for Results</name>
<instruction>
Call mcp__claude-in-chrome__computer with:
- action: wait
- duration: 3
- tabId: the captured tabId

This waits 3 seconds for the search results to fully load.
For complex queries, you may need to wait longer (up to 5 seconds).
</instruction>
</step>

<step id="8">
<name>Handle Cookie Dialog (If Present)</name>
<instruction>
Call mcp__claude-in-chrome__find with:
- tabId: the captured tabId
- query: "Necessary Cookies button"

If a cookie button is found:
- Extract its ref ID
- Call mcp__claude-in-chrome__computer with action: left_click and the cookie button ref
- This dismisses the cookie consent dialog

If no cookie button is found, continue to the next step.
</instruction>
</step>

<step id="9">
<name>Retrieve Search Results</name>
<instruction>
Call mcp__claude-in-chrome__get_page_text with:
- tabId: the captured tabId

This extracts the full text content of the search results page.
The results will include:
- Main answer summary synthesized from sources
- Basic facts (bullet points)
- Detailed sections with headings
- Source count (e.g., "Reviewed 10 sources")
- Related follow-up questions
</instruction>
</step>

<step id="10">
<name>Capture Screenshot (Optional)</name>
<instruction>
Optionally call mcp__claude-in-chrome__computer with:
- action: screenshot
- tabId: the captured tabId

This captures a visual representation of the results page.
</instruction>
</step>

</workflow>

<result_formatting>
After retrieving the results, present them to the user in a clear, organized format:

1. Start with a brief summary of what was found
2. Include the main answer sections with proper headings
3. Highlight key facts using bullet points
4. Note the number of sources reviewed
5. Optionally include related questions at the end
6. Include a screenshot if captured

Do not reproduce large blocks of copyrighted text verbatim.
Instead, summarize key points and provide context.
</result_formatting>

<error_handling>
If any step fails:

<element_not_found>
If the search input element is not found, the page structure may have changed.
Try using mcp__claude-in-chrome__read_page to inspect the current page structure.
</element_not_found>

<browser_disconnected>
If the browser extension loses connection, inform the user they need to:
- Restart Chrome
- Reconnect the Claude browser extension
Then retry the search.
</browser_disconnected>

<timeout>
If results take too long to load:
- Increase the wait duration in step 7
- Check the page with a screenshot to see if results have loaded
- Some complex queries may require up to 5-10 seconds
</timeout>

</error_handling>

<notes>
- Element references (ref_*) are dynamic and captured at runtime
- Cookie dialogs only appear on first visit or after clearing cookies
- Wait times may need adjustment based on query complexity
- Perplexity Pro features may affect available UI elements
</notes>

</skill_implementation>

## Expected Results

The skill returns:
- Comprehensive answer synthesized from multiple sources
- Organized sections with headings (e.g., Basic Facts, Purpose, Current Status)
- Citation information showing number of sources reviewed
- Related follow-up questions for deeper research
- Optional screenshot of the results page

## Limitations

- Requires Claude in Chrome browser extension to be installed and connected
- First-time use may encounter cookie consent dialogs
- Complex queries may require longer wait times
- Results depend on Perplexity.ai's availability and API

## Technical Details

This skill uses the Claude in Chrome MCP server tools:
- `tabs_context_mcp`: Browser tab management
- `navigate`: URL navigation
- `find`: Element location
- `computer`: Mouse/keyboard actions
- `get_page_text`: Content extraction

All browser interactions are automated following the workflow pattern documented in `perplexity-search-workflow.xml`.
