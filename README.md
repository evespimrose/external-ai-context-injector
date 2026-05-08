# trae-claude-context-workflow

> **한국어 문서**: [README-ko.md](./README-ko.md)

---

## What is This?

A two-AI collaboration pattern that **dramatically reduces Claude's conversation-token consumption** by offloading requirement structuring to an external AI (Trae, GPT-4o, Gemini, etc.) and feeding the result into Claude via a lightweight file-reference skill.

```
Raw feedback (any language, informal prose)
        │
        ▼
   [External AI — Trae / context-sharer skill]
   1. Structures notes into a compact .md spec
   2. Saves → docs/contextmd/cxt[N].md
   3. Copies "/doc-context docs\contextmd\cxt[N].md" to clipboard
        │   (cost: external AI's own API budget, NOT Claude's)
        ▼
   User: Ctrl+V in Claude → Enter
        │
        ▼
   Claude: Read(cxt[N].md) → interprets as direct instruction → executes
```

The `context-sharer` Trae skill automates steps 1–3 entirely. The user's only action is **Ctrl+V**.

---

## Why It Works — Token Math

Claude's API re-sends the **entire conversation history** on every turn. Raw text pasted into the chat accumulates linearly; a file reference keeps the history footprint tiny.

| Turns | Raw text (180 tok/turn) | doc-context | Saved |
|-------|------------------------|-------------|-------|
| 1 | 180 | 255 | −75 |
| 2 | 360 | 270 | **+90** |
| 5 | 900 | 315 | **+585** |
| 15 | 2,700 | 465 | **+2,235** |

**Break-even: ~1.45 turns.** From turn 2 onward, doc-context always wins.

> **Rule of thumb**: use raw text only for requests under ~50 tokens (e.g. `"rename this function to init"`).  
> Everything else → structure it and use doc-context.

---

## Workflow in 3 Steps

### Step 1 — Feed raw feedback to Trae (with `context-sharer`)

Tell Trae to structure your notes using the `context-sharer` skill:

```
Structure this into cxt5.md and prepare for Claude handover.

[your raw notes here]
```

Trae will:
1. Write `docs/contextmd/cxt5.md` with clean headers and bullets
2. Append the mandatory compliance footer
3. Copy `/doc-context docs\contextmd\cxt5.md` to your clipboard automatically

### Step 2 — Switch to Claude, press Ctrl+V → Enter

That's it. The `/doc-context` command is already on your clipboard.

### Step 3 — Claude executes

Claude reads the file and treats its content as your direct instruction — no copy-paste, no bloated history.

---

### Manual fallback (without `context-sharer`)

If you're using GPT, Gemini, or any other external AI instead of Trae:

```
Organize the following into a concise Markdown spec with numbered headers.
Keep it under 20 lines. Output only the Markdown, no commentary.

[your raw notes here]
```

Save the output as `docs/contextmd/cxt{N}.md` and run `/doc-context docs/contextmd/cxt{N}.md` in Claude manually.

---

## The `context-sharer` Trae Skill

Install this in Trae at `.trae/skills/context-sharer/SKILL.md`:

```markdown
---
name: "context-sharer"
description: "Generates a context file and automatically copies the /doc-context command to the clipboard for Claude handover. Invoke when creating or updating cxt[N].md files."
---

# Context Sharer

When the user provides raw instructions to be structured:

1. **Generate File**: Create/update `docs/contextmd/cxt[N].md`
2. **Format Content**: Use headers + bullets; append the footer:
   > All work must strictly follow the rules defined in SKILL.md and CLAUDE.md.
3. **Clipboard Injection**:
   ```powershell
   Set-Clipboard -Value "/doc-context docs\contextmd\cxt[N].md"
   ```
4. **Notify User**: "Created cxt[N].md — paste into Claude with Ctrl+V."

Core principles:
- **Zero Distortion**: preserve 100% of original intent
- **Zero Friction**: user only presses Ctrl+V in Claude
```

---

## The `doc-context` Claude Skill

Drop this file at `~/.claude/skills/doc-context/SKILL.md`:

```markdown
---
name: doc-context
description: Use when the user runs /doc-context <path> to load a structured context file as a direct instruction.
---

# DocContext — Load File as Context

## Overview
`/doc-context <relative-path>` reads the file and treats its contents as the user's direct instruction.

## Protocol
1. Read(<path>)
2. Interpret contents as direct user instruction
3. Execute immediately — no re-confirmation

## Rules
- Supports .md and .txt only
- Path is relative to the session working directory
- Do NOT echo the file contents back to the user
- If the file contains task instructions, execute them immediately
```

---

## Architecture Decision: Why Keep Two Tools?

Tested integrating both steps into Claude alone (Option A) vs the current 2-tool approach:

| | Current (Trae + doc-context) | Option A (Claude only) |
|-|------------------------------|------------------------|
| Claude token cost (10 turns) | ~390 | ~1,980 |
| Requirement artifact on disk | ✓ cxt*.md | ✗ |
| Structuring quality | External AI specialised | Claude has to parse informal prose |
| **Verdict** | ✅ Keep | ❌ +490% cost |

---

## File Naming & Git Integration

```
docs/
  contextmd/
    cxt1.md    ← first feedback batch
    cxt2.md    ← second batch
    ...
```

Committing `cxt*.md` files gives you **requirement traceability**: you can always `git log` to see what was requested and when, and re-run `/doc-context` against an old file to restore full context in a new session.

---

## When NOT to Use doc-context

- Request is a single short sentence (< 50 tokens)
- You expect the session to last only 1 turn
- The external AI is unavailable — fall back to raw paste, but keep it brief

---

## Quick-Start Checklist

- [ ] Install the `doc-context` skill (see above)
- [ ] Create `docs/contextmd/` in your project
- [ ] Write a structuring prompt template (save it in your external AI)
- [ ] On next feedback session: paste → structure → save → `/doc-context`
- [ ] Commit `cxt*.md` files alongside code changes

---

## Related Documents

- [`CLAUDE.md`](./CLAUDE.md) — paste-ready workflow guide for Claude
- [`context-instruction.md`](./context-instruction.md) — paste into any Claude session to activate the workflow
- [`ai_handover_protocol.md`](./ai_handover_protocol.md) — paste into any external ai session to activate the workflow
- [`Examples.md`](./Examples.md) — real before/after examples
