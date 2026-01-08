---
layout: post
title: "Git Hooks for Secret Detection: A Journey Through Wrong Tools and Right Solutions"
date: 2026-01-08
categories: git security development-tools
---

## Git Hooks for Secret Detection

I recently spent time trying to prevent API keys and tokens from being accidentally committed to Git repositories. The goal was simple: scan staged files before each commit and block the commit if secrets were detected.

I reached for the wrong tool first, learned valuable lessons about choosing the right solution for the job, and ultimately implemented a robust approach that works across all my projects.

## The False Start: Claude Code Hooks

My first instinct was to use Claude Code's `PreToolUse` hook system since I wanted to catch commits made through Claude's Bash tool. The initial pattern looked like this:

```json
{
  "matcher": "Bash(git commit*)",
  "hooks": [{
    "type": "command",
    "command": "bash scan_secrets.sh . --staged"
  }]
}
```

It didn't work. And working with Claude we got stuck in a few loops of configuring the hook and testing.

I refined it:

```json
{
  "matcher": "Bash(*git commit*)"
}
```

Still nothing.

## Root Cause: Understanding Hook Matchers

After investigation, I discovered the real issue: **Claude Code hook matchers only match tool names, not command content.**

The matcher field accepts:

- `"Bash"` - matches the Bash tool exactly
- `"Edit|Write"` - regex patterns for tool names
- `"*"` - all tools

Patterns like `Bash(*git commit*)` are treated as literal strings that match nothing. There's no wildcard syntax for command filtering in the matcher itself.

Even if the pattern worked, Claude Code hooks have fundamental limitations:

1. **Only catches Claude-initiated commits** - Manual `git commit` in the terminal bypasses it entirely
2. **Fires on ALL Bash commands** - Not just git commits, so you'd need complex filtering logic
3. **Adds significant complexity** - You'd have to parse JSON input within your script to extract the actual command

## The Right Tool: Native Git Hooks

The epiphany was realizing I should use git's built-in pre-commit hook system. This is exactly what it's designed for.

**Why git pre-commit hooks are the correct solution:**

- Catches **all commits** regardless of origin (Claude, terminal, IDE, etc.)
- Works across **all your projects** automatically
- No complex filtering or special configuration needed
- Standard, portable git workflow
- Widely supported and understood

## Implementation: Global Git Hook Templates

Rather than adding hooks to individual repositories, I set up a global git template that automatically installs hooks in new repositories:

### Step 1: Create the Hook Template

```bash
mkdir -p ~/.git-templates/hooks
```

### Step 2: Write the Pre-Commit Hook

Create `~/.git-templates/hooks/pre-commit`:

```bash
#!/bin/bash
# Global pre-commit hook - scans for secrets before allowing commits

SCAN_SCRIPT="$HOME/.config/claude-code/scripts/scan_secrets.sh"

if [[ -f "$SCAN_SCRIPT" ]]; then
    bash "$SCAN_SCRIPT" . --staged
    exit_code=$?
    if [[ $exit_code -ne 0 ]]; then
        echo ""
        echo "=========================================="
        echo "ERROR: Secret detected in staged files!"
        echo "=========================================="
        exit 1
    fi
fi

exit 0
```

### Step 3: Configure Git Globally

```bash
chmod +x ~/.git-templates/hooks/pre-commit
git config --global init.templateDir ~/.git-templates
```

That's it. All future `git init` commands will automatically get the hook.

### Step 4: Apply to Existing Repositories

For existing repositories, I created an install script:

```bash
#!/bin/bash
# Install global git hooks to current repository

TEMPLATE_DIR="$HOME/.git-templates/hooks"

if [[ ! -d ".git" ]]; then
    echo "ERROR: Not a git repository"
    exit 1
fi

cp "$TEMPLATE_DIR/pre-commit" .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
echo "Pre-commit hook installed successfully"
```

Run with: `bash ~/.config/claude-code/scripts/install-hooks.sh`

Or simply reinitialize: `git init` (safe, preserves history)

## Testing and Results

I tested the implementation with a file containing a fake OpenAI API key:

```bash
git add test_hook_secret.py
git commit -m "test: this should be blocked"
```

