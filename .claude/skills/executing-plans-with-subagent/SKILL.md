---
name: executing-plans-with-subagent
description: Use when executing implementation plans with independent tasks in the current session - dispatches fresh subagent for each task with code review between tasks, enabling fast iteration with quality gates
---

# Executing Plans with Subagent

## Overview
Execute plan by dispatching fresh subagent per task, with code review after each.

**Core principle:**
- Fresh subagent per task + review between tasks = high quality, fast iteration

**When to use:**
- Staying in this session
- Tasks are mostly independent
- Want continuous progress with quality gates

**When NOT to use:**
- Need to review plan first (use writing-plans skill)
- Tasks are tightly coupled (manual execution better)
- Plan needs revision (brainstorm first)

**Announce at start:** "I'm using the executing-plans-with-subagent skill to implement this plan."

## Workflow

### Phase 1: Load and Review Plan

**Goal**: Understand the plan and identify parallel execution opportunities

**Actions**:
1. Read plan file
2. Review critically - identify any questions or concerns
3. Check for parallel notation:
   - Look for "Task Dependencies" graph with `[x, y, z]` brackets
   - Look for "Parallel Group" headers
   - Look for "> **Parallel Execution:**" callouts
4. If concerns: Raise them with your human partner before starting
5. If no concerns: Create TodoWrite with all tasks and proceed

### Phase 2: Execute Task

**Goal**: Implement task by dispatching fresh subagent

**Actions**:

**For sequential tasks:**
1. Mark task as in_progress
2. Get git SHA before task
3. Dispatch fresh subagent:
```
Task tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N from [plan-file].

    Read that task carefully. Your job is to:
    1. Implement exactly what the task specifies
    2. Run verification from the task's **Verification** section:
       - If Test specified: Run the test file, must pass
       - If testIDs specified: Use Expo MCP automation_find_view for each testID
    3. If ANY verification fails: STOP immediately, report the failure, do NOT commit
    4. Only if all verification passes: Commit using **commit-helper skill**
    5. Report back

    Work from: [directory]

    Report:
    - What you implemented
    - Test results (pass/fail, which tests)
    - testID verification results (found/not found, which testIDs)
    - Files changed
    - Any issues encountered
```

**For parallel groups:**
1. Get git SHA before parallel group
2. Dispatch ALL tasks in group simultaneously (single message, multiple Task tool calls):
```
[Single message with multiple Task tool calls:]

Task tool (general-purpose):
  description: "Implement Task 3: UserProfile"
  prompt: |
    You are implementing Task 3 from [plan-file].
    [Same instructions as sequential: implement, verify, only commit if passes]

Task tool (general-purpose):
  description: "Implement Task 4: UserSettings"
  prompt: |
    You are implementing Task 4 from [plan-file].
    [Same instructions as sequential: implement, verify, only commit if passes]

Task tool (general-purpose):
  description: "Implement Task 5: UserNotifications"
  prompt: |
    You are implementing Task 5 from [plan-file].
    [Same instructions as sequential: implement, verify, only commit if passes]
```
3. Wait for all to complete
4. **Check each subagent's verification results** - if any failed, address before proceeding
4. **Review each task individually** - dispatch code-reviewers in parallel (one per task):
```
[Single message with multiple code-reviewer Task tool calls:]

Task tool (code-reviewer):
  description: "Review Task 3: UserProfile"
  ...

Task tool (code-reviewer):
  description: "Review Task 4: UserSettings"
  ...

Task tool (code-reviewer):
  description: "Review Task 5: UserNotifications"
  ...
```
5. Fix any issues found, then continue to next task/group

### Phase 3: Review

**Goal**: Validate implementation quality via code-reviewer subagent

**Actions**:
1. Dispatch code-reviewer subagent:
```
Task tool (code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from subagent's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```
2. Code reviewer returns: Strengths, Issues (Critical/Important/Minor), Assessment

### Phase 4: Apply Feedback

**Goal**: Address issues found during code review

**Actions**:
1. If Critical issues: Fix immediately via follow-up subagent
2. If Important issues: Fix before next task
3. If Minor issues: Note for later

**Dispatch follow-up subagent if needed:**
```
"Fix issues from code review: [list issues]"
```

### Phase 5: Continue

**Goal**: Iterate until all tasks complete

**Actions**:
1. Mark task as completed in TodoWrite
2. Move to next task
3. Repeat Phases 2-4

### Phase 6: Final Review

**Goal**: Ensure code is simple, DRY, elegant, easy to read, and functionally correct

**Actions**:
1. Say "I will review the code with code-reviewer agents"
2. Launch 3 code-reviewer agents in parallel with different focuses:
   - Simplicity/DRY/elegance
   - Bugs/functional correctness
   - Project conventions/abstractions
