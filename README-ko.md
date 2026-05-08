# trae-claude-context-workflow

> **English version**: [README.md](./README.md)

---

## 이게 뭔가요?

외부 AI(Trae, GPT-4o, Gemini 등)에 요구사항 구조화를 맡기고, 그 결과물을 파일 참조 방식으로 Claude에 주입하여 **대화 토큰 소모를 최대 85% 줄이는 2-AI 협업 패턴**입니다.

```
원문 피드백 (어떤 언어든, 구어체 OK)
        │
        ▼
   [외부 AI — Trae / context-sharer 스킬]
   1. 노트를 간결한 .md 스펙으로 구조화
   2. docs/contextmd/cxt[N].md 로 저장
   3. "/doc-context docs\contextmd\cxt[N].md" 를 클립보드에 자동 복사
        │   (비용: 외부 AI의 API 예산 — Claude 컨텍스트 외부)
        ▼
   사용자: Claude에서 Ctrl+V → Enter
        │
        ▼
   Claude: cxt[N].md 읽기 → 직접 지시로 해석 → 즉시 실행
```

`context-sharer` Trae 스킬이 1~3단계를 전부 자동화합니다. 사용자의 유일한 액션은 **Ctrl+V**입니다.

---

## 왜 효과가 있나요 — 토큰 수학

Claude API는 매 요청마다 **전체 대화 히스토리를 재전송**합니다. 원문을 채팅에 붙여넣으면 턴마다 누적되지만, 파일 참조 방식은 히스토리 발자국을 최소화합니다.

| 대화 횟수 | 생짜 텍스트 (180 tok/턴) | doc-context | 절감량 |
|-----------|------------------------|-------------|--------|
| 1턴 | 180 | 255 | −75 |
| 2턴 | 360 | 270 | **+90** |
| 5턴 | 900 | 315 | **+585** |
| 15턴 | 2,700 | 465 | **+2,235** |

**손익분기점: 약 1.45턴.** 2턴부터 doc-context가 무조건 유리합니다.

> **운용 기준**: 50토큰 미만의 단건 요청(예: `"이 함수 이름 init으로 바꿔"`)은 생짜 텍스트.  
> 그 외 모든 경우 → 구조화 후 doc-context 사용.

---

## 워크플로 3단계

### Step 1 — Trae에 원문 피드백 전달 (`context-sharer` 사용)

`context-sharer` 스킬로 노트를 구조화하도록 Trae에 지시합니다:

```
아래 내용을 cxt5.md로 정리하고 Claude 핸드오버를 준비해줘.

[원문 노트]
```

Trae가 자동으로:
1. `docs/contextmd/cxt5.md` 작성 (헤더·불릿 구조)
2. 필수 준수 푸터 추가
3. `/doc-context docs\contextmd\cxt5.md` 를 클립보드에 복사

### Step 2 — Claude로 전환, Ctrl+V → Enter

끝입니다. `/doc-context` 명령어가 이미 클립보드에 있습니다.

### Step 3 — Claude 실행

Claude가 파일을 읽고 내용을 직접 지시로 해석해 즉시 실행합니다 — 복붙 없음, 히스토리 비대화 없음.

---

### 수동 폴백 (`context-sharer` 없이)

GPT, Gemini 등 다른 외부 AI를 사용할 경우:

```
아래 개발자 노트를 간결한 Markdown 명세로 정리해줘.
번호가 붙은 2단계 헤더를 사용하고, 각 헤더 아래에 불릿으로 세부 요구사항을 작성해.
전체 25줄 이내로 유지해.
Markdown만 출력하고, 설명이나 코드 펜스는 넣지 마.

[원문 노트]
```

결과물을 `docs/contextmd/cxt{N}.md`로 저장하고 Claude에서 `/doc-context docs/contextmd/cxt{N}.md`를 직접 입력합니다.

---

## `context-sharer` Trae 스킬 설치

`.trae/skills/context-sharer/SKILL.md`에 설치합니다:

