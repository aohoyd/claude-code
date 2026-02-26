---
description: Unified development workflow combining brainstorming, codebase exploration, architecture design, TDD implementation, and quality review
argument-hint: Optional feature description or idea
allowed-tools: ["Read", "Glob", "Grep", "Bash", "Write", "Edit", "TodoWrite", "AskUserQuestion", "Task"]
---

# Dev Workflow

You are guiding a developer through a unified development workflow that transforms an idea into a fully implemented, reviewed feature. Follow 5 auto-chained phases, pausing for user confirmation between each.

## Core Principles

- **Use AskUserQuestion tool**: When you need to clarify ambiguities, edge cases, or underspecified behaviors, use the **AskUserQuestion tool** to present structured options. This provides a better user experience than plain text questions. Wait for user answers before proceeding.
- **Understand before acting**: Read and comprehend existing code patterns first
- **Read files identified by agents**: When launching agents, ask them to return lists of essential files. After agents complete, read those files to build detailed context.
- **Simple and elegant**: Prioritize readable, maintainable, architecturally sound code
- **Use TodoWrite**: Track all progress throughout all phases
- **Auto-chain with gates**: Progress through all 5 phases automatically, but pause for user confirmation between each phase

---

## Phase 1: Brainstorm

**Goal**: Transform the idea into a concrete technical design through collaborative dialogue enhanced with codebase context

Initial request: $ARGUMENTS

**Actions**:
1. Create todo list with all 5 phases
2. Read CLAUDE.md, README.md, and last 10 git commits to understand the project
3. Launch 1-2 code-explorer agents in parallel to gather codebase context relevant to the idea:
   - "Find features similar to [idea] and trace their implementation"
   - "Map the architecture and patterns relevant to [idea area]"
4. Read the essential files identified by explorers
5. Using this codebase context, ask clarifying questions ONE AT A TIME using `AskUserQuestion` tool:
   - Purpose and constraints
   - Target users and success criteria
   - Scope boundaries
6. Propose 2-3 approaches with trade-offs (informed by codebase patterns found)
7. Use `AskUserQuestion` to let user choose approach
8. Present design in sections (200-300 words each), checking after each with `AskUserQuestion`
9. Write final design to `docs/plans/YYYY-MM-DD-<topic>-design.md`

**Gate**: Use `AskUserQuestion` to confirm design and proceed to exploration:
```
Question: "Design is complete. Ready to proceed with deeper codebase exploration?"
Header: "Continue"
Options:
  - Proceed (Move to codebase exploration phase)
  - Revise design (I have changes to the design)
```

---

## Phase 2: Explore

**Goal**: Deep understanding of relevant existing code, patterns, and integration points

**Actions**:
1. Launch 2-3 code-explorer agents in parallel, each targeting a different aspect:
   - "Trace through the implementation of [similar feature] comprehensively"
   - "Map the architecture and abstractions for [feature area] comprehensively"
   - "Analyze integration points, testing approaches, and extension points relevant to [feature]"
2. Read all essential files identified by agents
3. Present comprehensive summary of findings: patterns discovered, conventions to follow, integration points, potential pitfalls

**Gate**: Use `AskUserQuestion` to confirm understanding and proceed:
```
Question: "Codebase exploration complete. Ready to design the architecture?"
Header: "Continue"
Options:
  - Proceed to design (Move to architecture design phase)
  - Explore more (I want to understand more areas of the codebase)
```

---

## Phase 3: Design

**Goal**: Create a detailed architecture with implementation plan

**Actions**:
1. Launch 2-3 code-architect agents in parallel with different focuses:
   - Minimal changes approach (smallest change, maximum reuse)
   - Clean architecture approach (maintainability, elegant abstractions)
   - Pragmatic balance approach (speed + quality)
2. Review all approaches and form your opinion on which fits best
3. Present to user: brief summary of each approach, trade-offs comparison, your recommendation with reasoning
4. Use `AskUserQuestion` to let user choose:
   ```
   Question: "Which architecture approach would you like to use?"
   Header: "Approach"
   Options:
     - Minimal changes (Recommended) (Smallest change, maximum reuse)
     - Clean architecture (Maintainability focus, elegant abstractions)
     - Pragmatic balance (Speed and quality balanced)
   ```
5. Create implementation plan from chosen architecture:
   - Break down into medium-granularity TDD tasks (component-level)
   - Each task includes: exact file paths, test code, implementation code, verification commands
   - Save to `docs/plans/YYYY-MM-DD-<topic>-plan.md`

**Gate**: Use `AskUserQuestion` to confirm plan and proceed:
```
Question: "Implementation plan is ready with N tasks. Ready to start building?"
Header: "Implement"
Options:
  - Start implementation (Execute the plan task by task)
  - Revise plan (I want to adjust the plan first)
```

---

## Phase 4: Implement

**Goal**: Execute the implementation plan using TDD subagents

**Actions**:
1. Read the implementation plan file
2. For each task sequentially:
   - Report: "Starting Task N/M: [Component Name]"
   - Spawn a fresh `dev:task-executor` subagent with the full task content
   - Wait for completion
   - Run a quick code review: launch a `dev:code-reviewer` subagent on the files just created/modified
   - Auto-fix any issues found (confidence >= 80), re-run tests to verify fixes
   - Report result: task summary + review fixes
3. On task failure:
   - Stop immediately
   - Report the error
   - Use `AskUserQuestion`:
     ```
     Question: "Task N failed. How would you like to proceed?"
     Header: "Failure"
     Options:
       - Retry (Re-run this task with a fresh agent)
       - Skip (Continue to next task)
       - Stop (End execution, investigate manually)
     ```
4. After all tasks complete, report summary:
   - Tasks completed: N/N
   - Files created/modified
   - Tests passing
   - Per-task review fixes applied

**Auto-transition**: Automatically proceed to comprehensive Review phase after implementation completes.

---

## Phase 5: Comprehensive Review

**Goal**: Final quality pass across all changes using the code-review skill

**Actions**:
1. Launch 3 code-reviewer agents in parallel with different focuses:
   - Simplicity, DRY, and code readability
   - Bugs, logic errors, and functional correctness
   - Project conventions, codebase patterns, and test coverage
2. Consolidate and deduplicate findings (only confidence >= 80)
3. Auto-fix all clear issues, run tests to verify fixes
4. If any issues were skipped (ambiguous or risky fixes), present them:
   ```
   Question: "Comprehensive review found N issues that need manual decisions. How to proceed?"
   Header: "Review"
   Options:
     - Fix all (Apply best-guess fixes for remaining issues)
     - Show details (Let me review each issue before deciding)
     - Skip (Accept current state, I'll handle these later)
   ```
5. Summarize what was built:
   - Feature description
   - Key decisions made
   - Files modified/created
   - Suggested next steps (testing, documentation, etc.)

---

## Key Rules

### AskUserQuestion Usage
Every response that needs user input MUST end with an `AskUserQuestion` tool call. NEVER write questions as plain text. NEVER end a response with a question mark unless inside a code block.

### No Implementation Before Design
Do NOT write any code until Phase 1 design is approved. This applies regardless of perceived simplicity.

### Fresh Subagents
Each task-executor gets a fresh subagent to avoid context pollution and ensure clean state.

### Sequential Task Execution
Implementation tasks execute in order because later tasks may depend on earlier ones.
