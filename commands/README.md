# Commands

This directory contains custom slash commands for Claude Code.

## How Commands Work

Files in this directory become available as slash commands. For example, `day-over.md` becomes `/day-over`.

### File Structure

Each command file needs:
1. **Frontmatter** with a `description` field
2. **Instructions** for what the command should do

```markdown
---
description: Brief description shown in /help
---

Your detailed command instructions here.
```

## Available Commands

| Command | Description |
|---------|-------------|
| `/day-over` | Create end-of-day summary in notes |
| `/standup` | Generate standup update from recent work |
| `/morning-briefing` | Daily overview of calendar, messages, tasks |
| `/week-review` | Weekly summary from daily summaries |
| `/memory-review` | Review and manage memory keeper items |
| `/project-status` | Quick status check on active projects |

## Creating New Commands

1. Create a new `.md` file in this directory
2. Add frontmatter with description
3. Write clear instructions for Claude

### Example: Custom Command

```markdown
---
description: Check status of all my PRs
---

Check GitHub for all my open pull requests. Show:
- PR title and number
- Repository name
- Status (draft, ready for review, approved, changes requested)
- How long it's been open
- Any failing checks

Format as a table sorted by oldest first.
```

### Tips for Good Commands

1. **Be specific** - Tell Claude exactly what to do
2. **Include output format** - Specify how results should be presented
3. **Handle edge cases** - What if there's no data? What if a tool fails?
4. **Reference tools** - Mention specific MCP tools when relevant

## Command Arguments

Commands can accept arguments using the `argument-hint` field:

```markdown
---
description: Quick status check for active projects
argument-hint: [project-name]
---

If a project name is provided, show detailed status for that project.
Otherwise, show overview of all active projects.
```

This enables: `/project-status my-app`

## Customizing Existing Commands

Feel free to modify these commands to match your workflow:

- Change file paths to match your system
- Adjust output formats
- Add or remove sections
- Reference your specific tools and integrations