3. Consolidate findings and identify highest severity issues that you recommend fixing
4. **Present findings to user and ask what they want to do** (fix now, fix later, or proceed as-is)
5. Address issues based on user decision

### Phase 7: Documentation

**Goal**: Document implementation for future reference

**Actions**:
1. Say: "All tasks complete. Creating implementation.md file."
2. Create implementation.md file at `agent/tasks/{number}-{feature-name}/implementation.md`
3. **EVERY implementation.md file MUST start with the following header:**
```markdown
# [number]-[feature-name] Implementation Log

**Date**: [Current Date]
**Status**: Completed
**Time**: [Duration]

## What Was Built

## Files Created

## Files Modified

## Key Implementation Details
```
4. Commit the changes with the following message:
```bash
git commit -m "feat: add implementation.md file"
```

### Phase 8: Memory Update

**Goal**: Update project documentation to reflect implementation changes

**Actions**:
1. Get git SHA from before plan execution started (captured in Phase 2)
2. Dispatch memory-manager subagent:
```
Task tool (memory-manager):
  description: "Analyze changes and suggest doc updates"
  prompt: |
    Analyze changes from BASE_SHA to HEAD.

    BASE_SHA: [sha from before plan execution]

    Focus on:
    - New patterns introduced
    - Architecture changes
    - Config/tooling changes

    Return suggestions for updates to:
    - agent/system/*.md
    - .claude/skills/*/SKILL.md
    - CLAUDE.md
```
3. Review suggestions returned by subagent
4. Apply appropriate updates (skip if no valuable changes)
5. If updates made, commit: `docs: update project memory after [feature-name]`

## Example: Sequential Workflow

```
You: I'm using executing-plans-with-subagent skill to execute this plan.

[Load plan, create TodoWrite]

Task 1: Hook installation script

[Dispatch implementation subagent]
Subagent: Implemented install-hook with tests, 5/5 passing

[Get git SHAs, dispatch code-reviewer]
Reviewer: Strengths: Good test coverage. Issues: None. Ready.

[Mark Task 1 complete]

Task 2: Recovery modes

[Dispatch implementation subagent]
Subagent: Added verify/repair, 8/8 tests passing

[Dispatch code-reviewer]
Reviewer: Strengths: Solid. Issues (Important): Missing progress reporting

[Dispatch fix subagent]
Fix subagent: Added progress every 100 conversations

[Verify fix, mark Task 2 complete]

...

[After all tasks]
[Dispatch final code-reviewer]
Final reviewer: All requirements met, ready to merge

Done!
```

## Example: Parallel Workflow

```
Plan has: 1 → 2 → [3, 4, 5] → 6
                   (parallel)

[Complete Task 1, Task 2 sequentially...]

Tasks 3-5: Parallel Group - User Components

[Get git SHA before parallel group]

[Single message with 3 Task tool calls:]
- Task 3: UserProfile component
- Task 4: UserSettings component
- Task 5: UserNotifications component

[All 3 subagents work simultaneously]

Subagent 3: Done - UserProfile implemented, tests passing
Subagent 4: Done - UserSettings implemented, tests passing
Subagent 5: Done - UserNotifications implemented, tests passing

[Review each task's changes]
[Mark Tasks 3, 4, 5 complete]

Task 6: User Integration (Sequential)
[Continue normally...]
```

## Advantages

**vs. Manual execution:**
- Fresh context per task (no confusion)
- Parallel-safe (subagents don't interfere)

**vs. Executing Plans:**
- Same session (no handoff)
- Continuous progress (no waiting)
- Review checkpoints automatic

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker mid-task (missing dependency, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- **Subagent reports test failure** - Do not proceed, ask human
- **Subagent reports testID not found** - Do not proceed, ask human
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

**Never skip verification:**
- Don't mark task complete if subagent reported test failure
- Don't mark task complete if subagent reported testID not found
- Don't proceed hoping "it will work later"

## Red Flags

**Never:**
- Skip code review between tasks
- Proceed with unfixed Critical issues
- Dispatch parallel subagents for tasks NOT marked as parallel-safe
- Parallel tasks that share files (check Files: in each task)
- Implement without reading plan task
- **Mark task complete if verification failed**
- **Skip verification steps specified in plan**
- **Commit code that failed tests or testID checks**

**If subagent fails task:**
- Dispatch fix subagent with specific instructions
- Don't try to fix manually (context pollution)

**If subagent reports verification failure:**
- Do NOT mark task as complete
- Stop and ask human for guidance
- Provide details: which test failed, which testID not found

## Integration

**Required workflow skills:**
- **writing-plans** - REQUIRED: Creates the plan that this skill executes
- **requesting-code-review** - REQUIRED: Review after each task (see Phase 3)
- **memory-manager** - REQUIRED: Documentation updates after completion (see Phase 8)

See code-reviewer template: requesting-code-review/references/code-reviewer.md