```markdown
---
name: "context-sharer"
description: "Generates a context file and automatically copies the /doc-context command to the clipboard for Claude handover. Invoke when creating or updating cxt[N].md files."
---

# Context Sharer

사용자가 구조화할 원문 지시를 제공하면:

1. **파일 생성**: `docs/contextmd/cxt[N].md` 생성/업데이트
2. **내용 포맷**: 헤더 + 불릿 구조로 작성; 아래 푸터 추가:
   > All work must strictly follow the rules defined in SKILL.md and CLAUDE.md.
3. **클립보드 주입**:
   ```powershell
   Set-Clipboard -Value "/doc-context docs\contextmd\cxt[N].md"
   ```
4. **사용자 알림**: "cxt[N].md를 생성했습니다. Claude에서 Ctrl+V로 붙여넣으세요."

핵심 원칙:
- **Zero Distortion**: 원문 의도 100% 보존
- **Zero Friction**: 사용자는 Claude에서 Ctrl+V만 누름
```

---

## `doc-context` Claude 스킬 설치

`~/.claude/skills/doc-context/SKILL.md`에 설치합니다:

```markdown
---
name: doc-context
description: Use when the user runs /doc-context <path> to load a structured context file as a direct instruction.
---

# DocContext — Load File as Context

## Overview
`/doc-context <relative-path>` 호출 시 지정 파일을 읽어 사용자의 직접 지시로 처리합니다.

## Protocol
1. Read(<path>)
2. 파일 내용을 사용자의 직접 지시로 해석
3. 즉시 실행 — 재확인 없음

## Rules
- .md / .txt 파일만 지원
- 경로는 세션 작업 디렉터리 기준 상대 경로
- 파일 내용을 사용자에게 다시 출력하지 않음
- 작업 지시가 있으면 즉시 수행
```

---

## 아키텍처 결정: 왜 두 도구를 유지하나요?

Claude 단독으로 구조화+실행을 통합하는 방안(Option A)과 현행 2-도구 방식을 비교 검증했습니다:

| | 현행 (외부 AI + doc-context) | Option A (Claude 단독) |
|-|------------------------------|------------------------|
| Claude 토큰 비용 (10턴) | ~390 | ~1,980 |
| 디스크 요구사항 아티팩트 | ✓ cxt*.md | ✗ |
| 구조화 품질 | 외부 AI 전담 | Claude가 구어체 직접 파싱 |
| **결론** | ✅ 유지 | ❌ +490% 비용 |

---

## 파일 명명 & Git 통합

```
docs/
  contextmd/
    cxt1.md    ← 첫 번째 피드백 배치
    cxt2.md    ← 두 번째 배치
    ...
```

`cxt*.md` 파일을 코드 커밋과 함께 커밋하면 **요구사항 추적**이 가능합니다. `git log`로 언제 무엇이 요청됐는지 확인하고, 새 세션에서 `/doc-context`를 재실행해 컨텍스트를 복원할 수 있습니다.

---

## doc-context를 쓰지 않아도 되는 경우

- 요청이 짧은 한 문장 (50토큰 미만)
- 세션이 1턴으로 끝날 것으로 예상될 때
- 외부 AI를 사용할 수 없을 때 — 이 경우 원문 붙여넣기로 폴백, 최대한 짧게 유지

---

## 빠른 시작 체크리스트

- [ ] `doc-context` Claude 스킬 설치 (위 참조)
- [ ] `context-sharer` Trae 스킬 설치 (위 참조)
- [ ] 프로젝트에 `docs/contextmd/` 디렉터리 생성
- [ ] 다음 피드백 세션부터: Trae에 전달 → Ctrl+V → Claude 실행
- [ ] `cxt*.md` 파일을 코드 변경사항과 함께 커밋

---

## 관련 문서

- [`CLAUDE.md`](./CLAUDE.md) — Claude에 붙여넣기만 하면 되는 워크플로 가이드
- [`context-instruction.md`](./context-instruction.md) — Claude 컨텍스트에 붙여넣으면 워크플로 즉시 활성화
- [`ai_handover_protocol.md`](./ai_handover_protocol.md) — 외부 AI 컨텍스트에 붙여넣으면 워크플로 즉시 활성화
- [`Examples.md`](./Examples.md) — 실제 before/after 예시
