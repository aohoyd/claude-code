---
name: brainstorming
description: "This skill should be used when the user asks to 'brainstorm with me', 'help me design', 'let's think through', 'design a feature', 'help me figure out', 'help me plan', 'what's the best way to', 'help me architect', or wants to collaboratively develop an idea into a concrete design. Guides structured dialogue to transform vague ideas into fully-formed technical designs before implementation."
allowed-tools: ["Read", "Glob", "Grep", "Bash", "Write", "TodoWrite", "AskUserQuestion", "Task"]
---

<CRITICAL-RULE>
EVERY response that needs user input MUST end with an `AskUserQuestion` tool call. NEVER write questions as plain text. NEVER end a response with a question mark unless it is inside a code block. If you catch yourself writing a question in your response text, STOP and use the AskUserQuestion tool instead.

This is the MOST IMPORTANT rule in this entire skill. It overrides your default behavior of asking questions conversationally. You MUST use the tool.
</CRITICAL-RULE>

# Brainstorming Ideas Into Designs

Help turn ideas into fully formed designs and specs through structured collaborative dialogue, informed by codebase context gathered via parallel Task tool subagents. Ask questions one at a time using the AskUserQuestion tool, refine the idea, then present the design and get user approval.

<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it. This applies to EVERY project regardless of perceived simplicity.
</HARD-GATE>

## Anti-Pattern: Asking Questions As Plain Text

This is the single most common mistake. Do NOT do this:

**WRONG — question in response text:**
> "What database would you prefer? PostgreSQL, MySQL, or SQLite? Let me know your thinking."

**RIGHT — AskUserQuestion tool call after brief context:**
> "Based on the project structure, here are the database options to consider."
> *(then immediately call AskUserQuestion tool)*

```
AskUserQuestion({
  questions: [{
    question: "Which database should this project use?",
    header: "Database",
    options: [
      { label: "PostgreSQL (Recommended)", description: "Best for production workloads with complex queries" },
      { label: "SQLite", description: "Lightweight, no server needed, good for prototypes" },
      { label: "MySQL", description: "Widely deployed, good ecosystem support" }
    ],
    multiSelect: false
  }]
})
```

## Process

You MUST create a task for each of these steps and complete them in order:

### Step 1: Explore project context

Read CLAUDE.md, README.md, and last 10 git commits. Then use the Task tool to launch 1-2 code-explorer subagents (subagent_type: "dev:code-explorer") in parallel to gather codebase context relevant to the idea. Example prompts:
- "Find features similar to [idea] and trace their implementation"
- "Map the architecture and patterns relevant to [idea area]"

Read the essential files identified by the subagents. Briefly summarize findings (3-5 sentences, no questions) then proceed directly to Step 2.

### Step 2: Ask clarifying questions (one at a time)

Ask ONE question per response using the `AskUserQuestion` tool. Wait for the answer. Then ask the next question in a new response. Typically ask 3-6 clarifying questions. Fewer if the idea is already well-defined; more if it is vague or has many design dimensions.

Leverage codebase findings to ask informed questions. For example, if subagents found an existing pattern, ask whether to follow or deviate from it.

Your response should follow this pattern:
1. Brief context or observation (1-3 sentences of plain text)
2. End with an `AskUserQuestion` tool call — exactly ONE question with 2-4 options

Example response for this step:
> "I see the current key system uses a flat map. There are a few directions we could take the prefix system."
> *(then call AskUserQuestion)*

```
AskUserQuestion({
  questions: [{
    question: "Should prefixes support deep nesting (e.g., ga... for go-to-apps subgroup)?",
    header: "Nesting",
    options: [
      { label: "Single level only (Recommended)", description: "Just g... v... etc. Simpler to implement and use" },
      { label: "Multi-level nesting", description: "Allow ga... gad... etc. More flexible but more complex" }
    ],
    multiSelect: false
  }]
})
```

### Step 3: Design architecture with parallel architect agents

Use the Task tool to launch 2-3 code-architect subagents (subagent_type: "dev:code-architect") in parallel, each with a different design focus. Pass each agent the idea description, the clarified requirements from Step 2, and the codebase context from Step 1. Example prompts:
- "Design a minimal-changes approach for [feature]. Maximize reuse of existing code. Context: [codebase findings + requirements]"
- "Design a clean-architecture approach for [feature]. Focus on maintainability and elegant abstractions. Context: [codebase findings + requirements]"
- "Design a pragmatic-balance approach for [feature]. Balance speed and quality. Context: [codebase findings + requirements]"

After all agents return, consolidate their blueprints:
1. Summarize each approach (2-3 sentences each)
2. Compare trade-offs: complexity, maintainability, amount of change, risk
3. State your recommendation with reasoning
4. End with an `AskUserQuestion` tool call to let user choose

```
AskUserQuestion({
  questions: [{
    question: "Which architecture approach should we go with?",
    header: "Approach",
    options: [
      { label: "Minimal changes (Recommended)", description: "Smallest change, maximum reuse of existing code" },
      { label: "Clean architecture", description: "Maintainability focus, elegant abstractions" },
      { label: "Pragmatic balance", description: "Speed and quality balanced" }
    ],
    multiSelect: false
  }]
})
```

### Step 4: Present design in sections

Present one section at a time (200-300 words max). After each section, use `AskUserQuestion` to get approval before proceeding.

```
AskUserQuestion({
  questions: [{
    question: "Does the architecture section look right?",
    header: "Architecture",
    options: [
      { label: "Looks good", description: "Proceed to next section" },
      { label: "Needs changes", description: "I have feedback on this section" }
    ],
    multiSelect: false
  }]
})
```

Cover: architecture, components, data flow, error handling, testing.

### Step 5: Write design doc

Save to `docs/plans/YYYY-MM-DD-<topic>-design.md` and commit.

### Step 6: Transition to implementation

Invoke the implementation-plan (dev:implementation-plan) skill. Do NOT invoke any other skill.

## Key Principles

- **YAGNI ruthlessly** — Remove unnecessary features from all designs
- **Codebase-informed** — Use subagent findings to propose patterns consistent with the existing codebase
- **Explore alternatives** — Always propose 2-3 approaches before settling
- **Incremental validation** — Present design section by section, get approval via AskUserQuestion before moving on

<SELF-CHECK>
Before generating EVERY response, verify:
1. Does my response contain a question for the user? → I MUST use AskUserQuestion tool, not plain text
2. Am I asking multiple questions? → I MUST ask only ONE question per response
3. Does my response end with text asking the user to "let me know" or similar? → WRONG. End with AskUserQuestion tool call instead.

If ANY of these checks fail, rewrite the response before sending it.
</SELF-CHECK>
