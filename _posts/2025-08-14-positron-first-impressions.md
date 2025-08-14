---
layout: post
title: Positron IDE - first impressions
date: 2025-08-14
categories: IDE, Positron, Posit, R, Python]
tags:
- IDE
- R
- Agentic AI
---

"[Positron IDE](https://positron.posit.co/) is a new integrated development environment (IDE) designed for data scientists and developers working with R and Python. It aims to provide a seamless experience for coding, debugging, and deploying applications in these languages."

I am a long - time Rstudio user and a more recent VS code user (for python and SQL). I was intrigued to see the development of the Positron IDE, but waited until the product emerged from public Beta before testing it. That time has come, and I have now spent a few hours with the IDE. Here are my first impressions.

### Installation
I went for the system level install, which needed admin rights on my work PC. On reflection, it would probably have been better to go for user level install, as I have already found annoying restrictions like requiring admin rights to change keybindings to send lines of SQL to the terminal. However the install went OK and I was able to configure the IDE to bring in my settings from Rstudio and VS code.

### Positron Assistant
GitHub copilot has become indispendable for me in data analysis. This is implemented in Positron as the Positron Assistant. It uses a combination of Github copilot for code completion and Anthropic LLM API Key for chat. I have copilot paid plan through work, but my anthropic key is paid for personally. I haven't really tested the chat interface properly but it all looks like its working so that is reassuring. I need to see if Google Gemini Code Assistant can be integrated as well, as I have a paid key for that too.

### Interface
I think the interface is a good combination of Rstudio and VS code. I really missed the plot viewer in VS code so it's great to see this implemented in the way I expect. The new Air terminal for R seems fine and has a more modern feel than the plain R console in Rstudio. It includes an autocomplete menu feature which I think will be very handy.

### Python Environment and Templates
It looks like Posit has integrated python environments quite well. A nice feature is the ability to create a folder from a template, which includes the Python version if you select a Python template. Packages can be added to the automatically provided pyproject.toml file and .venv using e.g. `uv add polars` in the normal way. This takes some of the pain away from managing Python environments, and also uses the great `uv` tool for package management.

### Summary
Overall, my first impressions of Positron IDE are positive. It feels like a good blend of Rstudio and VS code, with some nice features for Python development. The integration of GitHub copilot and Anthropic LLMs is promising, and the interface is intuitive for someone familiar with both Rstudio and VS code. I'm confident to switch to Positron for my data analysis work, and I'm looking forward to exploring its features further. I just need to figure out how to use the DuckDB cli.