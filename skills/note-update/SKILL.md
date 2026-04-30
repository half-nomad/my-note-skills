---
name: note-update
description: "Find and update related documents in your Obsidian vault based on conversation content. Use when: (1) user wants to reflect conversation insights into existing notes, (2) user wants to update a specific vault document, (3) user wants to find and enrich related notes based on discussion. Invoke with /note-update or /note-update [keyword|topic]."
argument-hint: "[keyword|topic]"
---

# Note Update

Find and update related documents in the Obsidian vault based on conversation content.

## Setup (MANDATORY — do this BEFORE any other step)

### 1. Load Config

Read `~/.claude/my-note-skills/config.json`.

**If the config file does not exist:**
1. Tell the user: "First-run setup needed. Running note-setup to detect your vault structure."
2. Invoke the `note-setup` skill (read its SKILL.md and follow the workflow)
3. After setup completes, reload the config

**If `OBSIDIAN_VAULT_PATH` env var is set:** override `config.vault_path` with it.

Resolved variables:
- `<vault>` = `config.vault_path`
- `<search_folders>` = `config.search_folders` (sorted by priority)
- `<excluded>` = `config.excluded_folders`

### 2. Read Vault Conventions

If `config.claude_md_path` is set and the file exists, read `<vault>/<config.claude_md_path>` for naming conventions and frontmatter rules.

**If the read fails, STOP and report the error. Do not proceed without vault context.**

## Workflow

### 1. Analyze Conversation

- If `$ARGUMENTS` provided → use as search keyword/topic
- If empty → extract key topics and keywords from the entire conversation

### 2. Search Related Documents

Search the vault (`<vault>/`) for related documents.

**Search folders** — iterate through `config.search_folders` in priority order. Skip any folder in `config.excluded_folders`.

**Search methods:**
- Filename matching (Glob)
- Frontmatter `tags`, `topics`, `domain` matching (Grep)
- Body keyword matching (Grep)

### 3. Present Candidates

Show search results to the user with relevance scoring. Example:

```
Related documents found:

1. <vault>/30.Expertise/32.Permanent notes/DocA.md
   - topics: [related-topic]
   - relevance: high (3 keyword matches)

2. <vault>/10.Initiative/ProjectB/README.md
   - relevance: medium (1 keyword match)

Which document(s) to update?
```

**Do NOT modify any file until the user selects one.**

### 4. Update Document

After user selects a document:

- Preserve existing document structure and tone
- Update frontmatter `modified` to current datetime
- Integrate new content naturally into existing sections
- Add relevant wikilinks (`[[...]]`) if applicable
- Follow naming/frontmatter conventions from `<vault>/<claude_md>` if loaded

### 5. Confirm Changes

Show before/after diff to the user and get final approval before saving.

### 6. Git Sync (Optional)

If `config.git.enabled` is true, ask the user whether to push changes:

```bash
cd <vault> && git add . && git commit -m "<config.git.commit_prefix>: document-name" && git push origin <config.git.branch>
```

If `config.git.enabled` is false, skip this step.

## Constraints

- **Never modify a document without user confirmation** — always: present candidates → user selects → show diff → user approves
- Follow the existing frontmatter format of each document
- Do NOT use Templater syntax (`<% ... %>`) — insert actual values directly
- Write updates in the same language as the existing document
- All file paths MUST be resolved through config — never hardcode `~/My_note/` or similar
- Honor `excluded_folders` strictly — never search or modify files inside them
