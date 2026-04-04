# Add Items to the Library - Complete Guide

## Overview

This guide explains the correct workflow for adding skills, agents, and prompts to your library catalog.

## Quick Reference

| Item Type | Source Pattern | Target Directory |
|----------|----------------|-----------------|
| Skill | `SKILL.md` | `.claude/skills/` or `~/.claude/skills/` |
| Agent | `AGENT.md` | `.claude/agents/<category>/` |
| Prompt | `.md` file | `.claude/commands/` |

---

## Step-by-Step Process

### Step 1: Sync the Library Repo

Always pull latest before making changes:

```bash
cd ~/.claude/skills/library
git pull
```

### Step 2: Analyze the Source

Determine what you're adding:

**Local Files:**
- Check if `SKILL.md` exists → Skill
- Check if `AGENT.md` exists → Agent
- Check if file ends in `.md` but no SKILL/AGENT → Prompt

**GitHub Repository:**
```bash
# List contents
gh api repos/OWNER/REPO/contents --jq '.[].name'

# Check for skills folder
gh api repos/OWNER/REPO/contents/skills --jq '.[].name'

# Get skill description from frontmatter
gh api "repos/OWNER/REPO/contents/skills/SKILL_NAME/SKILL.md" -q '.content' | base64 -d
```

### Step 3: Determine Installation Location

From `library.yaml`:

```yaml
default_dirs:
  skills:
    - default: .claude/skills/        # Project-level
    - global: ~/.claude/skills/      # Machine-level
  agents:
    - default: .claude/agents/      # Project-level
    - global: ~/.claude/agents/     # Machine-level
  prompts:
    - default: .claude/commands/    # Project-level
    - global: ~/.claude/commands/ # Machine-level
```

**Decision:**
- "global" / "machine-level" → use `~/.claude/...`
- Project-specific → use `.claude/...`
- Default (not specified) → `.claude/...`

### Step 4: Install the Item

**For Local Items:**
```bash
# Determine target path
TARGET_DIR="~/.claude/skills"  # or ~/.claude/agents/, ~/.claude/commands/

# Copy the item
cp -r /path/to/source-item $TARGET_DIR/
```

**For GitHub Items:**
```bash
# Clone shallow
git clone --depth 1 https://github.com/OWNER/REPO.git /tmp/repo

# Copy specific item
cp -r /tmp/repo/path/to/item ~/.claude/skills/

# Cleanup
rm -rf /tmp/repo
```

### Step 5: Add to Catalog

Edit `library.yaml`:

```yaml
library:
  skills:           # or agents, prompts
  - name: item-name
    description: Description of what it does
    source: ~/.claude/skills/item-name/SKILL.md  # or GitHub URL
    # requires: [skill:other-skill]  # omit if no dependencies
```

### Step 6: Commit and Push

```bash
cd ~/.claude/skills/library
git add library.yaml
git commit -m "feat: add item-name to library"
git push
```

---

## Examples

### Adding a Local Skill

```bash
# 1. Sync
cd ~/.claude/skills/library && git pull

# 2. Copy skill
cp -r /path/to/my-skill ~/.claude/skills/

# 3. Add to catalog
# Edit library.yaml:
- name: my-skill
  description: My custom skill description
  source: ~/.claude/skills/my-skill/SKILL.md

# 4. Commit
git add library.yaml && git commit -m "feat: add my-skill" && git push
```

### Adding from GitHub

```bash
# 1. Find the repo and check contents
gh api repos/OWNER/REPO/contents

# 2. Clone shallow
git clone --depth 1 https://github.com/OWNER/REPO.git /tmp/repo

# 3. Check structure
ls /tmp/repo/skills/

# 4. Copy to global skills
cp -r /tmp/repo/skills/my-skill ~/.claude/skills/

# 5. Add to library.yaml
- name: my-skill
  description: Description from SKILL.md frontmatter
  source: https://github.com/OWNER/REPO/blob/main/skills/my-skill/SKILL.md

# 6. Commit
cd ~/.claude/skills/library && git add library.yaml && git commit && git push
```

### Adding Multiple Skills from a Repo

```bash
# 1. List all skills
gh api repos/OWNER/REPO/contents/skills --jq '.[].name'

# 2. Clone
git clone --depth 1 https://github.com/OWNER/REPO.git /tmp/repo

# 3. Copy all
for dir in /tmp/repo/skills/*/; do
  cp -r "$dir" ~/.claude/skills/
done

# 4. Add each to library.yaml with descriptions extracted from frontmatter
```

---

## Common Issues

### Wrong Source Path

If item isn't found, check the actual path:
- `~/.claude/skills/skill-name/SKILL.md` vs `~/.claude/skills/skill-name.md`

### Duplicate Entries

Check if entry already exists:
```bash
grep -i "skill-name" library.yaml
```

### Dependencies Missing

Check for `requires:` field in source files and add those first:
```bash
grep -r "skill:" path/to/skill/SKILL.md
```

---

## Verification

After adding, verify:

```bash
# Check catalog count
python3 -c "
import yaml
lib = yaml.safe_load(open('library.yaml'))
print(f'Skills: {len(lib[\"library\"][\"skills\"])}')
print(f'Agents: {len(lib[\"library\"][\"agents\"])}')
print(f'Prompts: {len(lib[\"library\"][\"prompts\"])}')
"
```