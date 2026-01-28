# Sequential Thinking MCP Server

Sequential Thinking provides structured reasoning support for complex problems.

## What It Does

- Breaks down complex problems into steps
- Maintains reasoning chains
- Helps with multi-step analysis
- Supports backtracking and revision

## Installation

This is an official Anthropic MCP server:

```bash
# Test it works
npx -y @anthropics/mcp-server-sequential-thinking
```

## Configuration

Add to `~/.claude.json`:

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropics/mcp-server-sequential-thinking"]
    }
  }
}
```

## When It's Used

Claude automatically uses sequential thinking for:
- Complex debugging scenarios
- Multi-step planning
- Architectural decisions
- Problem decomposition

## Verification

Start Claude Code and ask:
> "Think through how to refactor this function step by step"

You should see structured reasoning in the response.
