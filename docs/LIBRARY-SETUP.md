# Library Setup - Complete Documentation

## Model Overview

Based on Indie Dev Dan's setup from https://www.youtube.com/watch?v=_vpNQ6IwP9w

```
┌─────────────────────────────────────────────────────────────┐
│          SOURCE REPOS (your project repos)                   │
│   Where skills/agents/prompts actually live               │
└──────────────────────┬────────────────────────────────────┘
                       │ references in
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              hd-library (YOUR REFERENCE CATALOG)          │
│   Private repo - your personal reference library      │
│   Contains:                                             │
│   - library.yaml (catalog of pointers)                │
│   - SKILL.md (meta-skill for /library commands)        │
│   - .claude/hooks/ (your hooks)                       │
└──────────────────────┬────────────────────────────────────┘
                       │
    ┌──────────────────┼──────────────────┐
    │                  │                  │
    ▼                  ▼                  ▼
┌─────────┐    ┌─────────┐    ┌─────────┐
│ Mac     │    │ Cloud   │    │ Team    │
│ Mini    │    │ Sandbox│    │ Members│
└─────────┘    └─────────┘    └─────────┘
     ▲              ▲              ▲
     └──────────────┼──────────────┘
           sync via /library sync
```

---

## Repository Purposes

### hd-library (hdiesel323/hd-library)
**Purpose:** Your personal reference catalog
**Type:** Private
**Use for:** Skills, agents, prompts you want accessible everywhere

### the-library (hdiesel323/the-library)
**Purpose:** Forkable template for others
**Type:** Public (or private if preferred)
**Use for:** Sharing your library system with team/template users

### idd-library (hdiesel323/idd-library)
**Purpose:** TODO - needs distinction
**Status:** Same description as hd-library - requires clarification

---

## Quick Start

### 1. Fork the Template (first-time only)
```bash
gh repo fork disler/the-library --private --clone=false
# Or via GitHub UI
```

### 2. Clone to Global Skills Directory
```bash
mkdir -p ~/.claude/skills/library
git clone <your-fork-url> ~/.claude/skills/library
```

### 3. Configure SKILL.md
Open `~/.claude/skills/library/SKILL.md` and update:
- **LIBRARY_REPO_URL**: Your fork URL
- **LIBRARY_YAML_PATH**: `~/.claude/skills/library/library.yaml`
- **LIBRARY_SKILL_DIR**: `~/.claude/skills/library/`

### 4. Test It Works
```bash
/library list
/library search <keyword>
```

---

## Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `/library install` | First-time setup | Fork, clone, configure |
| `/library add <details>` | Register new entry | Add skill from project |
| `/library use <name>` | Pull/install item | Install from catalog |
| `/library push <name>` | Push changes back | Update source |
| `/library remove <name>` | Remove from catalog | Unregister item |
| `/library list` | Show catalog | See all entries |
| `/library sync` | Re-pull all items | Refresh everything |
| `/library search <keyword>` | Find by keyword | Search catalog |

---

## Adding Hooks

Hooks are stored in `.claude/` in the library repo:

```
hd-library/
├── .claude/
│   ├── settings.json    ← Hook configuration
│   └── hooks/
│       ├── block-dangerous.sh
│       ├── protect-files.sh
│       ├── log-commands.sh
│       ├── require-tests-for-pr.sh
│       └── auto-commit.sh
├── library.yaml
├── SKILL.md
└── cookbook/
    └── hooks.md
```

### Adding a New Hook

1. Create script in `.claude/hooks/`
2. Make executable: `chmod +x .claude/hooks/your-hook.sh`
3. Add to `.claude/settings.json`:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/your-hook.sh" }
        ]
      }
    ]
  }
}
```

### Hook Exit Codes

| Exit Code | Effect |
|----------|--------|
| 0 | Success, continue |
| 1 | Warning logged, continue |
| 2 | Block the action |

### Enable Hooks on a New Device

```bash
# Pull the latest
cd ~/.claude/skills/library
git pull

# Hooks auto-load since .claude/settings.json is in the repo
```

---

## Sync Workflow

### On Your Main Device
```bash
# After creating new skills/agents/prompts:
/library add <name> --source /path/to/SKILL.md
/library push <name>
git push
```

### On Other Devices
```bash
# Pull updates
cd ~/.claude/skills/library
git pull

# Or use library sync
/library sync

# Use specific item
/library use <name>
```

---

## Best Practices

1. **Source of Truth**: Keep skills where they're used, not in library
2. **References Only**: Library stores pointers, not copies
3. **Private by Default**: Your skills should be private
4. **Hook Everything**: Automate safety nets via hooks
5. **Atomic Commits**: One skill/agent per commit

---

## Troubleshooting

### Hooks Not Running
```bash
# Check permissions
ls -la .claude/hooks/*.sh
chmod +x .claude/hooks/*.sh
```

### Library Not Found
```bash
# Verify path
ls ~/.claude/skills/library/SKILL.md

# Re-install if needed
/library install
```

### Sync Issues
```bash
# Manual sync
cd ~/.claude/skills/library
git pull --rebase
```

---

## References

- Video: https://www.youtube.com/watch?v=_vpNQ6IwP9w
- Template: https://github.com/disler/the-library
- Docs: https://code.claude.com/docs/en/hooks