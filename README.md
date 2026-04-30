# my-note-skills

Obsidian vault note management skills for Claude Code, Claude Desktop (Cowork), and Claude.ai web.

## What it does

- **`/note-new`** — Save conversation summary as a new note in your Obsidian Inbox, or copy a file there. Template-aware summary structure (Document / Card / Permanent / Meeting / Workflow / etc.)
- **`/note-update`** — Find related notes in your vault and update them with conversation insights
- **`/note-summary`** — Generate a structured summary in chat without saving (modes: brief / bullet / meeting / til / decision / workflow)
- **`/note-setup`** — First-run wizard that detects your vault structure (auto-runs on first use)

## Install

### Claude.ai web (chat) — individual skills as ZIP

Each skill is packaged as a separate `.zip` file (the official Claude.ai upload format):

1. Open **Claude.ai → Settings → Features → Skills → Upload skill**
2. Upload each `.zip` you need:
   - `dist/note-setup.zip`
   - `dist/note-new.zip`
   - `dist/note-update.zip`
   - `dist/note-summary.zip`

> Reference: [Agent Skills - platform.claude.com](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
> *"Upload your own Skills as zip files through Settings > Features."*

### Claude Desktop (Cowork) — bundled plugin

Use `dist/my-note-skills.plugin` (a zip archive with `.plugin` extension):

1. Open Claude Desktop → Settings → Extensions/Plugins
2. Click **Install from file** and select `my-note-skills.plugin`
3. Approve

### Claude Code (CLI)

```bash
unzip dist/my-note-skills.plugin -d ~/.claude/plugins/local/my-note-skills
# Inside Claude Code:
/plugin install ~/.claude/plugins/local/my-note-skills
```

Or extract just the skills:

```bash
unzip dist/my-note-skills.plugin -d /tmp/mns
cp -r /tmp/mns/skills/* ~/.claude/skills/
```

## First Run

The first time you call `/note-new` or `/note-update`, the `note-setup` wizard runs automatically and:

1. Asks for your Obsidian vault path (defaults: `~/My_note/`, `~/Documents/Obsidian/`, `~/Obsidian/`)
2. Scans the top-level folder structure
3. Detects Inbox, Templates, and search folders
4. Reads your vault's `CLAUDE.md` if present (for conventions)
5. Detects git config (branch, sync workflow)
6. Saves config to `~/.claude/my-note-skills/config.json`

You can re-run setup any time with `/note-setup`.

## Config

Location: `~/.claude/my-note-skills/config.json`

```json
{
  "vault_path": "/Users/you/Obsidian",
  "inbox_dir": "00.Inbox",
  "templates_dir": "90.Settings/92.Templates",
  "search_folders": [
    {"priority": 1, "path": "30.Expertise", "label": "Knowledge base"},
    ...
  ],
  "excluded_folders": ["90.Settings", ".obsidian"],
  "templates": {
    "default": "Document template.md",
    "card": "Card template.md",
    ...
  },
  "git": {
    "enabled": true,
    "branch": "master",
    "commit_prefix": "Update"
  }
}
```

### Override via env var

```bash
export OBSIDIAN_VAULT_PATH=~/another-vault
```

When set, this overrides `vault_path` from the config file.

## Folder structure expectations

The plugin works with any folder layout, but auto-detection is best for PARA-style vaults:

- `00.Inbox/` — temp notes
- `10.Initiative/` or `1.Projects/` — projects
- `20.Diary/` — daily/diary
- `30.Expertise/` or `3.Resources/` — knowledge base
- `40.Action/` or `4.Archives/` — completed/archive
- `90.Settings/92.Templates/` — Obsidian templates

Non-PARA layouts work too — the setup wizard will ask you to map folders manually.

## License

MIT
