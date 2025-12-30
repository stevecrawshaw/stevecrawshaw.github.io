# Fixing "bash: uname: command not found" Errors in VS Code Git Bash

## The Problem

You open VS Code, fire up the integrated Git Bash terminal, and are immediately greeted with:

```
bash: uname: command not found
bash: sed: command not found
$
```

Git Bash works perfectly fine when you open it standalone outside of VS Code. Even basic commands like `cat` and `ls` might fail. Something about VS Code's environment is breaking Git Bash's ability to find its own utilities.

## Why This Happens

This issue stems from VS Code's shell integration system combined with extension interference. VS Code tries to inject shell integration scripts into your terminal to provide enhanced features like command tracking, git suggestions, and command decorations. These scripts expect a proper Unix-style PATH and use utilities like `uname` and `sed`.

On Windows, Git Bash uses MINGW64 and has its Unix utilities in `/usr/bin`. When VS Code extensions (particularly the Python extension) modify the terminal environment to activate virtual environments or inject their own scripts, they can corrupt the PATH or prevent bash from finding these utilities.

This is actually a known issue - [VS Code Issue #269581](https://github.com/microsoft/vscode/issues/269581) documents this exact problem and it's been marked as a bug by the VS Code team.

## The Diagnostic Approach

The key to solving this was systematic elimination:

### 1. Confirm It's VS Code-Specific
First, verify that standalone Git Bash works fine. This confirms the issue is with VS Code's configuration, not your Git Bash installation.

### 2. Identify the Shell Integration Script
Run this in standalone Git Bash:
```bash
code --locate-shell-integration-path bash
```

This reveals where VS Code's shell integration script lives - typically in your VS Code installation directory.

### 3. Rule Out Your Own Config Files
Check if your bash startup files contain the problematic commands:
```bash
grep -r "uname" ~/.bashrc ~/.bash_profile ~/.profile
grep -r "sed" ~/.bashrc ~/.bash_profile ~/.profile
```

If these return nothing, you know your own config isn't the culprit.

### 4. Test With Extensions Disabled
This is the crucial test:
```bash
code --disable-extensions
```

Launch VS Code with all extensions disabled. If the errors disappear, you know an extension is causing the problem.

### 5. Binary Search for the Culprit
Enable extensions one by one (or in groups) to identify which one breaks Git Bash. Start with the most likely suspects:
- Python extension
- Claude Code
- GitHub Copilot
- GitLens

In my case, the Python extension was the culprit.

## The Solution

Once you've identified the Python extension as the problem, you need to prevent it from injecting its environment activation scripts into Git Bash.

Add these settings to your VS Code `settings.json`:

```json
{
  "python.terminal.activateEnvironment": false,
  "python.terminal.executeInFileDir": false,
  "terminal.integrated.inheritEnv": false,
  "terminal.integrated.profiles.windows": {
    "Git Bash": {
      "path": "C:\\Users\\YourUsername\\AppData\\Local\\Programs\\Git\\bin\\bash.exe",
      "env": {
        "MSYSTEM": "MINGW64",
        "CHERE_INVOKING": "1"
      }
    }
  },
  "terminal.integrated.defaultProfile.windows": "Git Bash"
}
```

The critical settings are:
- `python.terminal.activateEnvironment: false` - Prevents Python from auto-activating virtual environments in the terminal
- `MSYSTEM: "MINGW64"` - Tells Git Bash which subsystem to use
- `CHERE_INVOKING: "1"` - Ensures bash starts with the proper PATH

## Why This Works

The Python extension tries to be helpful by automatically activating Python virtual environments when you open a terminal. However, its activation scripts expect a Unix-style environment and use commands like `uname` to detect the operating system. When these scripts run before Git Bash has properly initialized its PATH, they fail and pollute your terminal with error messages.

By disabling automatic environment activation, you prevent these scripts from running at startup. You can still manually activate virtual environments when needed with `source venv/Scripts/activate`, but your terminal will start cleanly.

## Alternative Approaches

If you absolutely need automatic Python environment activation, consider:

1. **Use PowerShell for Python work** - Create separate terminal profiles and use PowerShell when working with Python, Git Bash for git operations.

2. **Manually activate environments** - Just run activation scripts yourself when needed rather than having them auto-run.

3. **Use WSL** - Windows Subsystem for Linux provides a native Linux environment where these tools work more reliably.

## Lessons Learned

This debugging experience reinforces a few key principles:

1. **Systematic elimination is your friend** - Don't guess; methodically narrow down the problem space.
2. **Check if it's environment-specific** - Always test whether the issue appears in other contexts.
3. **Extensions can have unintended side effects** - Popular extensions that modify the environment can conflict in unexpected ways.
4. **Read the error messages literally** - "command not found" really does mean the PATH is broken, not that the command doesn't exist.

The beauty of this solution is that it's minimal - we're not disabling shell integration entirely or resorting to complex workarounds. We're simply telling one extension to be less aggressive about modifying the terminal environment, which is the right balance between functionality and stability.
