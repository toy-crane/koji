# PRP 워크플로우 멘탈 모델

> **핵심 공식**: `PRP = PRD + 코드베이스 지식 + 검증 루프`
>
> AI가 **첫 번째 시도에서 프로덕션 레디 코드**를 배포할 수 있도록 설계된 프레임워크

---

## 1. 핵심 철학

### Context is King

AI가 성공하려면 **모든 필요한 정보**가 프롬프트에 있어야 한다.

```
일반적인 프롬프트:
"로그인 기능 구현해줘"
→ AI가 추측해야 함 → 실패 가능성 높음

PRP 방식:
"로그인 기능 구현해줘
 - 기존 auth 패턴: src/auth/service.ts:10-50
 - 에러 처리 방식: CustomError 클래스 사용
 - 테스트 패턴: describe/it 블록
 - 검증 명령어: bun run type-check && bun test"
→ AI가 정확히 무엇을 어떻게 해야 하는지 앎 → 성공
```

### Validation First

모든 변경은 **즉시 검증**된다. 실패하면 고치고, 통과해야 다음으로 진행.

```
코드 작성 → 검증 → 실패 → 수정 → 검증 → 통과 → 다음 단계
              ↑_________|
```

### Pattern Mirroring

새로운 패턴을 발명하지 않는다. **기존 코드베이스 스타일을 복사**한다.

```
잘못된 방식: "일반적인 best practice로 구현"
올바른 방식: "src/features/users/service.ts:10-30의 패턴을 그대로 따라서 구현"
```

### Evidence-Based Decisions

추측하지 않는다. **증거**를 기반으로 결정한다.

```
잘못된 방식: "아마 이 패턴을 사용하는 것 같다"
올바른 방식: "`src/features/projects/service.ts:45`에서 확인한 패턴을 사용"
```

---

## 2. 전체 워크플로우 개요

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           사용자 요청                                    │
│                     "새로운 기능을 구현해줘"                              │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  PHASE 1: 문서화 (PRD)                                                   │
│  ─────────────────────                                                  │
│  "무엇을 왜 만드는가?"                                                   │
│                                                                         │
│  • 문제 정의 + 증거 (Evidence)                                          │
│  • 사용자 정의 + Job to Be Done                                         │
│  • 핵심 가설 (Key Hypothesis)                                           │
│  • 성공 지표 + 제약사항                                                  │
│  • 구현 단계 분할 (Phase 1, 2, 3...)                                     │
│  • MoSCoW 우선순위 + Out of Scope                                       │
│  • 기술 리스크 + 결정 로그                                               │
│                                                                         │
│  OUTPUT: .claude/PRPs/prds/feature.prd.md                               │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  PHASE 2: 계획 (Plan)                                                    │
│  ─────────────────────                                                  │
│  "어떻게 만드는가?"                                                      │
│                                                                         │
│  • 코드베이스 탐색 → 기존 패턴 발견 (Explore Agent)                      │
│  • 외부 문서 조사 → 라이브러리 사용법 (WebSearch)                        │
│  • UX 설계 → Before/After 다이어그램                                    │
│  • 필수 읽기 파일 목록 (Mandatory Reading)                               │
│  • 구체적인 Task 목록 생성                                               │
│  • 테스트 전략 + 엣지 케이스 체크리스트                                  │
│  • 6단계 검증 명령어 정의                                                │
│  • 수용 기준 (Acceptance Criteria)                                       │
│  • 리스크 및 완화 전략                                                   │
│                                                                         │
│  OUTPUT: .claude/PRPs/plans/feature.plan.md                             │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  PHASE 3: 구현 (Implement)                                               │
│  ─────────────────────────                                              │
│  "실제로 만든다"                                                         │
│                                                                         │
│  • 프로젝트 환경 감지 (DETECT) - 패키지 매니저 자동 인식                 │
│  • Git 상태 확인 및 브랜치 관리 (PREPARE)                                │
│  • Plan의 Task를 순서대로 실행                                           │
│  • 매 변경 후 검증 (type-check)                                          │
│  • 편차 발생 시 문서화 (Deviation Handling)                              │
│  • 전체 완료 후 전체 검증 (lint, test, build)                            │
│  • PRD 자동 업데이트 (Phase 완료 시)                                     │
│  • Plan 아카이브                                                         │
│                                                                         │
│  OUTPUT: 작동하는 코드 + 구현 보고서 (Assessment vs Reality)             │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  PHASE 4: 완료 (Commit & PR)                                             │
│  ──────────────────────────                                             │
│  "배포 준비"                                                             │
│                                                                         │
│  • Git 커밋 생성 (Conventional Commits)                                  │
│  • PR 템플릿 확인 및 적용                                                │
│  • Pull Request 생성                                                    │
│  • 이슈 링크 연결                                                        │
│                                                                         │
│  OUTPUT: GitHub Pull Request                                            │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  PHASE 5: 코드 리뷰 (Review)                                             │
│  ─────────────────────────                                              │
│  "품질 검증"                                                             │
│                                                                         │
│  • PR 컨텍스트 수집 (메타데이터, diff)                                   │
│  • 구현 아티팩트 확인 (implementation report)                            │
│  • 코드 리뷰 체크리스트 실행                                             │
│  • 자동 검증 실행                                                        │
│  • 이슈 심각도 분류 (Critical/High/Medium/Low)                           │
│  • 리뷰 결과 GitHub에 게시                                               │
│                                                                         │
│  OUTPUT: PR 코멘트 + 리뷰 리포트                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. 각 Phase 상세

### Phase 1: PRD (문서화)

**목적**: "무엇을 왜 만드는가"를 명확히 정의

```
INPUT:  사용자의 막연한 아이디어
        "사용자 인증 시스템 만들어줘"

PROCESS:
        1. 질문을 통해 요구사항 구체화 (FOUNDATION)
           - 누가 사용하나? (Who)
           - 어떤 문제를 해결하나? (What)
           - 왜 지금 해야 하나? (Why now)
           - 성공은 어떻게 측정하나? (How to measure)

        2. 시장/기술 조사 (GROUNDING)
           - 경쟁 제품은 어떻게 하나? (WebSearch)
           - 기존 코드베이스에 유사한 기능이 있나? (Explore Agent)
           - 기술적으로 가능한가?

        3. 깊은 분석 (DEEP DIVE)
           - Vision: 성공 시 이상적인 최종 상태
           - Primary User: 가장 중요한 사용자 정의
           - Job to Be Done: "When X, I want to Y, so that Z"
           - Non-Users: 명시적으로 타겟이 아닌 사용자
           - Constraints: 기술/시간/비용 제약

        4. 스코프 결정 (DECISIONS)
           - MVP 정의: 최소 구현 범위
           - Must Have vs Nice to Have
           - Key Hypothesis: 테스트 가능한 가설
           - Out of Scope: 명시적으로 안 만드는 것

        5. 구현 단계 분할
           - Phase 1: 기본 로그인
           - Phase 2: JWT 토큰
           - Phase 3: 2FA

OUTPUT: PRD 문서 (.prd.md)
```

**핵심 산출물 - PRD 템플릿 구조**:

