---
description: Quick status check for active projects
argument-hint: [project-name]
---

Generate a quick status overview for active projects. If a project name is provided, show detailed status for that project. Otherwise, show overview of all active projects.

## 1. Identify Active Projects

Search notes folder for active project documentation:
- Use Glob tool with pattern `**/*.md` at `{{NOTES_PATH}}/Projects/`
- Look for project folders and README/index files
- Identify recently modified project files (last 30 days)

## 2. Gather Project Information

For each active project, collect:

### From Notes
- Project description and goals
- Current status/phase
- Recent updates (last 7-14 days)
- Open action items
- Blockers or issues
- Next steps

### From GitHub (if project has a repo)
- Search for related PRs: `mcp__github__search_pull_requests`
- Search for related issues
- Recent activity (commits, merged PRs)

### From Memory Keeper
- Search short-term memory for recent work on the project
- Use `mcp__memory-keeper-short__context_search` with project name
- Look for decisions, progress updates, blockers

## 3. Status Report Format

### For Single Project (if project name provided):

```markdown
# Project Status: [Project Name]

## Current Status
**Phase**: [Discovery/Design/Development/Testing/Launch/Maintenance]
**Health**: [On Track | At Risk | Blocked]
**Last Updated**: [Date]

## Overview
[2-3 sentence summary of the project and current state]

## Recent Activity (Last 7 Days)
- [Activity 1]
- [Activity 2]
- [Activity 3]

## Completed This Week
- [Completed item 1]
- [Completed item 2]

## In Progress
- [In progress item 1] - [% complete or status]
- [In progress item 2] - [% complete or status]

## Open Action Items
- [ ] [Action item 1] - [Owner/Due date if available]
- [ ] [Action item 2] - [Owner/Due date if available]

## Blockers
- [Blocker 1 with context]
- [Blocker 2 with context]

## GitHub Activity
- **Open PRs**: X ([links])
- **Open Issues**: Y ([links])
- **Recent Merges**: Z

## Next Steps
1. [Next immediate step]
2. [Following step]
3. [Future step]

## Risks & Concerns
- [Risk 1]
- [Risk 2]
```

### For All Projects Overview:

```markdown
# Active Projects Overview

## Summary
- **Total Projects**: X
- **On Track**: Y
- **At Risk**: Z
- **Blocked**: N

---

## [Project 1 Name]
**Status**: [On Track/At Risk/Blocked] [Phase]
**Recent**: [Last significant update]
**Next**: [Next immediate action]
**Blockers**: [Any blockers or "None"]

## [Project 2 Name]
**Status**: [On Track/At Risk/Blocked] [Phase]
**Recent**: [Last significant update]
**Next**: [Next immediate action]
**Blockers**: [Any blockers or "None"]

[Repeat for each project]

---

## Quick Actions Needed
1. [Urgent action for project X]
2. [Follow-up needed for project Y]
```

## 4. Status Indicators

Use consistent status indicators:
- **On Track**: Meeting milestones, no major blockers
- **At Risk**: Behind schedule, minor blockers, needs attention
- **Blocked**: Cannot proceed, critical blocker, needs escalation

Phase indicators:
- Discovery
- Design
- Development
- Testing
- Launch
- Maintenance

## 5. Tips for Project Health Assessment

Assess project health based on:
- **Velocity**: Is work progressing at expected pace?
- **Blockers**: Are there dependencies blocking progress?
- **Communication**: Is stakeholder alignment maintained?
- **Technical**: Are there technical challenges or debt?
- **Resources**: Are there resource constraints?

## Usage Examples

```bash
# Show all projects
/project-status

# Show specific project
/project-status My App Redesign

# Show specific project (partial name)
/project-status my-app
```

## Notes

- Focus on actionable information
- Highlight blockers prominently
- Keep it concise - this is a quick status check
- Update notes if information is stale
