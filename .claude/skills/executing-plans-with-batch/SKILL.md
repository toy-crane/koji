---
name: executing-plans-with-batch
description: Use when partner provides a complete implementation plan to execute in controlled batches with review checkpoints - loads plan, reviews critically, executes tasks in batches, reports for review between batches
---

# Executing Plans

## Overview

Load plan, review critically, execute tasks in batches, report for review between batches.

**Core principle:** Batch execution with checkpoints for architect review.

**Announce at start:** "I'm using the executing-plans-with-batch skill to implement this plan."

## Workflow

### Phase 1: Load and Review Plan

**Goal**: Understand the plan and identify any issues before execution

**Actions**:
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Create TodoWrite and proceed

### Phase 2: Execute Batch

**Goal**: Implement first batch of tasks (default: first 3 tasks)

**Actions**:
For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. **Run verification (REQUIRED before marking complete):**
   - If task has **Test** specified: Run the test file, must pass
   - If task has **testIDs** specified: Run testID verification (see below)
   - If either fails: **STOP and ask human for guidance**
4. Mark as completed only after all verification passes

**testID Verification Process:**
```
1. Ensure app is running on simulator
2. For each testID in task's Verification section:
   - Use Expo MCP: automation_find_view(testID)
   - If not found: STOP immediately
3. If all testIDs found: Verification passes
```

**When verification fails:**
```
Verification failed: testID "submit-button" not found on screen.

Possible causes:
- Component not rendered
- testID prop missing from implementation
- App crashed or showing wrong screen

What would you like me to do?
```

### Phase 3: Report

**Goal**: Communicate batch completion and wait for feedback

**Actions**:
1. Show what was implemented
2. Show verification output
3. Say: "Ready for feedback."

### Phase 4: Continue

**Goal**: Iterate based on feedback until all tasks complete

**Actions**:
1. Apply changes if needed based on feedback
2. Execute next batch
3. Repeat until complete

### Phase 5: Quality Review After All Tasks Complete

**Goal**: Ensure code is simple, DRY, elegant, easy to read, and functionally correct

**Actions**:
1. Say "I will review the code with code-reviewer agents"
2. Launch 3 code-reviewer agents in parallel with different focuses: simplicity/DRY/elegance, bugs/functional correctness, project conventions/abstractions
3. Consolidate findings and identify highest severity issues that you recommend fixing
4. **Present findings to user and ask what they want to do** (fix now, fix later, or proceed as-is)
5. Address issues based on user decision

### Phase 6: Documentation

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

### Phase 7: Memory Update

**Goal**: Update project documentation to reflect implementation changes

**Actions**:
1. Get git SHA from before plan execution started
2. Dispatch memory-manager subagent:
```
Task tool (memory-manager):
  description: "Analyze changes and suggest doc updates"
  prompt: |
    Analyze changes from BASE_SHA to HEAD.

    BASE_SHA: [sha from before batch execution]

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

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker mid-batch (missing dependency, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- **Test fails** - Do not proceed, ask human
- **testID verification fails** - Do not proceed, ask human
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

**Never skip verification:**
- Don't mark task complete if tests fail
- Don't mark task complete if testIDs not found
- Don't proceed hoping "it will work later"

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Between batches: just report and wait
- Stop when blocked, don't guess