```markdown
# {Feature Name}

## Problem Statement
{2-3문장: 누가 무슨 문제가 있고, 해결하지 않으면 어떤 비용이 발생하는가?}

## Evidence (증거)
- {문제가 존재한다는 증거: 사용자 인용, 데이터, 관찰}
- {증거가 없으면: "Assumption - needs validation through [method]"}

## Proposed Solution
{한 단락: 무엇을 만들고, 왜 다른 대안보다 이 방식인가}

## Key Hypothesis (핵심 가설)
We believe {capability} will {solve problem} for {users}.
We'll know we're right when {measurable outcome}.

## What We're NOT Building (안 만드는 것)
- {Item 1} - {why}
- {Item 2} - {why}

## Success Metrics (성공 지표)
| Metric | Target | How Measured |
|--------|--------|--------------|
| {Primary metric} | {Specific number} | {Method} |

## Open Questions (열린 질문)
- [ ] {Unresolved question 1}
- [ ] {Unresolved question 2}

---

## Users & Context

**Primary User**
- **Who**: {구체적인 설명}
- **Current behavior**: {현재 하는 것}
- **Trigger**: {필요가 발생하는 순간}
- **Success state**: {"완료"가 어떤 모습인가}

**Job to Be Done**
When {situation}, I want to {motivation}, so I can {outcome}.

**Non-Users**
{이것이 누구를 위한 것이 아닌지, 그리고 이유}

---

## Solution Detail

### Core Capabilities (MoSCoW 우선순위)
| Priority | Capability | Rationale |
|----------|------------|-----------|
| Must     | {Feature}  | {Why essential} |
| Should   | {Feature}  | {Why important but not blocking} |
| Could    | {Feature}  | {Nice to have} |
| Won't    | {Feature}  | {Explicitly deferred and why} |

### MVP Scope
{가설을 검증하기 위한 최소한}

### User Flow
{Critical path - 가치까지 가장 짧은 여정}

---

## Technical Approach

**Feasibility**: {HIGH/MEDIUM/LOW}

**Architecture Notes**
- {핵심 기술 결정과 이유}
- {의존성 또는 통합 지점}

**Technical Risks (기술 리스크)**
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| {Risk} | H/M/L | H/M/L | {How to handle} |

---

## Implementation Phases (구현 단계)

<!--
  STATUS: pending | in-progress | complete
  PARALLEL: 동시에 실행 가능한 phase (e.g., "with 3" or "-")
  DEPENDS: 먼저 완료해야 하는 phase (e.g., "1, 2" or "-")
  PRP: 생성된 plan 파일 링크
-->

| # | Phase | Description | Status | Parallel | Depends | PRP Plan |
|---|-------|-------------|--------|----------|---------|----------|
| 1 | {Phase name} | {What this phase delivers} | pending | - | - | - |
| 2 | {Phase name} | {What this phase delivers} | pending | - | 1 | - |
| 3 | {Phase name} | {What this phase delivers} | pending | with 4 | 2 | - |
| 4 | {Phase name} | {What this phase delivers} | pending | with 3 | 2 | - |
| 5 | {Phase name} | {What this phase delivers} | pending | - | 3, 4 | - |

### Phase Details

**Phase 1: {Name}**
- **Goal**: {달성하려는 것}
- **Scope**: {범위가 정해진 산출물}
- **Success signal**: {완료를 어떻게 아는가}

### Parallelism Notes (병렬성 노트)
{어떤 phase가 병렬로 실행 가능하고 왜 그런지 설명}

---

## Decisions Log (결정 로그)

| Decision | Choice | Alternatives | Rationale |
|----------|--------|--------------|-----------|
| {Decision} | {Choice} | {Options considered} | {Why this one} |

---

## Research Summary (조사 요약)

**Market Context**
{시장 조사에서 발견한 핵심 내용}

**Technical Context**
{기술 탐색에서 발견한 핵심 내용}
```

**PRD → Plan 연결**:
- PRD에서 `pending` 상태인 Phase 중 의존성이 `complete`인 것을 자동 선택
- Plan 생성 시 PRD의 해당 Phase Status를 `in-progress`로 업데이트
- Plan 완료 시 `complete`로 업데이트

---

### Phase 2: Plan (계획)

**목적**: "어떻게 만드는가"를 구체적으로 정의

```
INPUT:  PRD 파일 또는 기능 설명
        ".claude/PRPs/prds/auth.prd.md" (→ 자동으로 다음 Phase 선택)
        또는 "페이지네이션 추가해줘"

PROCESS:
        0. 입력 타입 감지 (DETECT)
           - .prd.md → PRD에서 다음 pending phase 선택
           - .md (Implementation Phases 포함) → PRD로 처리
           - 자유 텍스트 → 기능 설명으로 직접 사용

        1. 코드베이스 탐색 (Explore Agent 사용)
           - 유사한 기능이 이미 있나?
           - 어떤 패턴을 사용하나?
           - 어떤 디렉토리 구조인가?
           - 실제 코드 스니펫 수집 (발명 금지!)

        2. 외부 문서 조사 (WebSearch 사용)
           - 라이브러리 공식 문서 (버전 매칭!)
           - 알려진 gotchas
           - 보안 고려사항
           - 성능 최적화 패턴

        3. UX 설계 (DESIGN)
           - Before/After ASCII 다이어그램
           - 사용자 흐름 변화 문서화
           - 데이터 흐름 시각화

        4. 아키텍처 설계 (ARCHITECT)
           - 실행 순서 결정
           - 실패 모드 분석
           - 성능/보안 검토
           - Scope 경계 명확화

        5. Task 목록 생성
           - 각 Task는 원자적 (atomic)
           - 의존성 순서대로 정렬
           - 각 Task에 MIRROR 참조 + 검증 명령어 포함

OUTPUT: Plan 문서 (.plan.md)
```

**핵심 산출물 - Plan 템플릿 구조**:

