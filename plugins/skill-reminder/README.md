# skill-reminder

A Claude Code plugin that reminds Claude to check for and activate relevant skills before implementing features.

## What it does

When you ask Claude to implement something (create a feature, build functionality, add capabilities), this plugin triggers a reminder to check for relevant skills first. This helps ensure that established patterns, best practices, and specialized workflows are considered before diving into implementation.

## When it triggers

The hook activates on prompts that indicate implementation work:
- "Implement user authentication"
- "Create a new API endpoint"
- "Build a dashboard component"
- "Add caching to the system"

It does NOT trigger on:
- Simple questions or explanations
- Bug fixes
- File reading/research
- Minor edits

## Installation

```bash
# Test locally
claude --plugin-dir /path/to/skill-reminder

# Or symlink to your plugins directory
ln -s /path/to/skill-reminder ~/.claude/plugins/skill-reminder
```

## How it works

Uses a `UserPromptSubmit` hook with a prompt-based approach. When implementation work is detected, Claude receives a system message reminding it to check available skills using `/skills` or the Skill tool.