Result:

```
--- Security Scan Initiated ---
[!] BLOCKING: Potential secret in test_hook_secret.py
  Pattern matches:
    2:api_key ********************456"

RESULT: FAIL - Secrets detected. Commit blocked.
```

✅ **Working perfectly.** The hook caught the secret and prevented the commit.

## Integration with Global Configuration

This setup is documented in my global Claude Code configuration (`~/.claude/CLAUDE.md`) and provides a consistent secret detection strategy across all projects.

### File Locations

Once configured, the following files manage the global secret detection system:

| File | Purpose |
|------|---------|
| `~/.git-templates/hooks/pre-commit` | Global git pre-commit hook template |
| `~/.config/claude-code/scripts/scan_secrets.sh` | Secret scanning script (pattern matching) |
| `~/.config/claude-code/scripts/install-hooks.sh` | Hook installer for existing repositories |
| `~/.claude/CLAUDE.md` | User-level configuration and guidelines |

### Handling Blocked Commits

When the pre-commit hook detects a potential secret and blocks a commit:

1. **Identify the secret** - The hook output shows the file and line number
2. **Remove the hardcoded secret** - Delete the sensitive value from the file
3. **Use environment variables instead** - Reference secrets via `process.env` or `os.environ`
4. **Add to .gitignore** - Ensure `.env` and similar files are ignored
5. **Try the commit again** - The hook should now allow it

For example, instead of:

```python
API_KEY = "sk-proj-1234567890..."  # BAD - hardcoded
```

Use:

```python
import os
API_KEY = os.environ.get("OPENAI_API_KEY")  # GOOD - from environment
```

### Bypassing the Hook (Not Recommended)

In rare cases where you're certain something is a false positive:

```bash
git commit --no-verify
```

This skips the pre-commit hook, but should be avoided unless absolutely necessary. If you encounter frequent false positives, consider adjusting the scan patterns.

### Best Practices

- **Always use environment variables** for secrets, API keys, and credentials
- **Add `.env` files to `.gitignore`** to prevent accidental commits
- **Use secret managers** (AWS Secrets Manager, HashiCorp Vault, etc.) for production environments
- **Rotate credentials** if they were accidentally committed before being caught
- **Never commit then delete** - Once in git history, credentials are compromised

## Key Learnings

### 1. Right Tool for the Job

Don't force a tool into a purpose it wasn't designed for. Claude Code hooks are great for pre-flight checks on Claude-initiated operations, but git pre-commit hooks are the standard, portable solution for protecting commits.

### 2. Understand Your Tool's Constraints

Before implementing, invest time understanding what a tool can and can't do:

- Claude Code matchers filter by tool name, not command content
- Git hooks run locally and catch all commits
- Each has different strengths

### 3. Testing Matters

A simple test with real secrets (simulated) caught the issue immediately. Don't assume documentation matches implementation.

### 4. Global Solutions Scale Better

Setting up templates at the user level means every future project automatically gets protection. No per-project configuration needed.

### 5. Documentation Matters

Documenting the system in a global configuration file (`CLAUDE.md`) ensures consistency and provides clear guidance for handling blocked commits across all projects.

## What Secrets Are Detected

The scanning script detects patterns for:

- **OpenAI API keys**: `sk-` followed by 48+ characters
- **GitHub tokens**: `ghp_`, `gho_`, `ghu_`, `ghs_`, `ghr_` prefixes
- **AWS access keys**: `AKIA` prefix
- **Generic API keys**: Variables named `API_KEY`, `api_key`, `apikey`
- **Environment files**: `.env`, `.env.local`, `.env.production`

## Conclusion

What started as debugging a Claude Code hook became a lesson in choosing the right tool and implementing a better solution. The git pre-commit hook approach is now protecting all my repositories automatically, with zero per-project configuration required.

Sometimes the best solution isn't the fanciest one—it's the one designed specifically for the job.

---

**Tools used:**

- Git pre-commit hooks (native)
- Bash script for pattern matching
- Global git templates for scaling

**Next steps:** Apply this pattern to all existing repositories with `git init` or the install script, then enjoy automatic secret protection on every commit.