```markdown
# Feature: {Feature Name}

## Summary
{한 단락: 무엇을 만들고 고수준 접근 방식}

## User Story
As a {user type}
I want to {action}
So that {benefit}

## Problem Statement
{구체적인 문제 - 테스트 가능해야 함}

## Solution Statement
{어떻게 해결하는가 - 아키텍처 개요}

## Metadata
| Field | Value |
|-------|-------|
| Type | NEW_CAPABILITY / ENHANCEMENT / REFACTOR / BUG_FIX |
| Complexity | LOW / MEDIUM / HIGH |
| Systems Affected | {comma-separated list} |
| Dependencies | {external libs/services with versions} |
| Estimated Tasks | {count} |

---

## UX Design (UX 설계)

### Before State (현재 상태)
```
╔═══════════════════════════════════════════════════════════════╗
║                         BEFORE STATE                           ║
╠═══════════════════════════════════════════════════════════════╣
║   ┌─────────────┐         ┌─────────────┐         ┌─────────┐ ║
║   │   Screen/   │ ──────► │   Action    │ ──────► │ Result  │ ║
║   │  Component  │         │   Current   │         │ Current │ ║
║   └─────────────┘         └─────────────┘         └─────────┘ ║
║                                                               ║
║   USER_FLOW: [현재 단계별 경험 설명]                           ║
║   PAIN_POINT: [무엇이 빠졌거나, 고장났거나, 비효율적인가]        ║
║   DATA_FLOW: [현재 시스템에서 데이터가 어떻게 흐르는가]          ║
╚═══════════════════════════════════════════════════════════════╝
```

### After State (변경 후 상태)
```
╔═══════════════════════════════════════════════════════════════╗
║                          AFTER STATE                           ║
╠═══════════════════════════════════════════════════════════════╣
║   ┌─────────────┐         ┌─────────────┐         ┌─────────┐ ║
║   │   Screen/   │ ──────► │   Action    │ ──────► │ Result  │ ║
║   │  Component  │         │    NEW      │         │   NEW   │ ║
║   └─────────────┘         └─────────────┘         └─────────┘ ║
║                                  │                            ║
║                                  ▼                            ║
║                          ┌─────────────┐                      ║
║                          │ NEW_FEATURE │ ◄── [추가된 기능]     ║
║                          └─────────────┘                      ║
║                                                               ║
║   USER_FLOW: [새로운 단계별 경험 설명]                          ║
║   VALUE_ADD: [이 변경으로 사용자가 얻는 것]                      ║
║   DATA_FLOW: [변경 후 데이터가 어떻게 흐르는가]                  ║
╚═══════════════════════════════════════════════════════════════╝
```

### Interaction Changes (상호작용 변화)
| Location | Before | After | User Impact |
|----------|--------|-------|-------------|
| {path/component} | {old behavior} | {new behavior} | {사용자에게 무엇이 변하는가} |

---

## Mandatory Reading (필수 읽기)

**CRITICAL: 구현 에이전트는 어떤 Task도 시작하기 전에 반드시 이 파일들을 읽어야 함:**

| Priority | File | Lines | Why Read This |
|----------|------|-------|---------------|
| P0 | `path/to/critical.ts` | 10-50 | 정확히 MIRROR해야 할 패턴 |
| P1 | `path/to/types.ts` | 1-30 | IMPORT해야 할 타입 |
| P2 | `path/to/test.ts` | all | FOLLOW해야 할 테스트 패턴 |

**External Documentation (외부 문서):**
| Source | Section | Why Needed |
|--------|---------|------------|
| [Lib Docs v{version}](url#anchor) | {section name} | {specific reason} |

---

## Patterns to Mirror (미러링할 패턴)

**NAMING_CONVENTION:**
```typescript
// SOURCE: src/features/example/service.ts:10-15
// 이 패턴을 복사:
{코드베이스에서 가져온 실제 코드 스니펫}
```

**ERROR_HANDLING:**
```typescript
// SOURCE: src/features/example/errors.ts:5-20
// 이 패턴을 복사:
{코드베이스에서 가져온 실제 코드 스니펫}
```

**LOGGING_PATTERN:**
```typescript
// SOURCE: src/features/example/service.ts:25-30
// 이 패턴을 복사:
{코드베이스에서 가져온 실제 코드 스니펫}
```

**REPOSITORY_PATTERN:**
```typescript
// SOURCE: src/features/example/repository.ts:10-40
// 이 패턴을 복사:
{코드베이스에서 가져온 실제 코드 스니펫}
```

**TEST_STRUCTURE:**
```typescript
// SOURCE: src/features/example/tests/service.test.ts:1-25
// 이 패턴을 복사:
{코드베이스에서 가져온 실제 코드 스니펫}
```

---

## Files to Change (변경할 파일)

| File | Action | Justification |
|------|--------|---------------|
| `src/features/new/models.ts` | CREATE | 타입 정의 - 스키마에서 재내보내기 |
| `src/features/new/schemas.ts` | CREATE | Zod 검증 스키마 |
| `src/features/new/errors.ts` | CREATE | 기능별 에러 |
| `src/features/new/repository.ts` | CREATE | 데이터베이스 작업 |
| `src/features/new/service.ts` | CREATE | 비즈니스 로직 |
| `src/core/database/schema.ts` | UPDATE | 테이블 정의 추가 |

---

## NOT Building (안 만드는 것 - Scope 제한)

스코프 확장을 방지하기 위한 명시적 제외:

- {Item 1 - 명시적으로 범위 밖이며 이유}
- {Item 2 - 명시적으로 범위 밖이며 이유}

---

## Step-by-Step Tasks (단계별 Task)

순서대로 실행. 각 Task는 원자적이고 독립적으로 검증 가능.

### Task 1: CREATE `src/core/database/schema.ts` (update)

- **ACTION**: 스키마에 테이블 정의 추가
- **IMPLEMENT**: {구체적인 컬럼, 타입, 제약조건}
- **MIRROR**: `src/core/database/schema.ts:XX-YY` - 기존 테이블 패턴 따라서
- **IMPORTS**: `import { pgTable, text, timestamp } from "drizzle-orm/pg-core"`
- **GOTCHA**: {피해야 할 알려진 이슈, e.g., "id는 serial이 아닌 uuid 사용"}
- **VALIDATE**: `npx tsc --noEmit` - 타입이 컴파일되어야 함

### Task 2: CREATE `src/features/new/models.ts`

- **ACTION**: 타입 정의 파일 생성
- **IMPLEMENT**: 테이블 재내보내기, 추론된 타입 정의
- **MIRROR**: `src/features/projects/models.ts:1-10`
- **IMPORTS**: `import { things } from "@/core/database/schema"`
- **TYPES**: `type Thing = typeof things.$inferSelect`
- **GOTCHA**: 읽기 타입은 `$inferSelect`, 쓰기는 `$inferInsert` 사용
- **VALIDATE**: `npx tsc --noEmit`

{... 더 많은 Task들 ...}

---

## Testing Strategy (테스트 전략)

### Unit Tests to Write (작성할 유닛 테스트)

| Test File | Test Cases | Validates |
|-----------|------------|-----------|
| `src/features/new/tests/schemas.test.ts` | 유효한 입력, 유효하지 않은 입력 | Zod 스키마 |
| `src/features/new/tests/errors.test.ts` | 에러 속성 | 에러 클래스 |
| `src/features/new/tests/service.test.ts` | CRUD 작업, 접근 제어 | 비즈니스 로직 |

### Edge Cases Checklist (엣지 케이스 체크리스트)

- [ ] 빈 문자열 입력
- [ ] 필수 필드 누락
- [ ] 권한 없는 접근 시도
- [ ] Not found 시나리오
- [ ] 중복 생성 시도
- [ ] {기능별 엣지 케이스}

---

## Validation Commands (검증 명령어)

**IMPORTANT**: 이 플레이스홀더를 프로젝트의 package.json/config의 실제 명령어로 교체.

### Level 1: STATIC_ANALYSIS (정적 분석)
```bash
{runner} run lint && {runner} run type-check
# Examples: npm run lint, pnpm lint, ruff check . && mypy ., cargo clippy
```
**EXPECT**: Exit 0, 에러나 경고 없음

### Level 2: UNIT_TESTS (유닛 테스트)
```bash
{runner} test {path/to/feature/tests}
# Examples: npm test, pytest tests/, cargo test, go test ./...
```
**EXPECT**: 모든 테스트 통과, 커버리지 >= 80%

### Level 3: FULL_SUITE (전체 스위트)
```bash
{runner} test && {runner} run build
# Examples: npm test && npm run build, cargo test && cargo build
```
**EXPECT**: 모든 테스트 통과, 빌드 성공

### Level 4: DATABASE_VALIDATION (데이터베이스 검증 - 스키마 변경 시)
Supabase MCP 등으로 확인:
- [ ] 올바른 컬럼으로 테이블 생성됨
- [ ] RLS 정책 적용됨
- [ ] 인덱스 생성됨

### Level 5: BROWSER_VALIDATION (브라우저 검증 - UI 변경 시)
Browser MCP로 확인:
- [ ] UI가 올바르게 렌더링됨
- [ ] 사용자 플로우가 엔드투엔드로 작동함
- [ ] 에러 상태가 적절히 표시됨

### Level 6: MANUAL_VALIDATION (수동 검증)
{이 기능에 특화된 단계별 수동 테스트}

---

## Acceptance Criteria (수용 기준)

- [ ] 모든 지정된 기능이 사용자 스토리대로 구현됨
- [ ] Level 1-3 검증 명령어가 exit 0으로 통과
- [ ] 유닛 테스트가 새 코드의 >= 80% 커버
- [ ] 코드가 기존 패턴을 정확히 미러링 (네이밍, 구조, 로깅)
- [ ] 기존 테스트에 회귀 없음
- [ ] UX가 "After State" 다이어그램과 일치

---

## Completion Checklist (완료 체크리스트)

- [ ] 모든 Task가 의존성 순서대로 완료됨
- [ ] 각 Task가 완료 직후 검증됨
- [ ] Level 1: 정적 분석 (lint + type-check) 통과
- [ ] Level 2: 유닛 테스트 통과
- [ ] Level 3: 전체 테스트 스위트 + 빌드 성공
- [ ] Level 4: 데이터베이스 검증 통과 (해당 시)
- [ ] Level 5: 브라우저 검증 통과 (해당 시)
- [ ] 모든 수용 기준 충족

---

## Risks and Mitigations (리스크와 완화)

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| {Risk description} | LOW/MED/HIGH | LOW/MED/HIGH | {Specific prevention/handling strategy} |

---

## Notes

{추가 컨텍스트, 설계 결정, 트레이드오프, 향후 고려사항}
```

