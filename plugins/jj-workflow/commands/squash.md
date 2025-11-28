---
description: Squash the current jj changeset into its parent or a specified revision
argument-hint: [--into REV]
allowed-tools: Bash(jj:*)
---

Squash the current changeset.

First, show the current state with `jj log -r @::` so the user can see what will be squashed.

Then squash:
- If $ARGUMENTS contains "--into", extract the revision and run `jj squash --into <revision>`
- Otherwise, squash into the parent: run `jj squash`

After squashing, run `jj log -r @::` to show the result.

Note: Squashing combines the current changeset's changes into another changeset. By default, it squashes into the parent. Use `--into REV` to squash into a different revision.
