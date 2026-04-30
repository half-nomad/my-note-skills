---
name: note-summary
description: "Summarize the current conversation or workflow into a structured output without saving to disk. Use when: (1) user wants a quick recap of what was discussed, (2) user wants meeting-notes-style structure, (3) user wants a TIL/decision-log entry, (4) user wants to review a summary before deciding whether to save as a note. Invoke with /note-summary or /note-summary [mode]."
argument-hint: "[brief|bullet|meeting|til|decision|workflow]"
---

# Note Summary

Generate a structured summary of the current conversation/workflow. **Output to chat only — does NOT save to vault.** If the user wants to save, suggest piping to `/note-new`.

## Mode Selection

Determine mode from `$ARGUMENTS`. If empty, infer from conversation content; default to `bullet`.

| Mode | When to use | Output style |
|------|-------------|--------------|
| `brief` | Quick recap | 3 sentences max |
| `bullet` | General summary (default) | 5–8 bullets |
| `meeting` | Discussion / decisions / multiple participants | Structured: Agenda → Discussion → Decisions → Action Items |
| `til` | "Today I learned" — knowledge capture | Bullets of learnings + key takeaway |
| `decision` | Decision log | Context → Options → Decision → Rationale → Trade-offs |
| `workflow` | Coding/work session capture | Goal → Steps taken → Result → Files changed → Next steps |

User can override via `$ARGUMENTS` (e.g., `/note-summary meeting`).

## Output Templates

### `brief` mode

```
**요약**
[3 문장 이내. 핵심만.]
```

### `bullet` mode

```
**대화 요약**

- [핵심 주제 1]
- [핵심 주제 2]
- [중요한 결정/발견]
- [열린 질문 or 다음 단계]

(5–8 bullets, 한 줄당 한 아이디어)
```

### `meeting` mode

```
## 회의 요약

**일시**: [today's date]
**주제**: [main topic]

### 안건
- [agenda 1]
- [agenda 2]

### 논의 내용
- [discussion point 1]
- [discussion point 2]

### 결정 사항
- ✅ [decision 1]
- ✅ [decision 2]

### 액션 아이템
- [ ] [action] — [owner if mentioned]
- [ ] [action]

### 미해결 / 다음 회의
- [open question]
```

### `til` mode

```
## TIL — [today's date]

**오늘 배운 것**

- 💡 [learning 1] — [brief context]
- 💡 [learning 2]
- 💡 [learning 3]

**핵심 takeaway**: [one-sentence distillation]

**참고**: [external sources mentioned, if any]
```

### `decision` mode

```
## 결정 로그

**날짜**: [today's date]
**결정 사항**: [the decision]

### 배경 / 문제
[why this decision was needed]

### 검토한 옵션
1. **[Option A]** — [pros/cons]
2. **[Option B]** — [pros/cons]
3. **[Option C]** — [pros/cons]

### 선택
**[chosen option]**

### 근거
- [reason 1]
- [reason 2]

### 트레이드오프 / 알려진 제약
- [trade-off 1]
- [trade-off 2]

### 재검토 조건
[when to revisit, if applicable]
```

### `workflow` mode

```
## 작업 흐름 요약

**날짜**: [today's date]
**목표**: [what was being attempted]

### 진행 단계
1. [step 1 — what was tried]
2. [step 2 — what was tried]
3. [step 3 — what was tried]

### 결과
- ✅ [what worked]
- ❌ [what didn't work]

### 변경된 파일 (감지된 경우)
- `path/to/file.ext` — [nature of change]

### 학습 / 회고
- [insight gained]

### 다음 작업
- [ ] [next step 1]
- [ ] [next step 2]
```

## Workflow

### 1. Determine Mode

- Use `$ARGUMENTS` if provided
- Otherwise infer:
  - Multiple decisions discussed → `decision` or `meeting`
  - Coding/file-change work → `workflow`
  - Learning new concepts → `til`
  - Planning meeting → `meeting`
  - Otherwise → `bullet`

### 2. Extract Content

Scan the entire conversation:
- Topics raised
- Decisions made
- Files touched
- Commands run
- External references (URLs, tools)
- Action items / TODOs
- Open questions

### 3. Render

Output the chosen template **inline in chat**. Do NOT write to disk.

Match the conversation's primary language (typically Korean).

### 4. Suggest Next Step

After the summary, offer:

```
---
이 요약을 노트로 저장하려면 → /note-new
관련 기존 노트를 업데이트하려면 → /note-update
다른 형식으로 다시 요약 → /note-summary [brief|meeting|til|decision|workflow]
```

## Constraints

- **Read-only**: Never write files. Never modify the vault. Never call git.
- **Chat output only**: All content goes to the assistant message, not to disk.
- **No config dependency**: This skill works without `~/.claude/my-note-skills/config.json` — it's pure summarization.
- Match conversation language
- If conversation is too short (< 3 substantive turns), tell the user and suggest waiting until there's more content
- Do not invent facts not present in the conversation