---

### Phase 3: Implement (구현)

**목적**: Plan을 실제 코드로 변환

```
INPUT:  Plan 파일
        ".claude/PRPs/plans/auth.plan.md"

PROCESS:
        0. 프로젝트 환경 감지 (DETECT)
           - 패키지 매니저 식별 (bun/pnpm/yarn/npm/cargo/go 등)
           - 검증 스크립트 확인 (type-check, lint, test, build)

        1. Git 상태 준비 (PREPARE)
           ┌─ WORKTREE에 있음?
           │  └─ YES → 그대로 사용 (로그: "Using worktree")
           │
           ├─ MAIN/MASTER에 있음?
           │  └─ Q: 워킹 디렉토리 깨끗함?
           │     ├─ YES → 브랜치 생성: feature/{plan-slug}
           │     └─ NO  → 경고: "먼저 커밋하거나 stash하세요"
           │              STOP
           │
           └─ FEATURE 브랜치에 있음?
              └─ 그대로 사용

        2. Plan 읽기
           - Task 목록 파악
           - MIRROR 참조 파악
           - 검증 명령어 파악

        3. 각 Task 실행
           ┌─────────────────────────────────────┐
           │  Task N 실행                        │
           │                                     │
           │  1. MIRROR 파일 읽기                │
           │  2. 패턴 따라 구현                  │
           │  3. type-check 실행                │
           │     └─ FAIL → 수정 → 다시 검증     │
           │     └─ PASS → 다음 Task            │
           └─────────────────────────────────────┘

        4. 편차 처리 (Deviation Handling)
           - Plan에서 벗어나야 할 경우:
             - 무엇이 변경되었는지 기록
             - 왜 변경되었는지 기록
             - 편차 문서화와 함께 계속 진행

        5. 전체 검증
           type-check → lint → tests → build
           모두 통과해야 완료

        6. PRD 업데이트 (해당 시)
           - PRD에서 생성된 Plan인 경우
           - 해당 Phase의 Status를 `complete`로 변경

        7. Plan 아카이브
           - 완료된 Plan을 `completed/` 폴더로 이동

OUTPUT: 작동하는 코드 + 구현 보고서 (.report.md)
```

**핵심 산출물 - Implementation Report 구조**:

```markdown
# Implementation Report

**Plan**: `{plan_path}`
**Source Issue**: #{number} (해당 시)
**Branch**: `{branch-name}`
**Date**: {YYYY-MM-DD}
**Status**: {COMPLETE | PARTIAL}

---

## Summary

{구현된 내용의 간단한 설명}

---

## Assessment vs Reality (예측 vs 실제)

원래 조사의 평가와 실제 발생한 것을 비교:

| Metric | Predicted | Actual | Reasoning |
|--------|-----------|--------|-----------|
| Complexity | {from plan} | {actual} | {왜 일치하거나 달랐는지} |
| Confidence | {from plan} | {actual} | {e.g., "근본 원인이 맞았다" 또는 "X 때문에 피봇해야 했다"} |

**구현이 plan에서 벗어났다면, 이유 설명:**

- {무엇이 변경되었고 왜 - 구현 중 발견한 내용 기반}

---

## Tasks Completed

| # | Task | File | Status |
|---|------|------|--------|
| 1 | {task description} | `src/x.ts` | ✅ |
| 2 | {task description} | `src/y.ts` | ✅ |

---

## Validation Results

| Check | Result | Details |
|-------|--------|---------|
| Type check | ✅ | No errors |
| Lint | ✅ | 0 errors, N warnings |
| Unit tests | ✅ | X passed, 0 failed |
| Build | ✅ | Compiled successfully |
| Integration | ✅/⏭️ | {result or "N/A"} |

---

## Files Changed

| File | Action | Lines |
|------|--------|-------|
| `src/x.ts` | CREATE | +{N} |
| `src/y.ts` | UPDATE | +{N}/-{M} |

---

## Deviations from Plan

{편차 목록과 이유, 또는 "None"}

---

## Issues Encountered

{발생한 이슈와 해결 방법 목록, 또는 "None"}

---

## Tests Written

| Test File | Test Cases |
|-----------|------------|
| `src/x.test.ts` | {test functions list} |

---

## Next Steps

- [ ] 구현 리뷰
- [ ] PR 생성: `gh pr create` (해당 시)
- [ ] 승인 후 머지
```

---

### Phase 3-ALT: Ralph Loop (자율 구현)

**목적**: 사람 개입 없이 검증 통과할 때까지 자동 반복

```
INPUT:  Plan 파일 + 최대 반복 횟수
        ".claude/PRPs/plans/auth.plan.md --max-iterations 20"

PROCESS:
        ┌─────────────────────────────────────────────────┐
        │              State 파일 생성                     │
        │        .claude/prp-ralph.state.md               │
        │                                                 │
        │  - iteration: 1                                 │
        │  - max_iterations: 20                           │
        │  - plan_path: "..."                             │
        │  - Codebase Patterns: (재사용 가능한 패턴)      │
        │  - Progress Log: (각 반복의 기록)               │
        └─────────────────────────────────────────────────┘
                              │
                              ▼
        ┌─────────────────────────────────────────────────┐
        │              Iteration N                         │
        │                                                 │
        │  1. 이전 상태 읽기                              │
        │     - Codebase Patterns 섹션 확인              │
        │     - Progress Log 확인 (이전 반복에서 무엇을 했나) │
        │     - Git status 확인                          │
        │                                                 │
        │  2. 작업 식별                                   │
        │     - 아직 완료되지 않은 Task                   │
        │     - 실행할 검증 명령어                        │
        │     - 충족해야 할 수용 기준                     │
        │                                                 │
        │  3. 구현                                        │
        │     - Task 요구사항 읽기                        │
        │     - MIRROR/패턴 참조 읽기                     │
        │     - 변경 구현                                 │
        │     - Task별 검증 실행                          │
        │                                                 │
        │  4. 전체 검증 실행                              │
        │     - type-check                                │
        │     - lint                                      │
        │     - test                                      │
        │     - build                                     │
        │                                                 │
        │  5. 결과 추적                                   │
        │     - Plan 파일 업데이트 (완료된 Task 체크)      │
        │     - Progress Log 업데이트                     │
        │     - Codebase Patterns에 발견한 패턴 추가      │
        └─────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
              검증 실패            검증 전부 통과
                    │                   │
                    ▼                   ▼
        ┌─────────────────┐   ┌─────────────────────────┐
        │  응답 종료      │   │  <promise>COMPLETE      │
        │  (promise 없음) │   │  </promise> 출력        │
        └─────────────────┘   └─────────────────────────┘
                    │                   │
                    ▼                   ▼
        ┌─────────────────┐   ┌─────────────────────────┐
        │  Stop Hook      │   │  Stop Hook              │
        │  → 재실행       │   │  → 루프 종료            │
        │  (iteration++)  │   │  → 아카이브 생성        │
        └─────────────────┘   └─────────────────────────┘

OUTPUT: 작동하는 코드 + 학습 기록 아카이브
```

**핵심 메커니즘: `<promise>COMPLETE</promise>`**

- 모든 검증 통과 시에만 이 태그 출력
- Stop Hook이 이 태그를 감지하면 루프 종료
- 태그 없으면 같은 프롬프트로 다음 iteration 시작

**State 파일 구조**:

