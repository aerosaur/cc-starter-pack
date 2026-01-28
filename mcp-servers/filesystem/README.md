# Filesystem MCP Server

Filesystem server provides safe file operations with configurable access boundaries.

## What It Does

- Read files from allowed directories
- Write files (with safety checks)
- List directory contents
- Search for files
- All operations respect configured boundaries

## Installation

This is an official Anthropic MCP server:

```bash
# Test it works
npx -y @anthropics/mcp-server-filesystem /path/to/allowed/directory
```

## Configuration

Add to `~/.claude.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@anthropics/mcp-server-filesystem",
        "/Users/yourname/Documents",
        "/Users/yourname/Projects"
      ]
    }
  }
}
```

**Important:** List ALL directories you want Claude to access. The server will only allow operations within these paths.

## Security

The filesystem server enforces boundaries:
- Only reads/writes within configured directories
- Prevents access to sensitive system files
- Logs all operations for audit

## Common Configurations

### Developer Setup
```json
"args": [
  "-y",
  "@anthropics/mcp-server-filesystem",
  "/Users/yourname/Projects",
  "/Users/yourname/.config"
]
```

### Writer Setup
```json
"args": [
  "-y",
  "@anthropics/mcp-server-filesystem",
  "/Users/yourname/Documents",
  "/Users/yourname/Notes"
]
```

### Full Home Directory (Less Secure)
```json
"args": [
  "-y",
  "@anthropics/mcp-server-filesystem",
  "/Users/yourname"
]
```

## Available Operations

| Operation | Description |
|-----------|-------------|
| `read_file` | Read file contents |
| `write_file` | Write to a file |
| `list_directory` | List directory contents |
| `search_files` | Search for files by pattern |
| `get_file_info` | Get file metadata |

## Verification

Start Claude Code and ask:
> "List the files in my Documents folder"

You should see the directory listing (if Documents is in your allowed paths).
