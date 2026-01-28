# Personalization Guide

This guide is for manual customization. For automatic setup, just tell Claude Code: **"Set up this template for me"**

---

## Placeholders

The template uses these placeholders that get replaced during setup:

| Placeholder | What It Is | Example |
|-------------|------------|---------|
| `{{USER_NAME}}` | Your name | Sarah |
| `{{ROLE}}` | Your job title | Software Engineer |
| `{{TIMEZONE}}` | Your timezone | America/Los_Angeles |
| `{{LOCATION}}` | Your city (optional) | San Francisco, CA |

## Manual Setup

If you prefer to set things up yourself:

### 1. Copy Files

```bash
# Create directories
mkdir -p ~/.claude/context/tasks
mkdir -p ~/.claude/commands
mkdir -p ~/.claude/skills

# Copy main file
cp CLAUDE.md ~/.claude/CLAUDE.md

# Copy context files
cp context/*.md ~/.claude/context/
cp context/tasks/*.md ~/.claude/context/tasks/

# Copy commands
cp commands/*.md ~/.claude/commands/

# Copy skills
cp -r skills/* ~/.claude/skills/
```

### 2. Replace Placeholders

Edit `~/.claude/CLAUDE.md` and replace the placeholders in the Profile section:

```markdown
## Profile

- **Name**: `Sarah`
- **Role**: `Software Engineer`
- **Timezone**: `America/Los_Angeles`
- **Location**: `San Francisco, CA`
```

### 3. Customize Writing Style (Optional)

Edit `~/.claude/context/writing-style.md` to match how you communicate:

- Casual vs formal tone
- Use of contractions
- Sentence length preferences
- Platform-specific styles (Slack, email, etc.)

## Adding Integrations Later

The template works without any integrations. If you later want to add:

### Memory Keeper
Provides persistent memory across sessions. See `mcp-servers/memory-keeper/README.md`

### Google Calendar
For calendar-aware commands. Set up Google Calendar MCP server.

### Slack
For team communication. Set up Slack MCP server with your workspace token.

### GitHub
For code-aware commands. Set up GitHub MCP server with your personal access token.

Each integration enhances certain commands but none are required.

## File Locations

After setup:

| What | Location |
|------|----------|
| Main instructions | `~/.claude/CLAUDE.md` |
| Context files | `~/.claude/context/` |
| Commands | `~/.claude/commands/` |
| Skills | `~/.claude/skills/` |
| MCP config | `~/.claude.json` |
