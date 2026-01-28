# MCP Server Setup

This directory contains setup guides for MCP (Model Context Protocol) servers that enhance Claude Code's capabilities.

## What are MCP Servers?

MCP servers extend Claude Code's abilities by providing:
- **Persistent memory** across sessions
- **Tool access** (calendar, Slack, files, etc.)
- **Custom integrations** with your workflow

## Core Servers (Recommended)

These servers work out of the box with minimal configuration:

| Server | Purpose | Difficulty |
|--------|---------|------------|
| Memory Keeper | Persistent context across sessions | Medium |
| Sequential Thinking | Complex reasoning support | Easy |
| Time | Timezone-aware operations | Easy |
| Filesystem | Safe file access | Easy |

## Optional Servers (Bring Your Own)

These require credentials or additional setup:

| Server | Purpose | Requirements |
|--------|---------|--------------|
| Slack | Team communication | Workspace token |
| Google Calendar | Calendar operations | OAuth setup |
| GitHub | Repository access | Personal access token |
| Obsidian | Note-taking | Local REST API plugin |

## Quick Start

### 1. Install Prerequisites

Most MCP servers need Node.js and/or Python:

```bash
# Node.js (for npx servers)
node --version  # Should be v18+

# Python (for uvx servers)
python3 --version  # Should be 3.8+
pip install uvx
```

### 2. Configure .claude.json

Copy the example config:
```bash
cp .claude.json.example ~/.claude.json
```

Edit `~/.claude.json` to enable servers you want.

### 3. Set Up Individual Servers

Follow the README in each server's directory:
- `memory-keeper/README.md`
- `sequential-thinking/README.md`
- `time/README.md`
- `filesystem/README.md`

## Configuration Reference

MCP servers are configured in `~/.claude.json` under the `mcpServers` key:

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "command-to-run",
      "args": ["arg1", "arg2"],
      "env": {
        "ENV_VAR": "value"
      }
    }
  }
}
```

### Common Patterns

**NPX-based server:**
```json
{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@package/server-name"]
}
```

**UVX-based server:**
```json
{
  "type": "stdio",
  "command": "uvx",
  "args": ["package-name"]
}
```

**Bash script wrapper:**
```json
{
  "type": "stdio",
  "command": "bash",
  "args": ["/path/to/run.sh"]
}
```

## Troubleshooting

### Server Not Starting

1. Check the command exists: `which npx` or `which uvx`
2. Try running the command manually
3. Check logs in Claude Code

### Server Not Appearing

1. Restart Claude Code after config changes
2. Verify JSON syntax in `.claude.json`
3. Check server name matches expected pattern

### Permission Errors

1. Ensure scripts are executable: `chmod +x run.sh`
2. Check file paths are absolute
3. Verify environment variables are set

## Adding New Servers

Search for MCP servers:
- [Official Anthropic servers](https://github.com/anthropics)
- [Community servers on npm](https://www.npmjs.com/search?q=mcp-server)
- [Community servers on PyPI](https://pypi.org/search/?q=mcp-server)

Most servers follow standard patterns - check their documentation for specific setup instructions.