```markdown
---
iteration: 1
max_iterations: 20
plan_path: "{file_path}"
input_type: "{plan|prd}"
started_at: "{ISO timestamp}"
---

# PRP Ralph Loop State

## Codebase Patterns
(재사용 가능한 패턴을 여기에 통합 - 미래 iteration이 먼저 읽음)
- Use `sql<number>` template for type-safe SQL aggregations
- Always use `IF NOT EXISTS` in migrations
- Export types from actions.ts for UI components

## Current Task
Execute PRP plan and iterate until all validations pass.

## Plan Reference
{file_path}

## Instructions
1. Plan 파일 읽기
2. 모든 미완료 Task 구현
3. Plan의 모든 검증 명령어 실행
4. 검증 실패 시: 수정 후 재검증
5. Plan 파일 업데이트: 완료된 Task 체크, 노트 추가
6. 모든 검증 통과 시: <promise>COMPLETE</promise> 출력

## Progress Log
(각 iteration 후 학습 내용 추가)

### Iteration 1 - {ISO timestamp}

#### Completed
- {Task 1 요약}
- {Task 2 요약}

#### Validation Status
- Type-check: PASS/FAIL ({에러 수 if failing})
- Lint: PASS/FAIL
- Tests: PASS/FAIL ({X/Y passing})
- Build: PASS/FAIL

#### Learnings
- {발견된 패턴: "이 코드베이스는 Y에 X를 사용"}
- {발견된 Gotcha: "W 할 때 Z를 잊지 말 것"}
- {컨텍스트: "컴포넌트 X는 디렉토리 Y에 있음"}

#### Next Steps
- {아직 해야 할 것}
- {해결해야 할 구체적인 blocker}

---
```

**자기 참조 피드백 루프**:

```
Iteration 1: 구현 시도 → lint 실패 → "이 코드베이스는 X 패턴 사용" 학습
     │
     ▼
Iteration 2: 학습 내용 읽기 → X 패턴 적용 → test 실패 → "Y는 Z 디렉토리" 학습
     │
     ▼
Iteration 3: 모든 학습 적용 → 전체 검증 통과 → COMPLETE
```

**완료 시 아카이브 프로세스**:

```bash
# 아카이브 디렉토리 생성
DATE=$(date +%Y-%m-%d)
PLAN_NAME=$(basename {plan_path} .plan.md)
ARCHIVE_DIR=".claude/PRPs/ralph-archives/${DATE}-${PLAN_NAME}"
mkdir -p "$ARCHIVE_DIR"

# State 파일 복사 (모든 학습 내용 포함)
cp .claude/prp-ralph.state.md "$ARCHIVE_DIR/state.md"

# Plan 복사
cp {plan_path} "$ARCHIVE_DIR/plan.md"

# 통합된 학습 추출
cp .claude/PRPs/reports/{plan-name}-report.md "$ARCHIVE_DIR/learnings.md"

# State 파일 정리
rm .claude/prp-ralph.state.md

# Plan을 completed로 이동
mkdir -p .claude/PRPs/plans/completed
mv {plan_path} .claude/PRPs/plans/completed/
```

---

### Phase 4: Commit & PR (완료)

**목적**: 변경사항을 버전 관리하고 리뷰 요청

```
INPUT:  구현 완료된 코드

PROCESS:
        1. Git 상태 확인
           - 현재 브랜치 (main/master가 아니어야 함)
           - 커밋되지 않은 변경사항
           - PR할 커밋이 있는지
           - 이미 존재하는 PR 확인

        2. 파일 스테이징
           - (blank) → 모든 변경사항
           - `staged` → 현재 스테이징 사용
           - `*.ts` → 특정 패턴만
           - `except tests` → 테스트 제외

        3. 커밋 메시지 생성
           - Conventional Commits 형식: {type}: {description}
           - Types: feat, fix, refactor, docs, test, chore

        4. PR 템플릿 확인
           - .github/PULL_REQUEST_TEMPLATE.md 검색
           - 템플릿 있으면 사용, 없으면 기본 형식

        5. Push 및 PR 생성
           - git push -u origin HEAD
           - gh pr create --title --body

        6. 이슈 연결
           - 커밋 메시지에서 이슈 참조 추출
           - "Fixes #123", "Closes #123" 등

OUTPUT: GitHub Pull Request URL
```

**PR 기본 형식**:

```markdown
## Summary

{PR이 달성하는 것의 1-2문장 설명}

## Changes

{커밋 요약 목록}
- {commit 1}
- {commit 2}

## Files Changed

{count} files changed

<details>
<summary>File list</summary>

{변경된 파일 목록}

</details>

## Testing

- [ ] Type check passes
- [ ] Tests pass
- [ ] Manually verified

## Related Issues

{커밋 메시지에서 링크된 이슈들, 또는 "None"}
```

---

### Phase 5: Review (코드 리뷰)

**목적**: PR 코드의 품질 검증

```
INPUT:  PR 번호 또는 URL
        "123" 또는 "https://github.com/.../pull/123"

PROCESS:
        1. PR 컨텍스트 수집 (FETCH)
           - PR 메타데이터 가져오기 (gh pr view)
           - diff 가져오기 (gh pr diff)
           - 변경된 파일 목록
           - 기존 리뷰 코멘트

        2. 컨텍스트 이해 (CONTEXT)
           - CLAUDE.md 읽기 (프로젝트 규칙)
           - 구현 아티팩트 찾기 (implementation report)
           - PR 의도 파악
           - 변경된 파일 분류

        3. 코드 분석 (REVIEW)
           For each 변경된 파일:
           - 전체 파일 읽기 (diff만 아닌)
           - 유사한 파일 읽기 (기대 패턴 이해)
           - 구체적인 변경사항 체크

        4. 리뷰 체크리스트 실행
           #### Correctness (정확성)
           - [ ] 코드가 PR 주장대로 동작하나?
           - [ ] 로직 에러가 있나?
           - [ ] 엣지 케이스가 처리되나?
           - [ ] 에러 핸들링이 적절한가?

           #### Type Safety (타입 안전성)
           - [ ] 모든 타입이 명시적인가 (암시적 `any` 없음)?
           - [ ] 리턴 타입이 선언되었나?
           - [ ] 인터페이스가 적절히 사용되나?

           #### Pattern Compliance (패턴 준수)
           - [ ] 기존 코드베이스 패턴을 따르나?
           - [ ] 네이밍이 프로젝트 컨벤션과 일치하나?
           - [ ] 파일 구조가 올바른가?

           #### Security (보안)
           - [ ] 검증 없는 사용자 입력이 있나?
           - [ ] 노출될 수 있는 시크릿이 있나?
           - [ ] 인젝션 취약점이 있나?

           #### Performance (성능)
           - [ ] 명백한 N+1 쿼리나 루프가 있나?
           - [ ] 불필요한 async/await가 있나?
           - [ ] 메모리 누수 가능성이 있나?

           #### Completeness (완전성)
           - [ ] 새 코드에 테스트가 있나?
           - [ ] 필요시 문서가 업데이트되었나?
           - [ ] 모든 TODO가 처리되었나?

        5. 자동 검증 실행 (VALIDATE)
           - type-check
           - lint
           - tests
           - build

        6. 이슈 분류 (CATEGORIZE)
           | Level | Icon | Criteria |
           |-------|------|----------|
           | Critical | RED | 블로킹 - 반드시 수정 |
           | High | ORANGE | 머지 전에 수정해야 함 |
           | Medium | YELLOW | 고려해야 함 |
           | Low | BLUE | 제안사항 |

        7. 결정 (DECIDE)
           - APPROVE: Critical/High 이슈 없음, 모든 검증 통과
           - REQUEST CHANGES: High 이슈 존재, 수정 가능한 문제
           - BLOCK: Critical 보안/데이터 이슈, 근본적 접근 문제

        8. GitHub에 게시 (PUBLISH)
           - gh pr review --approve/--request-changes --body-file

OUTPUT: PR 코멘트 + 리뷰 리포트 (.claude/PRPs/reviews/pr-{NUMBER}-review.md)
```

---

### Phase 6: Debug (근본 원인 분석)

