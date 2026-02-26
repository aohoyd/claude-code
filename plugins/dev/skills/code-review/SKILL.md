---
name: Code Review
description: "This skill should be used when the user asks to 'review my code', 'check this code', 'review changes', 'code review', 'review what I wrote', 'check for bugs', or wants quality feedback on recent code changes. Launches parallel code-reviewer agents with different focuses and reports findings."
allowed-tools: ["Read", "Glob", "Grep", "Bash", "TodoWrite", "AskUserQuestion", "Task"]
---

# Code Review

Launch parallel code-reviewer subagents to review code changes, consolidate findings, and report issues. This skill only identifies and reports problems — it does NOT fix them. The caller (execute-plan, dev command, or user) decides what to do with the results.

Supports two modes: quick review (single focus, used after individual tasks) and comprehensive review (multiple focuses, used after full implementation).

## Quick Review Mode

Used by execute-plan after each task. Fast, focused on the files just changed.

### Process

1. Launch a single code-reviewer subagent (subagent_type: "dev:code-reviewer") with prompt:
   ```
   Review only these specific files for bugs, logic errors, and project convention violations:
   [list of files created/modified by the task]

   Context: These files were just created/modified as part of implementing [task description].
   Focus on correctness and convention adherence. Only report issues with confidence >= 80.
   ```
2. Return the reviewer's findings as-is

### Output Format

```
Review (Task N): [Component Name]
- Issue: [description] in file.py:42 (confidence: 90)
- Issue: [description] in file.py:78 (confidence: 85)
- Clean: no issues in test_file.py
```

## Comprehensive Review Mode

Used after all tasks complete or when triggered independently. Thorough, multi-focus.

### Process

1. Launch 3 code-reviewer subagents (subagent_type: "dev:code-reviewer") in parallel, each with a different focus:
   - **Simplicity & DRY**: "Review these files for code duplication, unnecessary complexity, and opportunities to simplify. Check that abstractions are justified and code is readable."
   - **Bugs & Correctness**: "Review these files for logic errors, null/undefined handling, race conditions, security vulnerabilities, and edge cases that could cause failures."
   - **Conventions & Patterns**: "Review these files for adherence to project conventions in CLAUDE.md, consistent patterns with the rest of the codebase, proper error handling, and test coverage."

   Pass each agent the full list of files created/modified during execution.

2. Consolidate findings from all 3 agents:
   - Deduplicate issues reported by multiple agents
   - Sort by severity (Critical first, then Important)
   - Filter: only keep issues with confidence >= 80

3. Report consolidated results:
   ```
   Comprehensive Review Complete

   Issues found: N (X critical, Y important)

   Critical:
   - [description] in file.py:42 (confidence: 95) — suggested fix: [fix]
   - [description] in file.py:78 (confidence: 90) — suggested fix: [fix]

   Important:
   - [description] in file.py:100 (confidence: 85) — suggested fix: [fix]
   ```

4. If no issues found, report: "Comprehensive review complete: code looks good."

## Standalone Trigger

When triggered independently (not from execute-plan), determine what to review:

1. Check for unstaged changes via `git diff`
2. If changes exist, review those files
3. If no changes, ask user what to review using `AskUserQuestion`
4. Default to comprehensive review mode

## Key Principles

- **Report only, never fix** — This skill identifies issues and suggests fixes but never applies them
- **Quality over quantity** — Only surface issues with confidence >= 80
- **Deduplicate** — When multiple agents report the same issue, consolidate into one
- **Actionable output** — Every issue includes file:line, confidence score, and a concrete fix suggestion
