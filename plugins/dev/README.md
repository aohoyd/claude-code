# Dev Plugin

Unified development workflow that combines collaborative brainstorming with deep codebase understanding, architecture design, TDD implementation, and automated quality review.

## Flow

**Idea -> Brainstorm -> Explore -> Design -> Implement -> Review**

The `/dev` command auto-chains all 5 phases with confirmation gates between each:

1. **Brainstorm** — Collaborative Q&A enhanced with parallel code-explorer agents to understand the codebase while shaping the idea. Produces a design document.
2. **Explore** — 2-3 parallel code-explorer agents deeply analyze relevant code areas, tracing execution paths and mapping patterns.
3. **Design** — 2-3 parallel code-architect agents propose different architecture approaches. User picks one, then an implementation plan with TDD tasks is generated.
4. **Implement** — Sequential task-executor subagents execute each plan task following TDD workflow (test first, implement, verify). A quick code review runs after each task and auto-fixes issues.
5. **Review** — Comprehensive final pass: 3 parallel code-reviewer agents check simplicity, correctness, and conventions. Issues are auto-fixed; ambiguous fixes are surfaced to the user.

## Components

### Command

- **`/dev`** — Main entry point. Accepts an optional feature description as argument.

### Skills (independently triggerable)

- **brainstorm** — "brainstorm with me", "help me design", "let's think through"
- **implementation-plan** — "create implementation plan", "make a plan", "break down into tasks"
- **execute-plan** — "execute plan", "run the plan", "implement the plan"
- **code-review** — "review my code", "check this code", "review changes"

### Agents

- **code-explorer** (yellow) — Traces execution paths, maps architecture layers
- **code-architect** (green) — Designs implementation blueprints
- **code-reviewer** (red) — Confidence-scored quality checks
- **task-executor** (green) — Executes individual TDD plan tasks

## Usage

```
/dev Add user authentication with OAuth2
```

Or trigger individual skills:
- "Let's brainstorm a caching strategy"
- "Create an implementation plan from the design doc"
- "Execute the plan in docs/plans/"

## Output Files

Design documents and implementation plans are saved to:
- `docs/plans/YYYY-MM-DD-<topic>-design.md`
- `docs/plans/YYYY-MM-DD-<topic>-plan.md`

## When to Use

- New features touching multiple files
- Features requiring architectural decisions
- Complex integrations with unclear requirements
- Any idea that benefits from structured thinking before coding