**목적**: 증상이 아닌 실제 근본 원인 찾기

```
INPUT:  이슈/에러/스택트레이스
        "로그인 시 500 에러 발생" 또는 스택트레이스

PROCESS:
        1. 분류 (CLASSIFY)
           - 입력 타입 판단 (Raw symptom vs Pre-diagnosed)
           - 모드 결정 (--quick: 2-3 Whys, 기본: full 5 Whys)
           - 증상 재진술

        2. 가설 수립 (HYPOTHESIZE)
           | Hypothesis | What must be true | Evidence needed | Likelihood |
           |------------|-------------------|-----------------|------------|
           | {H1} | {conditions} | {proof needed} | HIGH/MED/LOW |

        3. 5 Whys 조사 (INVESTIGATE)
           WHY 1: Why does [symptom] occur?
           → Because [intermediate cause A]
           → Evidence: [code reference, log, or test that proves this]

           WHY 2: Why does [intermediate cause A] happen?
           → Because [intermediate cause B]
           → Evidence: [proof]

           ... (5번 반복 또는 코드에 도달할 때까지)

           WHY 5: Why does [intermediate cause D] happen?
           → Because [ROOT CAUSE]
           → Evidence: [exact file:line reference]

        4. 검증 (VALIDATE)
           | Test | Question | Pass? |
           |------|----------|-------|
           | Causation | 근본 원인이 증거 체인을 통해 논리적으로 증상으로 이어지나? |
           | Necessity | 근본 원인이 없었다면 증상이 여전히 발생했을까? | N 필요 |
           | Sufficiency | 근본 원인만으로 충분한가, 공동 요인이 있나? |

        5. Git 히스토리 (Deep Mode)
           - 언제 문제 코드가 도입되었나?
           - 어떤 커밋/PR이 추가했나?
           - 최근에 변경되었나 안정적이었나?

OUTPUT: RCA 리포트 (.claude/PRPs/debug/rca-{issue-slug}.md)
```

**유효한 증거 vs 무효한 증거**:

| Valid Evidence | Invalid Evidence |
|----------------|------------------|
| `file.ts:123` with actual code snippet | "likely includes...", "probably because..." |
| 실제로 실행한 명령어 출력 | 코드 증거 없는 논리적 추론 |
| 동작을 증명하는 실행한 테스트 | 기술이 일반적으로 어떻게 작동하는지 설명 |

---

### Phase 7: Issue Workflow (이슈 워크플로우)

**두 단계로 구성**: Investigate → Fix

#### 7.1 Issue Investigate (이슈 조사)

```
INPUT:  GitHub 이슈 번호 또는 자유 텍스트 설명
        "123" 또는 "#123" 또는 "https://github.com/.../issues/123"
        또는 "로그인 버튼이 작동하지 않음"

PROCESS:
        1. 파싱 (PARSE)
           - 입력 타입 결정 (GitHub 이슈 vs 자유 텍스트)
           - 이슈 타입 분류
             | Type | Indicators |
             |------|------------|
             | BUG | "broken", "error", "crash", stack trace |
             | ENHANCEMENT | "add", "support", "feature" |
             | REFACTOR | "clean up", "improve", "simplify" |
             | CHORE | "update", "upgrade", "maintenance" |
             | DOCUMENTATION | "docs", "readme", "clarify" |

           - 평가 (Assessment)
             | Metric | Value | Reasoning |
             |--------|-------|-----------|
             | Severity (BUG) 또는 Priority (기타) | CRITICAL/HIGH/MEDIUM/LOW | {왜 이 값인지} |
             | Complexity | LOW/MEDIUM/HIGH | {파일 수, 통합 지점, 리스크 기반} |
             | Confidence | HIGH/MEDIUM/LOW | {증거 품질, 불확실성 기반} |

        2. 코드베이스 탐색 (EXPLORE)
           - 관련 기능에 직접적으로 관련된 파일
           - 현재 구현이 어떻게 작동하는지
           - 통합 지점 - 무엇이 이것을 호출하고, 이것이 무엇을 호출하는지
           - 미러링할 유사한 패턴
           - 에러 핸들링 패턴

        3. 분석 (ANALYZE)
           For BUG: 5 Whys 근본 원인 분석
           For ENHANCEMENT: 무엇을 추가/변경해야 하는지

        4. 아티팩트 생성 (GENERATE)
           - 상세한 구현 계획
           - 실제 코드 스니펫 포함
           - 검증 명령어 포함

        5. GitHub에 게시 (POST) - GitHub 이슈인 경우
           - 조사 결과를 이슈 코멘트로 게시
           - `/prp-issue-fix {number}`로 구현 안내

OUTPUT: 조사 아티팩트 (.claude/PRPs/issues/issue-{number}.md)
        + GitHub 코멘트 (해당 시)
```

#### 7.2 Issue Fix (이슈 수정)

```
INPUT:  이슈 번호 또는 아티팩트 경로
        "123" 또는 ".claude/PRPs/issues/issue-123.md"

PROCESS:
        1. 아티팩트 로드 (LOAD)
           - 아티팩트 찾기 및 파싱
           - 핵심 섹션 추출 (파일, 단계, 검증)

        2. 계획 검증 (VALIDATE)
           - 아티팩트의 "현재 코드"가 실제와 일치하는지 확인
           - Drift 감지 시 경고

        3. Git 상태 확인 (GIT-CHECK)
           - 적절한 브랜치에 있는지 확인
           - 브랜치 생성 또는 기존 사용

        4. 구현 (IMPLEMENT)
           - 아티팩트의 각 단계 실행
           - 기존 코드 스타일 정확히 매칭
           - 편차 발생 시 문서화

        5. 검증 (VERIFY)
           - 아티팩트의 검증 명령어 실행
           - 모두 통과해야 진행

        6. 커밋 (COMMIT)
           - Conventional Commits 형식
           - "Fixes #{issue-number}" 포함

        7. PR 생성 (PR)
           - 이슈에 연결된 PR 생성

        8. 셀프 리뷰 (REVIEW)
           - 변경사항 코드 리뷰
           - PR에 리뷰 결과 게시

        9. 아카이브 (ARCHIVE)
           - 아티팩트를 completed 폴더로 이동

OUTPUT: PR + 셀프 리뷰 코멘트
        아티팩트 아카이브 (.claude/PRPs/issues/completed/)
```

---

## 4. 문서 흐름도

```
                    ┌─────────────────────────────┐
                    │         사용자 요청          │
                    └──────────────┬──────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
        ┌──────────┐        ┌──────────┐        ┌──────────┐
        │ 대규모   │        │ 중간     │        │ 이슈/버그 │
        └────┬─────┘        └────┬─────┘        └────┬─────┘
             │                   │                   │
             ▼                   │                   ▼
        ┌──────────┐             │            ┌────────────┐
        │ /prp-prd │             │            │ /prp-debug │
        │          │             │            │     또는    │
        │ .prd.md  │             │            │ /prp-issue-│
        └────┬─────┘             │            │ investigate│
             │                   │            └────┬───────┘
             └───────────┬───────┴─────────────────┘
                         │
                         ▼
                  ┌──────────────┐
                  │  /prp-plan   │
                  │              │
                  │  .plan.md    │
                  └──────┬───────┘
                         │
            ┌────────────┴────────────┐
            │                         │
            ▼                         ▼
     ┌──────────────┐         ┌──────────────┐
     │ /prp-        │         │ /prp-ralph   │
     │ implement    │         │              │
     │              │         │ (자율 루프)  │
     │ (수동 실행)  │         └──────┬───────┘
     └──────┬───────┘                │
            │                        │
            └────────────┬───────────┘
                         │
                         ▼
                  ┌──────────────┐
                  │ /prp-commit  │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │ /prp-pr      │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │ /prp-review  │
                  └──────────────┘
```

