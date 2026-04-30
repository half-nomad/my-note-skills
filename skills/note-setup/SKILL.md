---
name: note-setup
description: "Initial setup for my-note-skills — scans the user's Obsidian vault, detects folder structure (Inbox, Templates, search folders), and saves configuration to ~/.claude/my-note-skills/config.json. Use when: (1) user runs /note-setup, (2) note-new or note-update detects missing config and triggers setup, (3) user wants to reconfigure the vault path or folder mapping. Invoke with /note-setup or /note-setup [vault_path]."
argument-hint: "[vault_path]"
---

# Note Setup

First-run configuration wizard for my-note-skills. Detects Obsidian vault structure and stores it in a config file so `note-new` and `note-update` can adapt to any user's folder system.

## Config Location

`~/.claude/my-note-skills/config.json`

Environment variable `OBSIDIAN_VAULT_PATH` overrides the `vault_path` in config when set.

## Workflow

### 1. Determine Vault Path

Resolution order:

1. `$ARGUMENTS` if provided (treat as absolute or `~/`-prefixed path)
2. `$OBSIDIAN_VAULT_PATH` environment variable
3. Ask the user, suggesting common defaults:
   - `~/My_note/`
   - `~/Documents/Obsidian/`
   - `~/Obsidian/`
   - User-typed path

Verify the path exists with `ls`. If not, ask again.

### 2. Detect Folder Structure

Run `ls -la <vault_path>` and analyze top-level entries.

**Auto-detect rules** (case-insensitive substring match):

| Config key | Match patterns | Fallback |
|-----------|---------------|----------|
| `inbox_dir` | `*inbox*`, `00.Inbox`, `0.Inbox` | `00.Inbox` (create if missing) |
| `templates_dir` | `*template*`, `92.Templates`, `_templates` | ask user |
| `claude_md_path` | `CLAUDE.md` (root) | optional, leave empty if absent |

**Search folders** (for `note-update`): list all top-level directories and present them to the user with suggested priority. Ask which to include and in what order.

Common PARA-style patterns to recognize:
- `10.*` / `1.Projects` → priority 2
- `20.*` / `2.Areas` → priority 4
- `30.*` / `3.Resources` → priority 1 (knowledge base)
- `40.*` / `4.Archives` → priority 3
- `00.Inbox` → priority 5
- `90.Settings` → **excluded by default**

### 3. Read CLAUDE.md (if exists)

If `<vault>/CLAUDE.md` exists, read it and extract:
- Naming conventions
- Git workflow (branch name, commit style)
- Tag/topic taxonomies

Surface findings to the user and ask whether to incorporate.

### 4. Detect Templates

If `templates_dir` found, list its contents (`ls`). For each `.md` file, infer the template type from filename. Present this mapping to the user for confirmation:

```
Detected templates:
- Document template.md      → general (default)
- Card template.md          → zettelkasten atomic
- Reference-note template.md → external source
- Literature Note template.md → study notes
- Permanent note template.md → permanent insight
- Post-template.md          → blog/social draft
- Draft Note template.md    → quick draft
- New Project.md            → project kickoff
```

Allow user to add/remove/rename mappings.

### 5. Detect Git Configuration

If `<vault>/.git` exists:
- Detect default branch: `git -C <vault> symbolic-ref refs/remotes/origin/HEAD` or fallback to `main` / `master`
- Confirm with user

If no git, set `git_enabled: false`.

### 6. Write Config

Save to `~/.claude/my-note-skills/config.json`:

```json
{
  "version": "0.1.0",
  "vault_path": "/Users/.../My_note",
  "inbox_dir": "00.Inbox",
  "templates_dir": "90.Settings/92.Templates",
  "claude_md_path": "CLAUDE.md",
  "search_folders": [
    {"priority": 1, "path": "30.Expertise", "label": "Knowledge base"},
    {"priority": 2, "path": "10.Initiative", "label": "Projects"},
    {"priority": 3, "path": "40.Action", "label": "Active work"},
    {"priority": 4, "path": "20.Diary", "label": "Diary"},
    {"priority": 5, "path": "00.Inbox", "label": "Inbox"}
  ],
  "excluded_folders": ["90.Settings", ".obsidian", ".trash"],
  "templates": {
    "default": "Document template.md",
    "card": "Card template.md",
    "reference": "Reference-note template.md",
    "literature": "Literature Note template.md",
    "permanent": "Permanent note template.md",
    "post": "Post-template.md",
    "draft": "Draft Note template.md",
    "project": "New Project.md"
  },
  "git": {
    "enabled": true,
    "branch": "master",
    "commit_prefix": "Update"
  }
}
```

Create the parent directory first: `mkdir -p ~/.claude/my-note-skills/`

### 7. Confirmation Summary

Show the user a summary of detected config and ask for final approval before writing. After save, show:

```
✓ Setup complete. Config saved to ~/.claude/my-note-skills/config.json

You can now use:
  /note-new [topic|file]       — Create a new note
  /note-update [keyword]       — Find and update related notes

To reconfigure later, run /note-setup again.
```

## Constraints

- **Always ask before writing config** — show the resolved structure first
- **Never modify the vault during setup** — read-only inspection only
- **Setup is idempotent** — running again overwrites existing config (after confirmation)
- Detect language from CLAUDE.md or vault content; default to user's conversation language
- If env var `OBSIDIAN_VAULT_PATH` is set, prefer it but still confirm with user on first run
