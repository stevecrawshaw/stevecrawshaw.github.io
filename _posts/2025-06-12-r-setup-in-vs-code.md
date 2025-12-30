---
layout: post
title: Setup for R in VS Code
date: 2025-06-11
categories: [R, Code]
tags:
- VS Code
- R
- setup
---

I mainly use the venerable RStudio for my R development, but I also use Visual Studio Code (VS Code) for python and SQL. Once [positron](https://positron.posit.co/start.html) is out of public beta I will probably switch as that seems like a good overall IDE for R, Python, and SQL and I think includes a better graphics window option than that available in VS code.

There are plenty of guides on the internet for setting up R in VS Code and I won't reproduce their content, but I will highlight my own pain points and fixes for others and future me. I recently struggled when updgrading to the latest version of R when I went to use R in VS Code. This was partly my own fault as I had installed R 4.4.0 in `C:\Program Files\` which required admin permissions, and I installed the upgrade in `C:\Users\` which didn't.

Firstly following the upgrade I needed to move my pre - existing libraries to the new `\library` folder in R 4.5.0. This did the trick in Rstudio. But when I went to use R in VS Code I was only able to see the R 4.4.0 libraries and terminal.

So the fix is to firstly set up .Renviron file with the right `.libPaths()` in your home directory. This is a file that R reads on startup to set up the environment. You can create this file if it doesn't exist.

The contents to add to the `.Renviron` file are:

```r
.libPaths(new = "C:\\Users\\steve.crawshaw\\AppData\\Local\\Programs\\R\\R-4.5.1\\library")
```

I decided to add `radian` as my R terminal in VS Code, which is an alternative to the default R terminal and gives some nice features like autocomplete with tab. I installed it using `uv tool install radian` in the terminal. This should ensure it is added to the path and available everywhere, including VS Code.

Then we need to make some changes to the user settings.json file in VS Code. You can access this by pressing `Ctrl + Shift + P` and typing `Preferences: Open Settings (JSON)`.
Add the following lines to the `settings.json` file:

```python
{
    // R settings
    "r.plot.useHttpgd": true,
    "r.rterm.windows": "C:\\Users\\steve.crawshaw\\.local\\bin\\radian.exe",
    "r.rpath.windows": "C:\\Users\\steve.crawshaw\\AppData\\Local\\Programs\\R\\R-4.5.1\\bin\\R.exe",
    "r.lsp.debug": true,
    "r.lsp.diagnostics": true,
    "r.alwaysUseActiveTerminal": true,
    "r.rterm.option": [
        "--no-save",
        "--no-restore"
    ],
    "r.bracketedPaste": true,
    "r.sessionWatcher": true,
}
```

Note that you will also need to create a `~/radian_profile` file to help to send the complete code chunk to the radian terminal without being broken up into separate lines. Here's how to do it.

You must disable Radian's internal auto-indentation. To do this:

```bash
code ~/.radian_profile
```

Paste these lines.

```R
options(radian.auto_match = FALSE)
options(radian.auto_indentation = FALSE)
```

Save and Close. Restart the Radian terminal.

Now all works as expected. I can use R in VS Code with the latest version and all my libraries are available.
