---
name: writing-plans
description: Use when design is complete and you need detailed implementation tasks for engineers with zero codebase context - creates comprehensive implementation plans with exact file paths, complete code examples, and verification steps assuming engineer has minimal domain knowledge
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Save plans to:** `agent/tasks/{number}-{feature-name}/plan.md`

## Core Principles

### Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Format

### Plan Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use executing-plans-with-batch or executing-plans-with-subagent to implement this plan task-by-task.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

### Task Structure

```markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.ts`
- Modify: `exact/path/to/existing.ts:123-145`

**Verification:**
- Test: `__tests__/exact/path/to/test.test.ts` (or "Skipped: [justification]")
- testIDs: `submit-button`, `role-input` (for UI tasks, omit if not UI)

**Step 1: Write the failing test**

```typescript
import { describe, it, expect } from 'vitest';
import { functionName } from '@/path/to/module';

describe('functionName', () => {
  it('should handle specific behavior', () => {
    const result = functionName(input);
    expect(result).toBe(expected);
  });
});
```

**Step 2: Run test to verify it fails**

Run: `bun test tests/path/test.test.ts -t "should handle specific behavior"`
Expected: FAIL with "functionName is not defined"

**Step 3: Write minimal implementation**

```typescript
export function functionName(input: InputType): ReturnType {
  return expected;
}
```

**Step 4: Run test to verify it passes**

Run: `bun test tests/path/test.test.ts -t "should handle specific behavior"`
Expected: PASS

**Step 5: Commit**
- Check if there are missing files to commit
- Commit the changes using **commit-helper skill**

### Parallel Task Notation

When tasks can be executed simultaneously, document them clearly to speed up implementation.

**Rules for parallel tasks:**
- Tasks MUST touch completely different files (no shared file modifications)
- Each parallel task should be independently testable
- Parallel groups should have a clear synchronization point

**Format 1: Dependency Graph Header**

Add after the plan header to show task flow:

```markdown
## Task Dependencies

1 → 2 → [3, 4, 5] → 6 → [7, 8] → 9
        (parallel)      (parallel)

**Legend:** `→` sequential, `[x, y, z]` parallel group
```

**Format 2: Parallel Group Notation**

```markdown
### Tasks 3-5: Parallel Group - User Components

> **Parallel Execution:** Tasks 3, 4, 5 have no file dependencies.
> Safe to dispatch 3 subagents simultaneously.
> **Sync point:** All must complete before Task 6.

#### Task 3: UserProfile component
**Files:** `features/user/UserProfile.tsx`, `features/user/UserProfile.test.tsx`

#### Task 4: UserSettings component
**Files:** `features/user/UserSettings.tsx`, `features/user/UserSettings.test.tsx`

#### Task 5: UserNotifications component
**Files:** `features/user/UserNotifications.tsx`, `features/user/UserNotifications.test.tsx`

---

### Task 6: User Integration (Sequential - depends on 3-5)
```

**When parallel is NOT safe:**
- Tasks modify the same file
- Tasks have import/export dependencies on each other
- Tasks affect shared state (database migrations, configs)

**For Claude (executor instruction):**
```markdown
> **For Claude:** Tasks 3-5 are parallel-safe. Use single message with
> multiple Task tool calls to dispatch subagents simultaneously.
```

### Verification Requirements

**Every task MUST have a Verification section.** This ensures Claude can verify its own work.

**Test file rules:**
- Specify test file path: `__tests__/features/[feature]/[type]/[name].test.ts`
- OR provide justification for skipping (must be explicit)

**Valid skip justifications:**
- "Skipped: DB schema only, no runtime code"
- "Skipped: Type generation, verified by TypeScript compilation"
- "Skipped: Config change, verified by build success"

**Invalid skip justifications:**
- "Skipped: Simple change" (vague)
- "Skipped: Will test later" (deferred)
- "Skipped: Manual testing sufficient" (not automated)

**testID rules (UI tasks only):**
- List key testIDs that must be verifiable on screen after implementation
- These will be checked using Expo MCP `automation_find_view`
- Focus on critical elements: submit buttons, primary inputs, key displays

**Example:**
```markdown
**Verification:**
- Test: `__tests__/features/scenario/hooks/use-create-scenario.test.ts`
- testIDs: `scenario-submit-button`, `user-role-input`, `ai-role-input`
```

**Example (non-UI task):**
```markdown
**Verification:**
- Test: `__tests__/features/scenario/api/scenario-api.test.ts`
```

**Example (skip with justification):**
```markdown
**Verification:**
- Test: Skipped: DB schema migration, no runtime code to test
```

## Workflow

### Phase 1: Load and Review Design

**Goal**: Understand the design and identify any issues before writing the plan

**Actions**:
1. Read design file or feature request
2. Review critically - identify any questions or concerns about the design
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Create TodoWrite and proceed

### Phase 2: Write the plan

**Goal**: Write a comprehensive implementation plan assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

**Actions**:
1. Write the plan using relevant skills (e.g. creating-edge-function, app-development, etc.)
2. Review the plan critically
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Commit the plan

## Remember
- Exact file paths always
- Complete code in plan (not "add validation")
- Exact commands with expected output
- Reference relevant skills with @ syntax
- DRY, YAGNI, TDD, frequent commits

## Execution Handoff
After saving the plan, announce:
"Plan complete and saved to `agent/tasks/{number}-{feature-name}/plan.md`.
**1. Subagent-Driven (this session)** - I dispatch fresh subagent per task, review between tasks, fast iteration
**2. Parallel Session (separate)** - Open new session with executing-plans-with-batch, batch execution with checkpoints
Which approach Do you want to use?
"

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use executing-plans-with-subagent skill
- Stay in this session
- Fresh subagent per task + code review

**If Parallel Session chosen:**
- Guide them to open new session in worktree
- **REQUIRED SUB-SKILL:** New session uses executing-plans-with-batch skill
