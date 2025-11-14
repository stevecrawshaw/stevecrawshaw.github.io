---
layout: post
title: Vibe coding - further learning
date: 2025-11-14
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

Following my previous post on vibe coding a streamlit app using Claude Code, I wanted to document some further learning and experiments I have done with this approach.

## 1. MCPs

I found an interesting [blog post](https://robertmarshall.dev/blog/turning-claude-code-into-a-development-powerhouse/) setting out an approach to using Model Context Protocol for saving tokens and increasing accuracy. In summary, `context7` to access docs for the libraries you use, `serena` - a "noetic" coding agent, and an AI Architect called "sequential thinking". These three mcps are installed, and called through a custom slash command called `//go` which invokes this prompt:

```text
Always use:
- serena for semantic code retrieval and editing tools
- context7 for up to date documentation on third party code
- sequential thinking for any decision making
Read the claude.md root file before you do anything.
#$ARGUMENTS
```

Serena mcp starts a web server and indicates the "memories" it is making. This is important to track where you are in the project between sessions. I found that this seemed to produce better results than not using it but haven't evaluated fully myself.

My mcp server setup is below - these live in `~/.claude.json`:

```json

"mcpServers": {
        "context7": {
          "type": "http",
          "url": "https://mcp.context7.com/mcp"
        },
        "duckdb": {
          "type": "stdio",
          "command": "uvx",
          "args": [
            "mcp-server-duckdb",
            "--db-path",
            "/home/steve/projects/lnrs-db-app/data/lnrs_3nf_o1.duckdb",
            "--keep-connection"
          ],
          "env": {
            "HOME": "/home/steve"
          }
        },
        "sequential-thinking": {
          "type": "stdio",
          "command": "npx",
          "args": [
            "-y",
            "@modelcontextprotocol/server-sequential-thinking"
          ],
          "env": {}
        },
        "serena": {
          "type": "stdio",
          "command": "uvx",
          "args": [
            "--from",
            "git+https://github.com/oraios/serena",
            "serena",
            "start-mcp-server",
            "--context",
            "ide-assistant"
          ],
          "env": {}
        },
        "github": {
          "type": "stdio",
          "command": "npx",
          "args": [
            "-y",
            "@modelcontextprotocol/server-github"
          ],
          "env": {
            "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_my_access_token"
          }
        }
     }
```

## 2. Mix Claude app with Claude Code

I went down a rabbit hole trying to get Claude code to integrate an ERD (Entity Relationship Diagram) onto the app so that users can get a sense of how the tables relate. Despite giving what I thought was a clear prompt, the output wasn't great. CC suggested a mermaid chart, but it rendered in a postbox view and wasn't at all aesthetic. I ended up asking Claude chatbot to make me an ERD from the xml schema file and it did it nicely in REACT and created an artifact. I then embedded the artifact as an iframe in the app. It looks much better than the first attempt.

## 3. Hooks for formatting

I like the code to look nice, and the [Ruff](https://docs.astral.sh/ruff/) tool is a nice fast linter to help keep the code compliant with various conventions. You can set up a "hook" which asks ruff to check and fix linting issues after every edit to a python file. Add this to your `.claude/settings.json` file. [Note that the syntax changed between versions and may well do again].

```json

  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_FILE_PATHS\" =~ \\.py$ ]]; then ruff check --fix \"$CLAUDE_FILE_PATHS\" && ruff format \"$CLAUDE_FILE_PATHS\"; fi"
          }
        ]
      }
    ]
  },
  "alwaysThinkingEnabled": true

```

And here's my `ruff.toml` file:

```toml

exclude = [
    ".bzr",
    ".direnv",
    ".eggs",
    ".git",
    ".git-rewrite",
    ".hg",
    ".ipynb_checkpoints",
    ".mypy_cache",
    ".nox",
    ".pants.d",
    ".pyenv",
    ".pytest_cache",
    ".pytype",
    ".ruff_cache",
    ".svn",
    ".tox",
    ".venv",
    ".vscode",
    "__pypackages__",
    "_build",
    "buck-out",
    "build",
    "dist",
    "node_modules",
    "site-packages",
    "venv",
]
line-length = 88
indent-width = 4
target-version = "py313"

[lint]
select = [
    #pycodestyle
    "E",
    "W",
    # pyflakes
    "F",
    # pyupgrade
    "UP",
    # flake8-bandit
    "S",
    # flake8-bugbear
    "B",
    # flake8-simplify
    "SIM",
    # isort
    "I"
]
ignore = []
fixable = ["ALL"]
unfixable = []
dummy-variable-rgx = "^(_+|(_+[a-zA-Z0-9_]*[a-zA-Z0-9]+?))$"

[format]
quote-style = "double"
indent-style = "space"
skip-magic-trailing-comma = false
line-ending = "auto"
docstring-code-format = true
docstring-code-line-length = "dynamic"

```


## 4. Dependencies

Pretty obvious one really but I got caught out with a new version of duckDB not being compatible with MotherDuck (1.4.1 vs 1.4.2). While motherduck will undoubtedly upgrade soon, I needed to align versions for development so I have stuck at 1.4.1. Updating duckDB to a specific version:

```bash
# Download the specific version
wget https://github.com/duckdb/duckdb/releases/download/v1.4.1/duckdb_cli-linux-amd64.zip

# Unzip the file
unzip duckdb_cli-linux-amd64.zip

# Make it executable
chmod +x duckdb

# Move it to a location in your PATH (optional but recommended)
sudo mv duckdb /usr/local/bin/

# Verify the installation
duckdb --version

```

You can just specify the dependency version in the `pyproject.toml` file:

```toml
[project]
name = "lnrs"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "duckdb==1.4.1",
    "ipykernel>=7.1.0",
    "polars>=1.34.0",
    "pyarrow>=21.0.0",
    "python-dotenv>=1.2.1",
    "sqlfluff>=3.5.0",
    "streamlit>=1.50.0",
]

[dependency-groups]
dev = [
    "ruff>=0.14.3",
]

```
