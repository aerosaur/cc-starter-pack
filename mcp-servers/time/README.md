# Time MCP Server

Time server provides timezone-aware operations for Claude Code.

## What It Does

- Get current time in any timezone
- Convert between timezones
- Handle date/time calculations
- Support scheduling operations

## Installation

This server uses uvx (Python):

```bash
# Install uvx if needed
pip install uvx

# Test it works
uvx mcp-server-time
```

## Configuration

Add to `~/.claude.json`:

```json
{
  "mcpServers": {
    "time": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-server-time"]
    }
  }
}
```

## When It's Used

Claude uses the time server for:
- Calendar operations (checking current time)
- Scheduling events (timezone conversion)
- Date calculations
- Session timestamps

## Available Operations

| Operation | Description |
|-----------|-------------|
| `get_current_time` | Get current time in specified timezone |
| `convert_time` | Convert time between timezones |

## Timezone Identifiers

Common timezone identifiers:

| Location | Identifier |
|----------|------------|
| US Pacific | `America/Los_Angeles` |
| US Eastern | `America/New_York` |
| UK | `Europe/London` |
| Central Europe | `Europe/Paris` |
| Japan | `Asia/Tokyo` |
| Australia Sydney | `Australia/Sydney` |

## Verification

Start Claude Code and ask:
> "What time is it in Tokyo?"

You should get the current time in Tokyo timezone.
