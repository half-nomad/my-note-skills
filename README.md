# my-note-skills

A Claude Code plugin marketplace hosting **my-note-skills** — four Obsidian vault note-management skills.

## Skills included

| Skill | Purpose |
|-------|---------|
| `/note-setup` | First-run wizard. Detects vault structure (Inbox, Templates, search folders) and saves config. Auto-runs on first use. |
| `/note-new [topic\|file]` | Save conversation summary as a new note (template-aware: Document/Card/Permanent/Reference/Project/Workflow/Meeting). |
| `/note-update [keyword]` | Find related notes in your vault and update them with conversation insights. |
| `/note-summary [mode]` | Structured summary in chat (no save). Modes: `brief`/`bullet`/`meeting`/`til`/`decision`/`workflow`. |

---

## Install

### Claude Code (recommended — marketplace)

```
/plugin marketplace add half-nomad/my-note-skills
/plugin install my-note-skills@my-note-skills
```

To remove later:
```
/plugin uninstall my-note-skills
/plugin marketplace remove my-note-skills
```

### Claude Code (direct local)

```bash
git clone https://github.com/half-nomad/my-note-skills.git ~/.claude/plugins/local/my-note-skills
# inside Claude Code:
/plugin install ~/.claude/plugins/local/my-note-skills/plugins/my-note-skills
```

### Claude Desktop (Cowork)

Use the bundled `.plugin` file from [Releases](https://github.com/half-nomad/my-note-skills/releases) or `dist/my-note-skills.plugin`:

1. Open Claude Desktop → Settings → Extensions/Plugins
2. **Install from file** → select `my-note-skills.plugin`

### Claude.ai web (chat) — individual skills

Claude.ai web requires uploading skills individually as `.zip` files via Settings → Features → Skills:

- `dist/note-setup.zip`
- `dist/note-new.zip`
- `dist/note-update.zip`
- `dist/note-summary.zip`

> Reference: [Agent Skills - platform.claude.com](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
> *"Upload your own Skills as zip files through Settings > Features."*

---

## First Run

The first time you call `/note-new` or `/note-update`, the `note-setup` wizard runs automatically and:

1. Asks for your Obsidian vault path (defaults: `~/My_note/`, `~/Documents/Obsidian/`, `~/Obsidian/`)
2. Scans top-level folder structure (auto-detects PARA-style layouts)
3. Detects Inbox, Templates, and search folders
4. Reads your vault's `CLAUDE.md` if present (for naming conventions)
5. Detects git config (branch, sync workflow)
6. Saves config to `~/.claude/my-note-skills/config.json`

Re-run any time with `/note-setup`.

## Config

Location: `~/.claude/my-note-skills/config.json`

```json
{
  "vault_path": "/Users/you/Obsidian",
  "inbox_dir": "00.Inbox",
  "templates_dir": "90.Settings/92.Templates",
  "search_folders": [
    {"priority": 1, "path": "30.Expertise", "label": "Knowledge base"}
  ],
  "excluded_folders": ["90.Settings", ".obsidian"],
  "templates": {
    "default": "Document template.md",
    "card": "Card template.md"
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

## Repository layout

```
my-note-skills/                              ← marketplace root
├── .claude-plugin/marketplace.json          ← marketplace manifest
├── plugins/my-note-skills/                  ← the plugin
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── note-setup/SKILL.md
│       ├── note-new/SKILL.md
│       ├── note-update/SKILL.md
│       └── note-summary/SKILL.md
├── dist/                                    ← release artifacts
│   ├── my-note-skills.plugin                ← bundled plugin (zip)
│   └── note-*.zip                           ← individual skills for Claude.ai
├── LICENSE
└── README.md
```

## License

MIT — see [LICENSE](LICENSE).
