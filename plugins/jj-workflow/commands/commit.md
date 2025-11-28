---
description: Describe the current jj changeset with a commit message
argument-hint: [optional message]
allowed-tools: Bash(jj:*, git:*)
---

Commit the current jj changeset.

First, check the current state:
1. Run `jj status` to see what changes are in the working copy
2. Run `jj diff` to see the actual changes

Then describe the changeset:
- If $ARGUMENTS is provided, use it as the commit message: run `jj describe -m "$ARGUMENTS"`
- If no message is provided, analyze the diff and generate an appropriate commit message, then run `jj describe -m "generated message"`

Commit message guidelines:
- Use imperative mood ("Add feature" not "Added feature")
- Keep the first line under 72 characters
- Be specific about what changed and why

After describing, run `jj log -r @` to confirm the changeset was updated.
