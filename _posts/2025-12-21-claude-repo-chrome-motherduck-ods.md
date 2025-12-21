---
layout: post
title: Claude setup repo, Motherduck MCP, Claude Code in Chrome
date: 2025-12-21
categories: [AI, Agents, Skills, MCP, HuWise, open data]
tags:
- Claude Code
- Motherduck
- Agentic coding
- Chrome
- HuWise
- OpenDataSoft
---

# Claude setup repo

I regularly peruse the [r/ClaudeAI](http://reddit.com/r/claudeai) and have amassed a set of "useful" links and resources. To make them available to me across machines and settings they are now in a [GitHub repo](https://github.com/stevecrawshaw/claude-setup).

# Claude in Chrome extension

I had a frustrating couple of sessions with Claude last week trying to get playwright MCP working on chromium in Ubuntu. I never got it working, and Claude just seemed to get stuck in a loop trying to diagnose the problem.

Anyway this week Anthropic released [Claude in Chrome](https://support.claude.com/en/articles/12012173-getting-started-with-claude-in-chrome) which is an extension, and MCP tool that seemingly does something similar to playwright and enables interaction with the browser. It didn't work with chromium - and I suspect because of chromium's installation with snap. So I installed Chrome proper. With a bit of config and transferring settings, bookmarks, extensions etc I was up and running with it.

I created two skills using claude code in the first day or two of operation, both using claude in chrome (CIC) to send search terms and specifications to web - based chatbots (Perplexity and Gemini). I have pro accounts with both of these, but no API service. My thinking is that this approach should be context efficient, and save an API billing. But there will be tradeoffs, such as complexity and delay.

## Perplexity search

I asked claude to plan a way of implementing this skill by experimenting with CIC to find out where the buttons are, assess the functionality etc. It made a plan and wrote a [pretty good skill](https://github.com/stevecrawshaw/claude-setup/blob/main/skills/perplexity-search/SKILL.md). It relies on me being logged in to my pro account.

## Gemini search

Similarly [Gemini search](https://github.com/stevecrawshaw/claude-setup/blob/main/skills/gemini-search/SKILL.md) \ AI can be invoked in the same way. I have a couple of goes at this, discovering which text format Claude likes as a "structured" output. So the skill takes a search term and prompts the user for the type of model (Fast, Thinking, Pro) and whether output is to be structured or not - my thinking is that "structured" is better for Claude to parse, though the structure is minimal and limited to denoting the search \ model and response. As I have two google accounts and only one of them is tied to the Gemini Pro, I passed the auth parameter for the pro account and email address in the skill.

## Motherduck MCP

Motherduck continues to impress. I love this fast cloud OLAP database and have built a few things using it. They've just written an [MCP Server](https://motherduck.com/docs/sql-reference/mcp/) for it which enables natural language introspection and queries of the database hosted on it. It was trivial to install, and works perfectly out of the box (minimal testing!) and with a simple prompt I was able to generate a comprehensive [json schema](https://github.com/stevecrawshaw/lnrs-db-app/blob/main/lnrs_weca_schema.json) for the LNRS database that I host on Motherduck. Very impressed!

## Opendatasoft (HuWise) MCP Tools

Someone has developed a [tool](https://github.com/pawneetdev/rest-to-mcp-adapter) to take [openapi or swagger API](https://opendata.westofengland-ca.gov.uk/api/explore/v2.1/swagger.json) specifications and turn it into MCP tools. What this means is that you can ask Claude Code, in natural language, questions about your data, and it will translate this, quite well into code to send a request to the API and return data. HuWise have done something similar on the portal in their “Explore data with AI” feature, and offer it in their [Data Marketplace](https://www.huwise.com/en/mcp-ai-agents/) product but this is a bit more powerful for deeper analysis - but requires some setup and a Claude account and Claude Code.

I simply cloned the repo and asked Claude to create MCP tools from the swagger json for my portal, and it did the rest.

Here is my cloned [repo](https://github.com/stevecrawshaw/rest-to-mcp-adapter), with the ods_server.py script providing the mcp tools for my portal. Full description of how to use in the [README file](https://github.com/stevecrawshaw/rest-to-mcp-adapter/blob/master/README.md). There's a discussion in the [Community](https://community.huwise.com/the-world-of-data-74/mcp-tools-to-access-portal-datasets-and-catalog-691) section of HuWise help hub.