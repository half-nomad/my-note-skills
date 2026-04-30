---
name: note-new
description: "Create a new Obsidian note from conversation summary, or copy an existing file to your vault Inbox. Use when: (1) user wants to save conversation content as a note, (2) user wants to send a file to vault Inbox, (3) user wants to organize a specific topic from the conversation into a note. Invoke with /note-new or /note-new [file_path|topic]."
argument-hint: "[file_path|topic]"
---

# Note New

Create a new Obsidian note from conversation summary or copy an existing file to the Inbox.

## Setup (MANDATORY — do this BEFORE any other step)

### 1. Load Config

Read `~/.claude/my-note-skills/config.json`.

**If the config file does not exist:**
1. Tell the user: "First-run setup needed. Running note-setup to detect your vault structure."
2. Invoke the `note-setup` skill (read its SKILL.md and follow the workflow)
3. After setup completes, reload the config

**If `OBSIDIAN_VAULT_PATH` env var is set:** override `config.vault_path` with it.

Resolved variables (used throughout this skill):
- `<vault>` = `config.vault_path`
- `<inbox>` = `<vault>/<config.inbox_dir>`
- `<templates>` = `<vault>/<config.templates_dir>`

### 2. Read Vault Conventions

If `config.claude_md_path` is set and the file exists, read `<vault>/<config.claude_md_path>` for naming conventions and template rules.

### 3. List Available Templates

`ls <templates>/` — confirm template files from `config.templates` are present.

**If any of these reads fail, STOP and report the error. Do not proceed without vault context.**

## Mode Decision

Determine mode from `$ARGUMENTS`:

- **File path exists on disk?** → File Copy mode
- **Text (topic) or empty?** → Note Create mode

## File Copy Mode

1. Copy the file to `<inbox>/` (preserve original filename)
2. If same filename exists, ask user before overwriting
3. Verify the copy succeeded
4. Only allow text-based files (md, txt, json, etc.)

## Note Create Mode

### 1. Analyze Conversation

- If `$ARGUMENTS` is topic text → summarize around that topic
- If empty → summarize the entire conversation's key content

**Summary depth**: Extract not just topics but also decisions, action items, file changes, and external references. The summary structure depends on the chosen template (see step 3).

### 2. Select Template

Choose from `config.templates` based on content type:

| Content Type | Config key | Default file |
|-------------|-----------|--------------|
| General knowledge, conversation summary | `default` | `Document template.md` |
| External article/video/book summary | `reference` | `Reference-note template.md` |
| Zettelkasten atomic idea | `card` | `Card template.md` |
| Study/reading notes | `literature` | `Literature Note template.md` |
| Personal insight, permanent thought | `permanent` | `Permanent note template.md` |
| Blog/social media draft | `post` | `Post-template.md` |
| Project kickoff | `project` | `New Project.md` |
| Quick draft | `draft` | `Draft Note template.md` |

**Default**: `config.templates.default` if unclear.

If the user specifies a template type (e.g., "card", "reference"), use that mapping from config.

If the chosen template file does not exist, fall back to `default` and warn the user.

### 3. Write Note

Read the selected template file from `<templates>/` to get the exact frontmatter fields and body structure, then:

1. **Copy frontmatter keys from the template exactly — and ONLY those keys.** Do not add new keys. If the template lacks a key for the information, put it in the body instead.
2. **Replace Templater placeholders** with actual values:
   - `<% tp.file.creation_date("YYYY-MM-DD") %>` → today's date
   - `<% tp.file.title %>` → note title
   - `modified:` → current datetime (YYYY-MM-DD HH:mm)
3. **Fill values per vault conventions** (from `<vault>/<claude_md>` if available):
   - `tags`, `topics`, `domain` — fill when applicable
   - `source` / `links` — vault internal wiki links only (`[[노트 제목]]`)
   - `source_url` / `links_url` — external URLs only
   - **External conversation/service origin (e.g., Claude Code, ChatGPT) → record in body, NOT in `source`**
   - Empty values are OK; never delete a key
4. **Write body — template-aware summary structure** (after the template's callout block):

   The summary shape depends on the chosen template:

   | Template | Body structure |
   |----------|----------------|
   | `default` (Document) | Bullet points: 핵심 주제 / 주요 발견 / 다음 단계 |
   | `card` | One atomic idea — single paragraph, no headings |
   | `reference` | 출처 / 핵심 주장 / 인용 / 내 해석 |
   | `literature` | 책/자료명 / 핵심 개념 / 인용 / 내 노트 |
   | `permanent` | 핵심 통찰 (1줄) / 근거 / 연결되는 개념 / 반례 |
   | `post` | Hook / 본문 / CTA |
   | `project` | 목표 / 범위 / 마일스톤 / 리스크 / 다음 작업 |
   | `draft` | Free-form notes |

   **Workflow capture** — if the conversation is a coding/work session, also include:
   - 목표 / 진행 단계 / 결과 (성공·실패) / 변경된 파일 / 학습 / 다음 작업

   **Meeting capture** — if the conversation is a discussion with decisions, also include:
   - 안건 / 논의 내용 / 결정 사항 / 액션 아이템 / 미해결 항목

   Do NOT force a structure that doesn't fit the content — adapt to what's actually in the conversation. The table above is the default; deviate when the content demands it.

### 4. Determine Filename

- Clear topic → use topic as filename: `Topic.md`
- Unclear → ask the user

### 5. Save

- Location: `<inbox>/`
- If same filename exists, ask user before overwriting
- Verify the file was saved

## Constraints

- **Frontmatter strictness**: Use ONLY keys defined in the chosen template. Never invent keys. If you think a key is missing, update the template file in `<templates>/` first, then use it — do not add ad-hoc keys to the note.
- **`source` field is for vault wiki links, not free text.** External origins (Claude Code conversation, etc.) belong in the body, not `source`.
- Do NOT use Templater syntax (`<% ... %>`) — insert actual values directly
- Use today's date for `created`, current datetime for `modified`
- Write note body in the same language as the conversation (typically the user's primary language)
- All file paths MUST be resolved through config — never hardcode `~/My_note/` or similar
