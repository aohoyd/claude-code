---
name: Brainstorm
description: This skill should be used when the user asks to "brainstorm with me", "help me design", "let's think through", "design a feature", "help me figure out", or wants to collaboratively develop an idea into a concrete design. Guides structured dialogue to transform vague ideas into fully-formed technical designs.
---

# Brainstorm Skill

Transform ideas into fully-formed technical designs through structured collaborative dialogue. Guide users from initial concept to documented design through natural conversation.

## Overview

This skill facilitates a three-phase design process:
1. **Context & Understanding** - Gather project context and understand the idea
2. **Exploration** - Propose approaches and refine through Q&A
3. **Documentation** - Present design in sections and write to file

## Phase 1: Context & Understanding

### Gather Project Context

Before exploring the idea, gather context about the current project state:

1. Read CLAUDE.md if it exists (project conventions and guidelines)
2. Read README.md if it exists (project purpose and structure)
3. Review last 10 git commits to understand recent activity:
   ```bash
   git log --oneline -10
   ```

Summarize the project context briefly before proceeding.

### Understand the Idea

Ask questions one at a time to fully understand what the user wants to build. Focus on:

- **Problem**: What problem does this solve?
- **Users**: Who will use this and when?
- **Scope**: What's in and out of scope?
- **Constraints**: Any technical or business constraints?
- **Success criteria**: How will we know it works?

Continue asking until the idea is clear. Summarize understanding before moving to exploration.

## Phase 2: Exploration

### Propose Approaches

Present 2-3 different approaches to solving the problem. For each approach:

- **Name**: Give it a memorable name
- **Summary**: 1-2 sentences describing the approach
- **Pros**: Key advantages
- **Cons**: Key disadvantages
- **Recommendation**: When this approach is best suited

### Guide Decision

Do not simply wait for the user to pick. Ask clarifying questions to help them decide:

- "Do you prioritize simplicity over flexibility here?"
- "How important is extensibility for future features?"
- "What's your tolerance for complexity in the initial implementation?"
- "Are there existing patterns in the codebase we should follow?"

Continue the dialogue until the user feels confident about an approach.

## Phase 3: Documentation

### Present Design in Sections

Once an approach is chosen, present the complete design in small sections (200-300 words each). After each section, check:

> "Does this section look right so far?"

Wait for confirmation before proceeding to the next section.

### Section Structure

Present the design in this order:

1. **Problem Statement** - What we're solving and why
2. **Proposed Solution** - High-level description of the chosen approach
3. **Technical Approach** - Architecture, patterns, and key decisions
4. **Components** - Main pieces and their responsibilities
5. **Data Flow** - How data moves through the system
6. **Edge Cases** - Error handling and boundary conditions
7. **Open Questions** - Any remaining uncertainties

### Handle Objections

If the user says something looks wrong:

1. Ask what specifically needs changing
2. Revise just that section
3. Re-present the revised section for approval
4. Only proceed to next section after approval

### Write Design Document

After all sections are approved, write the complete design to:

```
docs/plans/YYYY-MM-DD-<topic>-design.md
```

Where:
- `YYYY-MM-DD` is today's date
- `<topic>` is a kebab-case summary of the feature (e.g., `user-authentication`, `payment-integration`)

Create the `docs/plans/` directory if it doesn't exist.

### Design Document Template

```markdown
# [Feature Name] Design

**Date:** YYYY-MM-DD
**Status:** Draft

## Problem Statement

[Content from section 1]

## Proposed Solution

[Content from section 2]

## Technical Approach

[Content from section 3]

## Components

[Content from section 4]

## Data Flow

[Content from section 5]

## Edge Cases

[Content from section 6]

## Open Questions

[Content from section 7]
```

## Handoff to Implementation Planning

After writing the design document, trigger the implementation-plan skill:

> "The design is complete and saved to `docs/plans/YYYY-MM-DD-<topic>-design.md`.
>
> Ready to create the implementation plan? This will break down the design into detailed, actionable tasks with a TDD approach."

Wait for user confirmation before proceeding with implementation planning.

## Key Principles

### One Question at a Time

Never overwhelm with multiple questions. Ask one question, wait for the answer, then follow up based on the response.

### Collaborative Not Prescriptive

Guide the user to their own conclusions rather than dictating solutions. Present options, ask clarifying questions, and let the user drive decisions.

### Check Understanding Frequently

Summarize understanding at key points:
- After gathering context
- After understanding the idea
- After selecting an approach
- After each design section

### Adapt to User Expertise

Gauge the user's technical level and adjust explanations accordingly. Don't over-explain to experts or under-explain to novices.
