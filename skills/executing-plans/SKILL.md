---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

## CRITICAL CONSTRAINTS

**You MUST NOT call `EnterPlanMode` or `ExitPlanMode` during this skill.** This skill operates in normal mode, executing a plan that already exists on disk. Plan mode is unnecessary and dangerous here — it restricts Write/Edit tools needed for implementation.

# Executing Plans

## Overview

Load plan, review critically, execute tasks in batches, report for review between batches.

**Core principle:** Batch execution with checkpoints for architect review.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

## The Process

### Step 0: Check for Resume
If `3-PLAN.md` has a "Current Status" section:
1. Read the status to understand where to continue
2. Skip completed tasks (marked `[x]`)
3. Resume from the task indicated in "Next action"

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Proceed to task setup

### Step 1b: Bootstrap Tasks from Plan (if needed)

If TaskList returned no tasks or tasks don't match plan:

1. Parse the plan document for `## Task N:` or `### Task N:` headers
2. For each task found, use TaskCreate with:
   - subject: The task title from the plan
   - description: Full structured content (Goal, Files, Acceptance Criteria, Verify, Steps) with `json:metadata` code fence at the end containing files, verifyCommand, acceptanceCriteria
   - activeForm: Present tense action (e.g., "Implementing X")
3. **CRITICAL - Dependencies:** For EACH task that has blockedBy in the plan or .tasks.json:
   - Call `TaskUpdate` with `taskId` and `addBlockedBy: [list-of-blocking-task-ids]`
   - Do NOT skip this step - dependencies are essential for correct execution order
4. Call `TaskList` and verify blockedBy relationships show correctly (e.g., "blocked by #1, #2")


### Step 2: Execute Batch
**Default: First 3 tasks**

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. **Use metadata for verification:** Parse the `json:metadata` code fence from the task description. Run `verifyCommand` and check each `acceptanceCriteria` before marking complete.
4. Mark as completed
5. Update task checkbox: `### [ ] Task N` → `### [x] Task N`
6. Update "Current Status" section with progress and next action

### Step 3: Report
When batch complete:
- Show what was implemented
- Show verification output
- Say: "Ready for feedback."

**On pause or session end:**
- Update "Current Status" with current progress and next action
- This enables seamless resume in future sessions

### Step 4: Continue
Based on feedback:
- Apply changes if needed
- Execute next batch
- Repeat until complete

### Step 5: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers-extended-cc:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker mid-batch (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

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
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **superpowers-extended-cc:using-git-worktrees** - REQUIRED: Set up isolated workspace before starting
- **superpowers-extended-cc:writing-plans** - Creates the plan this skill executes
- **superpowers-extended-cc:finishing-a-development-branch** - Complete development after all tasks
