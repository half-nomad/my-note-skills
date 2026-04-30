# my-note-skills (plugin)

Four skills for managing an Obsidian vault:

- **`/note-setup`** — First-run wizard. Detects vault path, folder structure, templates. Saves to `~/.claude/my-note-skills/config.json`. Auto-runs on first use of the other skills.
- **`/note-new [topic|file]`** — Save conversation summary as a new note in your vault Inbox, or copy a file there. Template-aware summary structure (Document / Card / Permanent / Reference / Project / Workflow / Meeting).
- **`/note-update [keyword]`** — Find related notes in the vault and update them with conversation insights. Always asks before modifying.
- **`/note-summary [mode]`** — Structured summary in chat (no save). Modes: `brief` / `bullet` / `meeting` / `til` / `decision` / `workflow`.

See the [marketplace README](../../README.md) for installation instructions.

## Config override

Set `OBSIDIAN_VAULT_PATH` to override the configured vault path:

```bash
export OBSIDIAN_VAULT_PATH=~/another-vault
```
