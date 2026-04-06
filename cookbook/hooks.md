# Add Hooks to the Library

## Overview

This guide explains how to add Claude Code hooks to your library. Hooks are configuration in `.claude/settings.json` that run automatically before/after actions.

## What Are Hooks?

Hooks are automatic actions that fire every time Claude Code:
- **PreToolUse** — Before Claude does something (can block it)
- **PostToolUse** — After Claude does something (cleanup, formatting)
- **Stop** — When Claude finishes a response

## Quick Reference

| Hook Type | When It Runs | Common Use |
|-----------|-------------|------------|
| PreToolUse | Before any tool use | Block dangerous commands, protect files |
| PostToolUse | After tool completes | Format code, run tests, lint |
| Stop | After every response | Auto-commit, cleanup |

## Step-by-Step Process

### Step 1: Determine Scope

**User-level (all projects):**
- Path: `~/.claude/settings.json`
- Use when: Hooks should apply everywhere

**Project-level (in git):**
- Path: `.claude/settings.json`
- Use when: Team should share same hooks

### Step 2: Choose or Create Hooks

From @zodchiii's tweet (8 essential hooks):

| # | Hook Name | Description | Type |
|---|-----------|-------------|------|
| 1 | Auto-format | Run prettier after Write/Edit | PostToolUse |
| 2 | Block dangerous | Block rm -rf, DROP TABLE, etc | PreToolUse |
| 3 | Protect files | Block edits to .env, secrets | PreToolUse |
| 4 | Run tests | Run test suite after edits | PostToolUse |
| 5 | Require tests for PR | Block PR unless tests pass | PreToolUse |
| 6 | Auto-lint | Run ESLint after edits | PostToolUse |
| 7 | Log commands | Log every Bash command | PreToolUse |
| 8 | Auto-commit | Commit on Stop | Stop |

### Step 3: Create Hook Scripts

Create `.claude/hooks/` directory with executable scripts:

```bash
mkdir -p .claude/hooks
chmod +x .claude/hooks/*.sh
```

**Example: block-dangerous.sh**
```bash
#!/usr/bin/env bash
set -euo pipefail
cmd=$(jq -r '.tool_input.command // ""')

dangerous_patterns=(
  "rm -rf"
  "git reset --hard"
  "git push.*--force"
  "DROP TABLE"
  "DROP DATABASE"
)

for pattern in "${dangerous_patterns[@]}"; do
  if echo "$cmd" | grep -qiE "$pattern"; then
    echo "Blocked: '$cmd' matches dangerous pattern" >&2
    exit 2  # Exit 2 blocks the action
  fi
done
exit 0
```

### Step 4: Configure settings.json

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/block-dangerous.sh" }
        ]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/protect-files.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write 2>/dev/null; exit 0" }
        ]
      }
    ]
  }
}
```

### Step 5: Add to .gitignore

Add log file if using command logging:
```
.claude/command-log.txt
```

### Step 6: Commit

```bash
git add .claude/
git commit -m "chore: add Claude Code hooks"
git push
```

---

## Hook Exit Codes

| Exit Code | Effect |
|-----------|--------|
| 0 | Success, continue |
| 1 | Warning logged, continue |
| 2 | **Block the action** (sends error to Claude) |

---

## Common Issues

### Hook Not Running

- Check: Script is executable (`chmod +x`)
- Check: Path is correct in settings.json
- Check: Matcher matches the tool name

### Blocked But Expected

- Review: Add exceptions to your hook script
- Example: Allow specific paths/commands

### Context Overflow

- Use `| tail -5` or `| tail -10` to limit output
- Use `--silent` flag for npm commands

---

## Verification

```bash
# Test a hook manually
.claude/hooks/your-hook.sh

# Check if hooks are loaded
jq '.hooks' .claude/settings.json

# List hook scripts
ls -la .claude/hooks/
```