**이슈 수정 경로:**

```
/prp-issue-investigate {number}
           │
           ▼
   .claude/PRPs/issues/issue-{number}.md
           │
           ▼
/prp-issue-fix {number}
           │
           ▼
   PR + 셀프 리뷰 + 아티팩트 아카이브
```

---

## 5. 파일 구조

```
.claude/
├── PRPs/
│   ├── prds/                    # PRD 문서들
│   │   └── feature.prd.md       # "무엇을 왜"
│   │
│   ├── plans/                   # Plan 문서들
│   │   ├── feature.plan.md      # "어떻게"
│   │   └── completed/           # 완료된 Plan 아카이브
│   │
│   ├── reports/                 # 구현 보고서
│   │   └── feature-report.md    # 구현 결과 + Assessment vs Reality
│   │
│   ├── issues/                  # 이슈 조사 결과
│   │   ├── issue-123.md         # 조사 아티팩트
│   │   └── completed/           # 완료된 이슈 아카이브
│   │
│   ├── debug/                   # 디버그/RCA 리포트
│   │   └── rca-{issue-slug}.md  # 근본 원인 분석
│   │
│   ├── reviews/                 # PR 리뷰 리포트
│   │   └── pr-{number}-review.md
│   │
│   └── ralph-archives/          # Ralph 루프 기록
│       └── 2024-01-16-feature/
│           ├── state.md         # 최종 상태 + 학습 내용
│           ├── plan.md          # 사용된 Plan
│           └── learnings.md     # 통합된 학습
│
├── prp-ralph.state.md           # Ralph 루프 활성 상태 (임시)
│
└── commands/prp-core/           # 명령어 정의
    ├── prp-prd.md               # PRD 생성
    ├── prp-plan.md              # Plan 생성
    ├── prp-implement.md         # 구현 실행
    ├── prp-ralph.md             # 자율 루프
    ├── prp-ralph-cancel.md      # Ralph 취소
    ├── prp-commit.md            # 커밋
    ├── prp-pr.md                # PR 생성
    ├── prp-review.md            # 코드 리뷰
    ├── prp-debug.md             # 근본 원인 분석
    ├── prp-issue-investigate.md # 이슈 조사
    └── prp-issue-fix.md         # 이슈 수정
```

---

## 6. 핵심 연결 고리

### PRD ↔ Plan 연결

```
PRD (feature.prd.md)
┌─────────────────────────────────────────────────────┐
│ Implementation Phases:                              │
│ | # | Phase    | Status      | Parallel | Depends | PRP Plan           |
│ | 1 | 기본설정 | complete    | -        | -       | phase-1.plan.md    |
│ | 2 | 핵심기능 | in-progress | -        | 1       | phase-2.plan.md ◄──┼── 현재 작업 중
│ | 3 | UI       | pending     | with 4   | 2       | -                  |
│ | 4 | Auth     | pending     | with 3   | 2       | -                  |
│ | 5 | Deploy   | pending     | -        | 3, 4    | -                  |
└─────────────────────────────────────────────────────┘
                        │
                        │ /prp-plan feature.prd.md
                        │ (자동으로 Phase 2 선택, Phase 3-4 병렬 가능 안내)
                        ▼
Plan (phase-2.plan.md)
┌─────────────────────────────────────────────────────┐
│ Source PRD: feature.prd.md                          │
│ Phase: #2 - 핵심기능                                │
│                                                     │
│ Tasks:                                              │
│ - Task 1: CREATE models.ts                          │
│ - Task 2: CREATE service.ts                         │
│ - ...                                               │
└─────────────────────────────────────────────────────┘
```

### Plan ↔ 코드베이스 연결

```
Plan (feature.plan.md)
┌─────────────────────────────────────────────────────┐
│ Mandatory Reading:                                  │
│ | Priority | File                    | Why          │
│ | P0       | src/features/X/srv.ts   | MIRROR 패턴  │
│ | P1       | src/features/X/types.ts | IMPORT 타입  │
│                                                     │
│ Patterns to Mirror:                                 │
│                                                     │
│ NAMING_CONVENTION:                                  │
│ // SOURCE: src/features/users/service.ts:10-15     │
│ export function createUser() { ... }               │◄── 실제 코드 복사
│                                                     │
│ Task 1:                                             │
│ - MIRROR: src/features/users/models.ts:1-20        │◄── 이 파일 패턴 따라서
│ - IMPLEMENT: 새 모델 생성                           │
│ - GOTCHA: $inferSelect for read, $inferInsert for write │
│ - VALIDATE: npx tsc --noEmit                        │
└─────────────────────────────────────────────────────┘
```

### Ralph 상태 ↔ Stop Hook 연결

```
State (.claude/prp-ralph.state.md)
┌─────────────────────────────────────────────────────┐
│ iteration: 3                                        │
│ max_iterations: 20                                  │
│                                                     │
│ ## Codebase Patterns                                │
│ - X 패턴 사용                                        │◄── 다음 iteration에서 읽힘
│ - Y는 Z 디렉토리                                     │
│ - sql<number> template for type-safe SQL            │
│                                                     │
│ ## Progress Log                                     │
│ ### Iteration 1: lint 실패                          │
│ - Learnings: "이 코드베이스는 single quotes 사용"   │
│ ### Iteration 2: test 실패                          │
│ - Learnings: "mock은 __mocks__ 디렉토리에"          │
│ ### Iteration 3: 전부 통과 ✅                       │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
Stop Hook 감지
┌─────────────────────────────────────────────────────┐
│ if <promise>COMPLETE</promise> in output:           │
│     → Implementation Report 생성                    │
│     → state 파일을 ralph-archives로 이동            │
│     → Plan을 completed로 이동                       │
│     → CLAUDE.md에 영구 패턴 추가 (해당 시)          │
│     → 루프 종료                                     │
│ else:                                               │
│     → iteration++                                   │
│     → 같은 프롬프트 재전달                          │
└─────────────────────────────────────────────────────┘
```

### Issue 조사 ↔ 수정 연결

```
Investigation Artifact (.claude/PRPs/issues/issue-123.md)
┌─────────────────────────────────────────────────────┐
│ # Investigation: Login Button Not Working           │
│                                                     │
│ ## Assessment                                       │
│ | Metric | Value | Reasoning |                      │
│ | Severity | HIGH | 사용자 로그인 불가, 핵심 기능   │
│ | Complexity | LOW | 1개 파일, 명확한 수정         │
│ | Confidence | HIGH | 명확한 근본 원인, 증거 확실  │
│                                                     │
│ ## Evidence Chain                                   │
│ WHY: 클릭 시 아무 반응 없음                         │
│ ↓ BECAUSE: onClick 핸들러 누락                      │
│   Evidence: `src/Login.tsx:45` - <button> tag       │
│ ↓ ROOT CAUSE: handleSubmit 함수 연결 안 됨          │
│   Evidence: `src/Login.tsx:12` - 정의만 있고 사용X  │
│                                                     │
│ ## Implementation Plan                              │
│ Step 1: UPDATE `src/Login.tsx:45`                   │
│   Current: <button>Login</button>                   │
│   Required: <button onClick={handleSubmit}>         │
│                                                     │
│ ## Validation                                       │
│ npm run type-check && npm test && npm run build     │
└─────────────────────────────────────────────────────┘
                        │
                        │ /prp-issue-fix 123
                        ▼
Execution
┌─────────────────────────────────────────────────────┐
│ 1. 아티팩트 로드 및 현재 코드와 검증                 │
│ 2. 브랜치 생성: fix/issue-123-login-button          │
│ 3. Step 1 실행: Login.tsx 수정                      │
│ 4. 검증: type-check, test, build 통과               │
│ 5. 커밋: "fix: Add onClick handler to login button" │
│ 6. PR 생성: Fixes #123                              │
│ 7. 셀프 리뷰 + 코멘트                               │
│ 8. 아티팩트 → issues/completed/                     │
└─────────────────────────────────────────────────────┘
```

