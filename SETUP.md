# Claude Code Template Setup

**Instructions for Claude Code**: When a user asks you to "set up this template", "install this", or "configure Claude Code with this template", follow these steps.

---

## Step 1: Gather User Information

Use the `AskUserQuestion` tool to collect the following information:

### Questions to Ask
```
1. What's your name? (used for messages sent on your behalf)
2. What's your job title/role? (e.g., "Software Engineer", "Product Manager")
3. What timezone are you in?
   Options: America/New_York, America/Chicago, America/Denver, America/Los_Angeles, Europe/London, Europe/Paris, Asia/Tokyo, Australia/Sydney, Other
4. What city/area are you in? (optional - used for weather in morning briefing)
```

---

## Step 2: Create Directory Structure

Run these commands to set up the Claude Code directories:

```bash
mkdir -p ~/.claude/context/tasks
mkdir -p ~/.claude/commands
mkdir -p ~/.claude/skills
```

---

## Step 3: Copy and Personalize Files

For each file in this template, copy it to the appropriate location and replace placeholders with the user's information.

### Placeholder Mapping
| Placeholder | Replace With |
|-------------|--------------|
| `{{USER_NAME}}` | User's name |
| `{{ROLE}}` | User's job title |
| `{{TIMEZONE}}` | User's timezone |
| `{{LOCATION}}` | User's city/area (or "your location" if not provided) |

### Files to Copy

**Main instruction file:**
```
Source: ./CLAUDE.md
Destination: ~/.claude/CLAUDE.md
Action: Copy and replace all placeholders
```

**Context files:**
```
Source: ./context/writing-style.md → ~/.claude/context/writing-style.md
Source: ./context/current-session.md → ~/.claude/context/current-session.md
Source: ./context/tasks/communication.md → ~/.claude/context/tasks/communication.md
Source: ./context/tasks/git.md → ~/.claude/context/tasks/git.md
Source: ./context/tasks/calendar.md → ~/.claude/context/tasks/calendar.md
Action: Copy and replace all placeholders
```

**Commands:**
```
Source: ./commands/*.md → ~/.claude/commands/
Action: Copy all command files and replace placeholders
```

**Skills:**
```
Source: ./skills/* → ~/.claude/skills/
Action: Copy all skill directories (no placeholder replacement needed in most skills)
```

---

## Step 4: Verify Installation

After copying all files, verify the setup:

```bash
# Check files exist
ls -la ~/.claude/CLAUDE.md
ls -la ~/.claude/context/
ls -la ~/.claude/commands/
ls -la ~/.claude/skills/

# Count installed items
echo "Commands installed: $(ls ~/.claude/commands/*.md 2>/dev/null | wc -l)"
echo "Skills installed: $(ls -d ~/.claude/skills/*/ 2>/dev/null | wc -l)"
```

---

## Step 5: Welcome Message

After setup is complete, display:

```
✅ Claude Code Template Installed!

Your setup:
- Name: [their name]
- Role: [their role]
- Timezone: [their timezone]

Installed:
- 6 slash commands (/day-over, /standup, /morning-briefing, /week-review, /memory-review, /project-status)
- 17 skills (frontend, backend, design, writing, and more)
- Context files for communication, git, and calendar workflows

Try it out:
- Ask me to help with any coding task - relevant skills will activate automatically
- Type /help to see available commands

Optional next steps:
- Set up Memory Keeper MCP for persistent memory across sessions (see mcp-servers/ folder)
- Customize ~/.claude/context/writing-style.md to match your communication style
- Add your own commands in ~/.claude/commands/
```

---

## Troubleshooting

### If files already exist
Ask the user:
```
I found existing Claude Code configuration files. How would you like to proceed?

Options:
1. Backup existing and replace (Recommended)
2. Merge with existing (keep both)
3. Skip existing files (only add new ones)
4. Cancel setup
```

### If permissions error
```bash
chmod -R u+rw ~/.claude/
```

---

## Quick Setup Command

If the user just says "set this up" or "install this", run through all steps automatically, asking questions as needed. Aim for a smooth, conversational setup experience.
