---
description: Split the current jj changeset into multiple changesets
allowed-tools: Bash(jj:*)
---

Split the current changeset into multiple smaller changesets.

First, show what's in the current changeset:
1. Run `jj status` to see all modified files
2. Run `jj diff --stat` to see a summary of changes

Then run `jj split` to interactively split the changeset.

Note: The split command opens an interactive interface where you can select which changes go into each new changeset. This is useful for breaking up large changes into logical, reviewable commits.

After splitting, run `jj log -r @::` to show the resulting changesets.
