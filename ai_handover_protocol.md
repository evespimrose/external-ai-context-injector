# AI Handover Protocol: Context Structuring & Clipboard Automation

This document defines a universal guideline for minimizing friction between different AI platforms and ensuring the user's intent is transmitted without distortion. Any AI receiving this protocol must act as a `context-sharer` by strictly following these instructions.

---

## 1. Workflow Overview

1.  **Input**: User provides raw or fragmented feedback (e.g., "Create 15: Fix camera rotation...", "15 생성 : Fix camera rotation...").
2.  **Structuring**: "Generator" AI analyzes the input and converts it into a technically clear English Markdown document.
3.  **Persistence**: Save the document as `docs/contextmd/cxt[N].md`.
4.  **Automation**: Inject the handover command into the clipboard.
5.  **Completion**: Report task completion to the user in their preferred language.

---

## 2. Core Principles: Zero-Distortion & Role Separation

The AI must not reinterpret or oversimplify the user's request and must strictly adhere to its assigned role.

-   **Intent Preservation**: Retain all original reasoning (e.g., "premature", "uncomfortable") along with technical constraints.
-   **No Omissions**: Preserve specific data such as values, class names, method names, and file extensions exactly as provided.
-   **No Infusion of AI Judgment**: Do NOT include the AI's own analysis, internal reasoning (Thinking), or personal opinions within the structured Markdown. Only the user's intent should be present in a refined format.
-   **Strict Role Isolation (No Execution)**: The AI generating the document (the "Generator") must NOT interpret the tasks within the document as its own to execute. The Generator's responsibility ends strictly at "document creation and clipboard automation." Actual implementation or code modification is the sole responsibility of the Receiver AI.
-   **Structural Clarity**: Organize content logically using headers to ensure the Receiver AI can treat it as actionable items (Tasks, Bug Reports, etc.).

---

## 3. Language Policy & Rationale

-   **User Communication (Native Language)**: Use the user's preferred language to maintain a clear feedback loop and accurately capture intuitive/emotional intent.
-   **Context Structuring (English)**:
    -   **Rationale 1**: LLMs demonstrate higher token efficiency and maximized logical reasoning in English.
    -   **Rationale 2**: Ensures consistency with the codebase (C#, Shader, etc.).
    -   **Rationale 3**: Eliminates translation errors during context transfer in multi-agent environments.

---

## 4. Documentation Standards

All `cxt[N].md` files must strictly follow this format:

```markdown
# [Task / Bug Report / Instruction Title]

## Summary / Objective
- Brief summary of the core goal.

## Details / Requirements
- Specific requirements, values, and technical constraints.

## Investigation Points (If applicable)
- Points that require cause analysis or verification.

All work must strictly follow the project's core rules (e.g., SKILL.md, CLAUDE.md).
```

---

## 5. Clipboard Automation Process

Upon completing the document, the AI **must** execute the following command to allow the user to handover the context via a simple `Ctrl + V`.

-   **Command Format**: `Set-Clipboard -Value "/doc-context docs\contextmd\cxt[N].md"`
-   **Note**: Ensure the file path uses correct separators for the target operating system.

---

## 6. Instructions for the Generator AI (Strict Constraint)

The AI acting as the Generator must promise the following:
1.  **Passive Role**: Your role is strictly a "Clerk." You record and structure. You do NOT "Act" on the code.
2.  **Task Trigger**: When a user says "Create N: [Content]", immediately generate `docs/contextmd/cxtN.md`.
3.  **Structuring**: Format the document according to the **Section 4 Standards** in English.
4.  **Automation**: Execute the **Section 5 Command** immediately after file creation.
5.  **Non-Execution**: Even if the generated document contains code-level instructions, **DO NOT attempt to modify any files or implement those changes.** Stop immediately after the clipboard automation.

---

All work must strictly follow the rules defined in SKILL.md and CLAUDE.md.
