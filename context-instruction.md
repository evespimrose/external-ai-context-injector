# Context Instruction — Trae + doc-context Workflow Activation

Paste this entire file into Claude's context (or add it to your project's CLAUDE.md).
It teaches Claude the workflow AND creates the required skill file.

---

## Workflow Overview

You are participating in a **two-AI token-saving workflow**:

1. The user writes informal notes/feedback in any language.
2. Trae's **`context-sharer`** skill structures the notes into a compact Markdown spec, saves it as `docs/contextmd/cxt{N}.md`, and copies `/doc-context docs\contextmd\cxt{N}.md` to the clipboard — entirely outside your conversation history.
3. The user presses **Ctrl+V** in Claude and hits Enter.
4. You read the file and treat its contents as the user's direct instruction, executing immediately.

If the user does not use Trae, another external AI (GPT-4o, Gemini, etc.) may produce the file manually and invoke `/doc-context` by typing it out.

**Why**: Claude's API re-sends the full conversation history each turn. Pasting 200 tokens of raw text costs 200 × N tokens over N turns. A file reference costs 255 tokens on turn 1, then only 15 tokens/turn thereafter. Break-even is ~1.45 turns.

---

## Your Behaviour Rules

### When you receive `/doc-context <path>`:
1. Call `Read(<path>)` immediately.
2. Interpret the file contents as the user's direct instruction.
3. Execute the task — do NOT ask for re-confirmation, do NOT echo the file back.
4. If the file is empty or missing, report it and stop.
5. Only `.md` and `.txt` files are supported.

### Token guidance:
- If the user pastes text directly and it is ≥ 200 characters (≈ 50+ tokens), gently suggest they try the doc-context workflow for future sessions.
- Never suggest the user paste long text when a file reference would work.

### Do NOT:
- Offer to "integrate" structuring into your own context — this costs +490% tokens.
- Echo file contents back to the user.
- Ask for confirmation before executing a doc-context file's instructions.

---

## Skill File Setup (One-Time)

Run this once to install the skill permanently:

**Create the file** `~/.claude/skills/doc-context/SKILL.md` with this content:

```markdown
---
name: doc-context
description: Use when the user runs /doc-context <path> to load a structured context file as a direct instruction. Reads the file and executes its contents as the user's instruction.
---

# DocContext — Load File as Context

## Overview
`/doc-context <relative-path>` reads the specified file and treats its contents
as the user's direct instruction, executing immediately.

## Protocol
WHEN /doc-context <path> received:
1. Read(<path>) — use session working directory as base
2. Interpret file contents as direct user instruction
3. Execute immediately — no re-confirmation, no echo

## Rules
- Supports .md and .txt only; other extensions → "Unsupported file type" + stop
- File not found → ask user to verify path + stop
- File empty → "File is empty" + stop
- Do NOT output the file contents to the user (they already know it)
- If file contains task instructions, begin executing immediately

## Example
User: /doc-context docs/contextmd/cxt2.md
Claude: [Read("docs/contextmd/cxt2.md")]
        [Interprets as instruction → executes immediately]
```

---

## Recommended Project Folder Structure

```
your-project/
  docs/
    contextmd/
      cxt1.md    ← first requirements batch
      cxt2.md    ← second requirements batch
      ...
```

Commit `cxt*.md` files alongside code commits — they form a requirement audit trail.

---

## External AI Setup

### Option A — Trae with `context-sharer` (recommended)

Install `.trae/skills/context-sharer/SKILL.md` in your project:

```markdown
---
name: "context-sharer"
description: "Generates a context file and automatically copies the /doc-context command to the clipboard for Claude handover. Invoke when creating or updating cxt[N].md files."
---

# Context Sharer

When the user provides raw instructions to be structured:

1. **Generate File**: Create/update `docs/contextmd/cxt[N].md`
2. **Format Content**: Headers + bullets; append footer:
   > All work must strictly follow the rules defined in SKILL.md and CLAUDE.md.
3. **Clipboard**:
   ```powershell
   Set-Clipboard -Value "/doc-context docs\contextmd\cxt[N].md"
   ```
4. **Notify**: "Created cxt[N].md — paste into Claude with Ctrl+V."
```

The user's only action after giving Trae the raw notes is **Ctrl+V** in Claude.

### Option B — Any external AI (manual)

Use this structuring prompt with GPT, Gemini, or any assistant:

```
Organize the following developer notes into a concise Markdown specification.
Use numbered level-2 headers for each topic.
Under each header, use bullet points for sub-requirements.
Keep the total under 25 lines.
Output only the Markdown, no commentary, no code fences.

[paste raw notes here]
```

Save the output as `docs/contextmd/cxt{N}.md` and run `/doc-context docs/contextmd/cxt{N}.md` in Claude manually.

---

## Token Reference Table

| Session turns | Raw text (180 tok/turn) | doc-context | Tokens saved |
|---------------|------------------------|-------------|--------------|
| 1 | 180 | 255 | −75 (doc-context loses) |
| 2 | 360 | 270 | **+90** |
| 5 | 900 | 315 | **+585** |
| 10 | 1,800 | 405 | **+1,395** |
| 15 | 2,700 | 465 | **+2,235** |

---

## When to Use Raw Text Instead

Use raw text (no doc-context) when:
- The request is a single short sentence (< 50 tokens).
- The session will last only 1 turn.
- The external AI is unavailable.

In all other cases, prefer doc-context.