---

## 7. 검증 체계

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              검증 피라미드                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│                          ┌─────────────┐                                │
│                          │   Manual    │  Level 6                       │
│                          │ Validation  │  수동 테스트 체크리스트         │
│                          └──────┬──────┘                                │
│                                 │                                       │
│                     ┌───────────┴───────────┐                           │
│                     │       Browser         │  Level 5                  │
│                     │     Validation        │  UI가 작동하나? (MCP)      │
│                     └───────────┬───────────┘                           │
│                                 │                                       │
│              ┌──────────────────┴──────────────────┐                    │
│              │           Database                   │  Level 4          │
│              │          Validation                  │  테이블/RLS 맞나?  │
│              └──────────────────┬──────────────────┘                    │
│                                 │                                       │
│           ┌─────────────────────┴─────────────────────┐                 │
│           │                 Build                      │  Level 3       │
│           │                                            │  배포 가능한가? │
│           └─────────────────────┬─────────────────────┘                 │
│                                 │                                       │
│        ┌────────────────────────┴────────────────────────┐              │
│        │                  Unit Tests                      │  Level 2    │
│        │                                                  │  기능 맞나?  │
│        └────────────────────────┬────────────────────────┘              │
│                                 │                                       │
│  ┌──────────────────────────────┴──────────────────────────────┐        │
│  │                    Static Analysis                          │ Level 1│
│  │              (type-check + lint)                            │ 문법OK?│
│  └─────────────────────────────────────────────────────────────┘        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

검증 타이밍:
- Task 단위: Level 1만 (빠른 피드백)
- 전체 완료: Level 1-6 전부 (완전한 검증)

검증 명령어 예시:
┌─────────────────────────────────────────────────────────────────────────┐
│ Level 1: {runner} run type-check && {runner} run lint                   │
│ Level 2: {runner} test {path/to/feature/tests}                          │
│ Level 3: {runner} run build                                             │
│ Level 4: Supabase MCP로 테이블/RLS 확인                                 │
│ Level 5: Browser MCP로 UI 플로우 확인                                   │
│ Level 6: 수동 테스트 단계 실행                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 8. 한 문장 요약

| 개념 | 한 문장 |
|------|---------|
| **PRP** | AI가 한 번에 성공하도록 모든 맥락을 담은 구현 가이드 |
| **PRD** | "무엇을 왜 만드는가" + 증거 + 가설 + 단계 분할표 + 리스크 |
| **Plan** | "어떻게 만드는가" + 코드베이스 패턴 + UX 설계 + 6단계 검증 명령어 |
| **Implement** | Plan을 따라 구현하고 매번 검증 + 편차 문서화 + PRD 자동 업데이트 |
| **Ralph** | 검증 통과할 때까지 자동 반복 + Codebase Patterns 누적 |
| **Stop Hook** | `<promise>COMPLETE</promise>` 감지해서 루프 제어 |
| **Review** | PR 코드 리뷰 + 이슈 분류 + GitHub 게시 |
| **Debug** | 5 Whys로 근본 원인 찾기 + Evidence Chain |
| **Issue Workflow** | Investigate(조사) → Fix(수정) 2단계 프로세스 |
| **Pattern Mirroring** | 새 패턴 발명 금지, 기존 코드 복사 |
| **State 파일** | Ralph 루프의 학습 내용 누적 저장소 |
| **Assessment** | Severity/Priority + Complexity + Confidence + Reasoning |

---

## 9. 왜 이렇게 설계했나?

### 문제: AI는 맥락 없이 추측한다

```
일반 프롬프트 → AI 추측 → 코드베이스 스타일과 불일치 → 수정 반복 → 시간 낭비
```

### 해결: 모든 맥락을 미리 제공한다

```
PRP (맥락 풍부) → AI가 정확히 이해 → 코드베이스 스타일 일치 → 한 번에 성공
```

### 문제: 검증 없이 진행하면 오류가 누적된다

```
Task 1 → Task 2 → Task 3 → 마지막에 검증 → 100개 에러 → 어디서 틀렸는지 모름
```

### 해결: 매 단계 즉시 검증한다

```
Task 1 → 검증 → Task 2 → 검증 → Task 3 → 검증 → 마지막 검증 → 성공
                                               (에러 즉시 발견 및 수정)
```

### 문제: 사람이 계속 개입해야 한다

```
구현 → 실패 → 사람에게 보고 → 지시 대기 → 구현 → 실패 → ... (느림)
```

### 해결: 자율 루프로 자동 반복한다 (Ralph)

```
구현 → 실패 → 자동 수정 → 구현 → 실패 → 자동 수정 → 성공 (빠름)
        ↑____________________|
        (Stop Hook이 자동 재실행)
```

### 문제: 학습이 누적되지 않는다

```
실패 → 수정 → 다음 실패 → 같은 실수 반복 → 비효율적
```

### 해결: Codebase Patterns를 누적 저장한다

```
Iteration 1 학습 → State에 저장 → Iteration 2가 읽음 → 같은 실수 안 함
```

### 문제: 근본 원인 대신 증상만 수정한다

```
에러 발생 → 즉시 수정 시도 → 다른 곳에서 같은 에러 → 무한 반복
```

### 해결: 5 Whys로 근본 원인을 찾는다

```
증상 → WHY → 중간 원인 → WHY → ... → 근본 원인 발견 → 한 번에 해결
```

---

## 10. 시작하기

```bash
# 대규모 기능 (3+ 단계)
/prp-prd "사용자 인증 시스템"
/prp-plan .claude/PRPs/prds/user-auth.prd.md
/prp-ralph .claude/PRPs/plans/user-auth-phase-1.plan.md

# 중간 기능 (단일 단계)
/prp-plan "페이지네이션 추가"
/prp-implement .claude/PRPs/plans/pagination.plan.md

# 이슈/버그 수정
/prp-issue-investigate 123
/prp-issue-fix 123

# 디버깅 (근본 원인 분석)
/prp-debug "로그인 시 500 에러 발생"
/prp-debug --quick "TypeError: Cannot read property..."

# 코드 리뷰
/prp-review 456
/prp-review https://github.com/owner/repo/pull/456

# 커밋 및 PR
/prp-commit                    # 모든 변경사항
/prp-commit typescript files   # *.ts 만
/prp-commit except tests       # 테스트 제외
/prp-pr                        # PR 생성

# Ralph 취소
/prp-ralph-cancel
```

---

## 11. 명령어 요약표

| 명령어 | 목적 | 입력 | 출력 |
|--------|------|------|------|
| `/prp-prd` | PRD 생성 | 아이디어/기능 설명 | `.claude/PRPs/prds/*.prd.md` |
| `/prp-plan` | Plan 생성 | PRD 또는 기능 설명 | `.claude/PRPs/plans/*.plan.md` |
| `/prp-implement` | Plan 실행 | Plan 파일 경로 | 작동하는 코드 + report |
| `/prp-ralph` | 자율 루프 | Plan/PRD + max iterations | 완성된 코드 + archive |
| `/prp-ralph-cancel` | Ralph 취소 | - | 루프 중단 |
| `/prp-commit` | 커밋 생성 | 타겟 설명 (optional) | git commit |
| `/prp-pr` | PR 생성 | base branch (optional) | GitHub PR |
| `/prp-review` | 코드 리뷰 | PR 번호/URL | PR 코멘트 + review report |
| `/prp-debug` | 근본 원인 분석 | 에러/증상 | `.claude/PRPs/debug/rca-*.md` |
| `/prp-issue-investigate` | 이슈 조사 | 이슈 번호/설명 | `.claude/PRPs/issues/issue-*.md` |
| `/prp-issue-fix` | 이슈 수정 | 이슈 번호 | PR + 셀프 리뷰 |
