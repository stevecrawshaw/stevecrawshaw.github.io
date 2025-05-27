---
layout: post
title: Corey Schafer's UV Video
date: 2025-05-27
categories: [Python, Data Science, UV]
tags:
- uv
- .venv
- dependencies
- install
- python
- packages
---

# Corey Schafer's UV Video

[YouTube Video](https://www.youtube.com/watch?v=AMdG7IjgSPM&t=2s)

## Some tricks I picked up from the video

1. ```uv init new_directory``` [automatically creates a git repo, with a python .gitignore]
    - can then create github repo with ```gh auth login```; ```gh repo create <repository-name> --public --source=. --push``` 
2. Even if the .venv isn't there, uv run myscript.py will create the .venv from the pyproject.toml file and install the dependencies.
3. ```uv sync``` [updates the .venv with the latest dependencies from pyproject.toml]
4. ```uv add -r requirements.txt``` [adds the requirements from a requirements.txt to the pyproject.toml file - can then delete requirements.txt]
5. ```uv tool install ruff``` [installs ruff in isolated env but makes available on the PATH]
6. ```uv tool run ruff check .``` [installs and runs ruff in a temporary isolated environment]
7. ```uvx ruff check . ``` [shortcut for the above command]
