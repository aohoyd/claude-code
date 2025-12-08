# Brainstorm Plugin

A collaborative design workflow plugin for Claude Code that transforms ideas into fully-formed designs and implementation plans through natural dialogue.

## Features

- **Brainstorming**: Turn vague ideas into concrete designs through guided Q&A
- **Implementation Planning**: Convert designs into detailed, TDD-style implementation plans
- **Plan Execution**: Execute plans task-by-task using fresh subagents

## Workflow

The plugin provides a complete idea-to-implementation pipeline:

```
Idea → Brainstorm → Design Doc → Implementation Plan → Execution
```

Each stage builds on the previous, with explicit handoffs and user confirmation.

## Skills

### brainstorm
**Trigger with:** "brainstorm with me", "help me design", "let's think through", "design a feature"

Guides collaborative design through three phases:

1. **Context & Understanding**
   - Reads CLAUDE.md, README.md, last 10 git commits
   - Asks questions one at a time to understand your idea
   - Summarizes understanding before proceeding

2. **Exploration**
   - Proposes 2-3 different approaches
   - Asks clarifying questions to help you decide
   - Refines based on your feedback

3. **Documentation**
   - Presents design in 200-300 word sections
   - Checks after each section: "Does this look right?"
   - Revises sections based on feedback
   - Writes final design to `docs/plans/YYYY-MM-DD-<topic>-design.md`

### implementation-plan
**Trigger with:** "create implementation plan", "make a plan", "break down into tasks"

Converts designs into actionable TDD-style plans:

1. Analyzes the design document
2. Breaks down into medium-granularity tasks (component-level)
3. Creates task structure with:
   - Exact file paths to create/modify
   - Test code (written first)
   - Implementation code
   - Verification commands
4. Writes to `docs/plans/YYYY-MM-DD-<topic>-plan.md`
5. Asks for explicit confirmation before handoff to execution

### execute-plan
**Trigger with:** "execute plan", "run the plan", "implement the plan"

Orchestrates plan execution:

1. Parses the plan file
2. Spawns fresh task-executor subagent for each task
3. Executes tasks sequentially (TDD workflow)
4. Reports progress conversationally
5. On failure: stops and asks how to proceed (retry/skip/stop)

## Agent

### task-executor
A focused subagent that executes a single task from the plan:

- Follows TDD steps exactly (test first, then implement)
- Creates/modifies files at specified paths
- Runs verification commands
- Reports results or errors

## Installation

**Option 1: Plugin directory flag**
```bash
claude --plugin-dir /path/to/brainstorm
```

**Option 2: Copy to project**
```bash
cp -r brainstorm /your/project/.claude-plugin/
```

## Output Files

All designs and plans are written to `docs/plans/`:

| File | Purpose |
|------|---------|
| `YYYY-MM-DD-<topic>-design.md` | Design document with problem statement, solution, architecture |
| `YYYY-MM-DD-<topic>-plan.md` | Implementation plan with TDD-style tasks |

## Example Usage

```
You: brainstorm with me about adding user authentication

Claude: [Reads project context]
        Let me understand your idea. What type of authentication
        are you thinking - email/password, OAuth, or something else?

You: email/password with session management

Claude: [Asks more questions, proposes approaches, presents design sections]
        ...
        The design is complete. Ready to create the implementation plan?

You: yes

Claude: [Creates TDD-style implementation plan]
        ...
        Would you like to hand off execution to the execute-plan skill?

You: yes

Claude: [Spawns subagents to execute each task]
        Task 1/5 complete: User Model
        Task 2/5 complete: Password Hashing
        ...
```

## Plan Format

Implementation plans use this structure:

```markdown
# [Feature Name] Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use execute-plan to implement this plan task-by-task.

**Goal:** [One sentence describing what this builds]
**Architecture:** [2-3 sentences about approach]
**Tech Stack:** [Key technologies/libraries]

---

### Task 1: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Test: `tests/exact/path/to/test.py`

**Step 1: Write the failing test**
[test code]

**Step 2: Run test to verify it fails**
Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL

**Step 3: Write minimal implementation**
[implementation code]

**Step 4: Run test to verify it passes**
Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS
```
