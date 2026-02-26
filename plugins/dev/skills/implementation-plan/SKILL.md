---
name: Implementation Plan
description: This skill should be used when the user asks to "create implementation plan", "make a plan", "break down into tasks", "create actionable steps", or after the brainstorm skill completes a design. Converts design documents into detailed, TDD-style implementation plans with executable tasks.
---

# Implementation Plan Skill

Convert design documents into detailed, actionable implementation plans with TDD-style task structure. Each task includes specific files, tests, and verification steps.

## Overview

This skill takes a design document and produces:
1. A structured implementation plan with header metadata
2. Medium-granularity tasks (component-level, not line-by-line)
3. TDD workflow for each task (test first, implement, verify)
4. A plan file ready for execution by the execute-plan skill

## Input Requirements

Before creating a plan, locate the design document:

1. If coming from brainstorm skill, use the design document just created
2. If triggered directly, ask user for the design document location
3. Read and understand the design thoroughly before planning

## Plan Structure

### Required Header

Every plan MUST start with this header:

```markdown
# [Feature Name] Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use execute-plan to implement this plan task-by-task.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

### Task Structure

Each task follows this TDD-style structure:

```markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Step 1: Write the failing test**

\`\`\`python
def test_specific_behavior():
    result = function(input)
    assert result == expected
\`\`\`

**Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

**Step 3: Write minimal implementation**

\`\`\`python
def function(input):
    return expected
\`\`\`

**Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS
```

## Planning Process

### Step 1: Analyze Design

Read the design document and identify:

- **Components**: What distinct pieces need to be built?
- **Dependencies**: What order should things be built in?
- **Interfaces**: How do components connect?
- **Existing code**: What files need modification vs creation?

### Step 2: Determine Task Granularity

Use medium granularity (component-level):

**Too coarse:**
- "Implement authentication module" (too vague)

**Just right:**
- "Create login form component"
- "Add validation logic for email/password"
- "Implement session management"
- "Add logout functionality"

**Too fine:**
- "Create LoginForm.tsx file" (too atomic)
- "Add email input field" (too atomic)
- "Add password input field" (too atomic)

### Step 3: Order Tasks by Dependencies

Arrange tasks so each builds on previous ones:

1. Core utilities and shared code first
2. Data models and schemas
3. Business logic
4. API endpoints or interfaces
5. UI components
6. Integration and glue code
7. Final polish and edge cases

### Step 4: Write Each Task

For each task:

1. **Name clearly**: What component this creates or modifies
2. **List files**: Exact paths for create, modify, and test files
3. **Write test first**: Show the actual test code
4. **Specify run command**: Exact command to run the test
5. **Write implementation**: Show the minimal code
6. **Verify command**: Exact command to verify success

### Step 5: Review for Completeness

Verify the plan covers:

- All components from the design
- All edge cases mentioned in design
- Necessary refactoring of existing code
- Integration between components
- Final verification that everything works together

## Writing the Plan File

Save the plan to:

```
docs/plans/YYYY-MM-DD-<topic>-plan.md
```

Where:
- `YYYY-MM-DD` is today's date
- `<topic>` matches the design document topic

Create the directory if it doesn't exist.

## Handoff to Execution

After writing the plan file, explicitly ask user:

> "The implementation plan is complete and saved to `docs/plans/YYYY-MM-DD-<topic>-plan.md`.
>
> The plan contains N tasks that will be executed sequentially, each with TDD steps.
>
> **Would you like to hand off execution to the execute-plan skill with subagents?**
>
> This will execute each task using a fresh subagent, following the TDD workflow in the plan."

Wait for explicit user confirmation before triggering execute-plan.

## Key Principles

### Executable Tasks

Every task must be executable without ambiguity. Include:
- Exact file paths
- Actual code snippets
- Specific commands to run
- Expected outputs

### TDD Workflow

Every task follows test-first development:
1. Write test that defines expected behavior
2. Verify test fails (proves test is meaningful)
3. Write minimal implementation
4. Verify test passes

### Medium Granularity

Each task should:
- Take 5-30 minutes to complete
- Focus on one logical component
- Be independently verifiable
- Build on previous tasks

### Clear Dependencies

Make dependencies explicit in task ordering. If Task 3 requires Task 2's output, make that clear.
