# Memory Keeper MCP Server

Memory Keeper provides persistent context storage across Claude Code sessions. Without it, Claude starts fresh each conversation.

## Why Memory Keeper?

- **Remember decisions** - No re-explaining your preferences
- **Track progress** - Know what was done in previous sessions
- **Store contacts** - Keep Slack IDs, preferences, working styles
- **Save patterns** - Build a knowledge base of reusable solutions

## Two Stores: Short-Term and Long-Term

| Store | Purpose | Retention |
|-------|---------|-----------|
| Short-term | Active work, recent decisions | Session + days |
| Long-term | Permanent knowledge, contacts | Forever |

The CLAUDE.md file includes rules for when to promote items from short to long-term.

## Installation Options

### Option 1: Use an Existing MCP Server

Several community memory servers exist:
- Search npm for `mcp-server-memory`
- Search PyPI for `mcp-memory`

### Option 2: Build Your Own

Memory Keeper needs:
1. A SQLite database (or similar) for storage
2. MCP protocol implementation
3. Basic CRUD operations for context items

### Basic Schema

```sql
CREATE TABLE context_items (
  id INTEGER PRIMARY KEY,
  key TEXT UNIQUE,
  value TEXT,
  category TEXT,
  priority TEXT,
  channel TEXT,
  created_at DATETIME,
  updated_at DATETIME
);
```

### Required Operations

| Operation | Description |
|-----------|-------------|
| `context_save` | Store a new item |
| `context_get` | Retrieve items (with filters) |
| `context_search` | Search items by query |
| `context_status` | Get storage statistics |
| `context_batch_save` | Store multiple items |

## Configuration

Add to `~/.claude.json`:

```json
{
  "mcpServers": {
    "memory-keeper-short": {
      "type": "stdio",
      "command": "bash",
      "args": ["/path/to/memory-keeper-short/run.sh"]
    },
    "memory-keeper-long": {
      "type": "stdio",
      "command": "bash",
      "args": ["/path/to/memory-keeper-long/run.sh"]
    }
  }
}
```

## run.sh Template

Create a `run.sh` script that starts your memory server:

```bash
#!/bin/bash
cd "$(dirname "$0")"

# Set database path
export DB_PATH="./context.db"

# Start the server
# Replace with your actual server command
node index.js
# or: python server.py
# or: npx @your-package/memory-server
```

Make it executable:
```bash
chmod +x run.sh
```

## Usage in CLAUDE.md

The template CLAUDE.md includes memory protocols:

1. **Session Start** - Load recent context
2. **Auto-Save Triggers** - When to save automatically
3. **Promotion Rules** - When to move to long-term
4. **Channel Structure** - How to organize items

## Categories

| Category | Use For |
|----------|---------|
| `decision` | Choices made, approaches selected |
| `progress` | Work completed, milestones |
| `note` | General observations |
| `error` | Bugs, failures, issues |
| `warning` | Concerns, risks |
| `task` | To-dos, action items |

## Priorities

| Priority | Use For |
|----------|---------|
| `high` | Critical decisions, important fixes |
| `normal` | Regular work, standard items |
| `low` | Nice-to-know, minor details |

## Channels

Organize items by topic:

```
projects.my-app          # Project-specific context
tools.docker             # Tool configurations
people.contacts          # Contact information
workflows.deployment     # Reusable patterns
```

## Verification

Test your setup:

1. Start Claude Code
2. Ask Claude to save something to memory
3. Start a new session
4. Ask Claude to recall what was saved

If it remembers, you're set up correctly!
