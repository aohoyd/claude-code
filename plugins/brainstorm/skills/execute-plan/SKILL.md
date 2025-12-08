---
name: Execute Plan
description: This skill should be used when the user asks to "execute plan", "run the plan", "implement the plan", "start implementation", or after the implementation-plan skill creates a plan. Orchestrates sequential task execution using fresh subagents for each task.
---

# Execute Plan Skill

Orchestrate the execution of implementation plans by spawning fresh subagents for each task. Execute tasks sequentially, report progress conversationally, and handle failures gracefully.

## Overview

This skill:
1. Reads an implementation plan file
2. Executes each task sequentially using fresh task-executor subagents
3. Reports progress after each task
4. Stops and asks for guidance on any failure

## Input Requirements

### Locate the Plan File

Before execution, identify the plan to execute:

1. If coming from implementation-plan skill, use the plan just created
2. If triggered directly, ask user for the plan file location
3. Verify the plan has the required header format

### Validate Plan Format

Check that the plan starts with:

```markdown
# [Feature Name] Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use execute-plan to implement this plan task-by-task.

**Goal:** ...
**Architecture:** ...
**Tech Stack:** ...
```

If the format is incorrect, ask user to confirm this is the right file.

## Execution Process

### Step 1: Parse the Plan

Read the plan file and extract:

1. **Header metadata**: Goal, architecture, tech stack
2. **Task list**: All tasks in order (### Task N: ...)
3. **Task count**: Total number of tasks

Report to user:

> "Found N tasks in the implementation plan.
>
> **Goal:** [goal from header]
>
> Starting execution..."

### Step 2: Execute Tasks Sequentially

For each task in order:

1. **Report start**: "Starting Task N: [Component Name]"
2. **Spawn subagent**: Use the Task tool with task-executor agent
3. **Pass task content**: Include the full task markdown (files, steps, code)
4. **Wait for completion**: Let subagent complete or fail
5. **Report result**: Success or failure

### Step 3: Spawning Subagents

Use the Task tool to spawn a fresh task-executor agent for each task:

```
Task tool parameters:
- subagent_type: "brainstorm:task-executor"
- prompt: Include the full task content from the plan
- description: "Task N: [Component Name]"
```

The prompt to the subagent should include:

1. The task content (files, steps, code snippets)
2. Context about the overall goal
3. Instructions to follow the TDD steps exactly

### Step 4: Handle Results

**On success:**
- Report: "Task N complete: [brief summary]"
- Proceed to next task

**On failure:**
- Stop immediately
- Report: "Task N failed: [error details]"
- Ask user how to proceed:
  > "Task N encountered an error:
  > [error details]
  >
  > How would you like to proceed?
  > 1. Retry this task
  > 2. Skip and continue to next task
  > 3. Stop execution and investigate"

Wait for user decision before continuing.

## Progress Reporting

Report progress conversationally only (do not modify the plan file).

### After Each Task

```
Task N/M complete: [Component Name]
- Created: file1.py, file2.py
- Tests passing: 3/3
```

### After All Tasks

```
Execution complete!

Summary:
- Tasks completed: N/N
- Files created: X
- Files modified: Y
- Tests passing: Z

All tasks from the implementation plan have been executed.
```

## Error Handling

### Stop on Failure

When a task fails:

1. **Stop immediately** - Do not proceed to next task
2. **Report the error** - Show what went wrong
3. **Ask for guidance** - Let user decide next steps

### Failure Options

Present these options to the user:

1. **Retry**: Re-run the same task with a fresh subagent
2. **Skip**: Mark as skipped, continue to next task
3. **Stop**: End execution, let user investigate manually

### Partial Completion

If execution stops mid-way:

```
Execution paused at Task N/M.

Completed tasks: 1-[N-1]
Failed task: N
Remaining tasks: [N+1]-M

To resume, fix the issue and run execute-plan again.
```

## Subagent Instructions

When spawning task-executor subagents, include these instructions:

```
Execute this task from an implementation plan:

[Full task content from plan]

Instructions:
1. Follow the TDD steps exactly as written
2. Create files at the exact paths specified
3. Run tests with the exact commands given
4. Report any deviations from expected results
5. If a step fails, stop and report the error

Context:
- Goal: [goal from plan header]
- This is task N of M total tasks
- Previous tasks have been completed successfully
```

## Key Principles

### Fresh Subagent Per Task

Each task gets a fresh subagent to:
- Avoid context pollution between tasks
- Ensure clean state for each task
- Make failures isolated and recoverable

### Sequential Execution

Tasks execute in order because:
- Later tasks may depend on earlier ones
- TDD requires tests to exist before implementation
- Errors should be caught early

### Conversational Progress

Report progress in conversation to:
- Keep user informed
- Avoid modifying plan files
- Enable easy review of what happened

### Fail Fast

Stop on first failure to:
- Prevent cascading errors
- Give user control over recovery
- Avoid wasted work on dependent tasks
