# jj-workflow

A Claude Code plugin that streamlines Jujutsu (jj) version control workflow with simple commands.

## Features

- **`/jj-workflow:commit [message]`** - Describe the current changeset. Auto-generates a commit message from the diff if no message provided.
- **`/jj-workflow:new`** - Create a new empty changeset on top of the current one.
- **`/jj-workflow:squash [--into REV]`** - Squash the current changeset into its parent (or a specified revision).
- **`/jj-workflow:split`** - Interactively split the current changeset into multiple changesets.

## Installation

Copy the plugin directory to your Claude Code plugins location:

```bash
# For local testing
claude --plugin-dir /path/to/jj-workflow

# Or copy to your plugins directory
cp -r jj-workflow ~/.claude/plugins/
```

## Prerequisites

- [Jujutsu (jj)](https://github.com/martinvonz/jj) must be installed and available in your PATH
- You must be in a jj repository (a directory with `.jj/`)

## Usage Examples

```bash
# Commit with auto-generated message
/jj-workflow:commit

# Commit with custom message
/jj-workflow:commit "Add user authentication feature"

# Create new changeset
/jj-workflow:new

# Squash into parent
/jj-workflow:squash

# Squash into specific revision
/jj-workflow:squash --into abc123

# Split current changeset
/jj-workflow:split
```
