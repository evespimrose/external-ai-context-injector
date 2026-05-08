# Trae + doc-context Workflow Guide

## What This Workflow Does
Cuts Claude conversation-token usage by routing requirement structuring to an external AI (Trae, GPT-4o, etc.) and loading the result via a file reference instead of pasting raw text.

---

## Core Rule
- **< 50 tokens / 1-turn task** → paste raw text directly
- **Everything else** → structure with external AI → save as `cxt*.md` → `/doc-context`

---

## Step-by-Step

### 1. Collect raw feedback
Write your notes in any language, any style.

### 2. Hand off to Trae with `context-sharer`
```
Structure this into cxt{N}.md and prepare for Claude handover.
[paste your notes]
```
Trae: creates `docs/contextmd/cxt{N}.md` → copies `/doc-context docs\contextmd\cxt{N}.md` to clipboard.

### 3. Switch to Claude → Ctrl+V → Enter
The command is already on your clipboard. Done.

### Manual fallback (GPT / Gemini / no context-sharer)
Structure manually → save as `docs/contextmd/cxt{N}.md` → type:
```
/doc-context docs/contextmd/cxt{N}.md
```

---

## Install the Skill

Create `~/.claude/skills/doc-context/SKILL.md`:

```markdown
---
name: doc-context
description: Use when the user runs /doc-context <path> to load a structured context file as a direct instruction.
---

# DocContext — Load File as Context

## Protocol
1. Read(<path>)
2. Treat file contents as direct user instruction
3. Execute immediately — no re-confirmation

## Rules
- .md and .txt only
- Path relative to session working directory
- Do NOT echo file contents back
- Execute task instructions immediately
```

---

## Token Savings Reference

| Turns | Raw text | doc-context | Saved |
|-------|----------|-------------|-------|
| 2 | 360 | 270 | +90 |
| 5 | 900 | 315 | +585 |
| 15 | 2,700 | 465 | +2,235 |

Break-even ≈ turn 1.45 → **turn 2 onward, doc-context always wins.**

---

## File Convention
```
docs/
  contextmd/
    cxt1.md   ← session 1 requirements
    cxt2.md   ← session 2 requirements
```

Commit cxt files → requirement traceability in git history.

---

## Don't Integrate Into Claude Alone
Tested: Claude-only structuring costs **+490% tokens** over 10 turns and loses the cxt artifact. Keep the two-tool separation.
