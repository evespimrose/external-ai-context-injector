# Examples — Trae + doc-context Workflow

Before/after comparisons showing how raw feedback becomes a structured context file.

In each example, Trae's **`context-sharer`** skill handles structuring + file creation + clipboard injection automatically. The user only presses **Ctrl+V** in Claude.

---

## Example 1 — UI/UX Refinement (multi-requirement)

### Raw feedback (before structuring)
```
플레이 테스트 해봤는데 전반적으로 잘 돌아가긴 하는데 세부 조정 필요함.
1. 편집 모드 진입할 때 카메라가 튀지 않게 해줘. 그리고 그리드 크기에 맞게
카메라 사이즈 54로 맞춰줘. 패닝 속도가 너무 느려서 마우스 여러 번 끌어야 함.
그리드 가로세로 크기에 맞게 blend해줬으면 좋겠음.
2. 편집 모드 바닥 그리드 색상 흰색에 알파 1.0으로 바꿔줘.
```
**~300 chars / ~200 tokens** — if pasted raw, this repeats in every turn.

### After structuring (saved as `cxt3.md`, ~130 tokens)
```markdown
# Task: Editor Mode UX & Visual Refinement

## 1. Camera Behavior
- On entering edit mode, camera must NOT jump/transition
- Set orthographic size to 54 (matches current grid dimensions)
- Pan speed: blend multiplier based on grid W/D dimensions
  - Goal: single mouse drag covers the full grid width

## 2. Floor Grid Visuals
- Color: White (1, 1, 1), Alpha: 1.0
```

### Invocation
```
/doc-context docs/contextmd/cxt3.md
```

### Token savings over 10 turns
| Method | 10-turn cost |
|--------|-------------|
| Raw paste | ~2,000 tok |
| doc-context | ~385 tok |
| **Saved** | **~1,615 tok** |

---

## Example 2 — Bug Report

### Raw feedback
```
NullRef 터짐. 신규 생성 플로우에서 객체가 null인 상태로 파일 경로 접근하려고 함.
재현: 저장된 파일 없이 새 항목 생성하면 바로 크래시.
```

### Structured cxt file
```markdown
# Task: NullReferenceException Fix

## Bug
- Location: ItemController.SaveItem(ItemData data)
- Cause: data.FilePath accessed when data == null (new item creation flow, no prior save)
- Repro: create new item without saving → immediate crash

## Fix Required
- Add null guard before FilePath access
- if (data != null && !string.IsNullOrEmpty(data.FilePath))
```

### Result
Single-turn fix. Even for a 1-turn session, the structured file **documents the bug in git history** when committed alongside the fix — raw paste does not.

---

## Example 3 — Analysis / Architecture Request

### Raw feedback
```
doc-context 쓰는 게 실제로 토큰 절감이 되는 건지 수치로 확인해줘.
그리고 외부 AI 구조화 단계를 클로드 안으로 통합하는 게 더 효율적인지도 검토해줘.
```

### Structured cxt file
```markdown
# Task: doc-context Workflow Analysis

## Objective 1 — Token Efficiency Measurement
- Compare: raw text input vs doc-context file reference
- Measure cumulative token cost over N turns
- Provide break-even point

## Objective 2 — Integration Feasibility
- Evaluate: merging external-AI structuring into Claude's context
- Compare token cost, artifact retention, structuring quality
- Deliver evidence-based recommendation
```

### Result
Claude produced a full quantitative token analysis and a `/try`-style feasibility report.  
**Conclusion**: Keep 2-tool separation — integration costs +490% tokens and loses the cxt artifact on disk.

---

## Example 4 — Multi-topic Batch (3 fixes in one file)

### Structured cxt file
```markdown
# Task: Renderer & State Cleanup

## 1. Renderer Not Initialised
- Renderer.Initialize() never called on mode entry → GPU buffers null → nothing visible
- Fix: call Cleanup() → Initialize(w, h, d, root) on each mode entry

## 2. Default Color Reset
- After Initialize(), paint all cells with DefaultColor before MarkAllDirty()

## 3. Exit Cleanup
- Call renderer?.Cleanup() on mode exit so renderer stops drawing after leaving the mode
```

**Key benefit**: Three related fixes described in one file. Claude executes all three in one session, and the file serves as a self-contained commit message artifact.

---

## Pattern Summary

| Use Case | Approach |
|----------|----------|
| Single-line change (`"rename X to Y"`) | Raw text — 1 turn, < 50 tok |
| Bug report with context | doc-context — file stays in git |
| UX refinement with multiple topics | doc-context — bundle into one cxt file |
| Architecture / feasibility analysis | doc-context + `/try` skill |
| Multi-session requirement | doc-context — re-use same cxt file across sessions |

---

## Anti-Patterns

### Don't: Paste the full requirement in chat
```
User: [pastes 400 characters of informal notes directly]
```
Over 10 turns this costs ~2,600 extra tokens vs doc-context.

### Don't: Ask Claude to structure your notes
```
User: "Can you organize these notes and then implement them?"
```
Claude structuring + implementing in the same context costs +490% more tokens than the 2-tool approach.

### Do: Use Trae (`context-sharer`) for structuring, Claude for execution
```
User → Trae: "Structure this into cxt1.md and prepare for Claude handover."
Trae: [structures → saves cxt1.md → copies command to clipboard]
User → Claude: Ctrl+V → Enter
Claude: [executes immediately]
```
