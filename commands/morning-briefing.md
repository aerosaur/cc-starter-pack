---
description: Daily overview of calendar, tasks, and priorities
---

# Morning Briefing

Provide a morning overview based on available information sources.

**Note:** This command adapts based on what MCP servers you have configured. It will use whatever is available.

## 1. Check Available Integrations

Determine which integrations are available:
- Memory Keeper (for recent context)
- Google Calendar (for today's events)
- Time server (for current time/timezone)

Skip sections for integrations that aren't configured.

## 2. Current Time & Date

If time server is available, get current time in user's timezone.
Otherwise, note the current date.

## 3. Calendar Events (if available)

If Google Calendar is configured:
- Fetch today's events
- Show meetings with times, titles, and attendees
- Highlight back-to-back meetings or conflicts
- Identify blocks of free time

## 4. Recent Memory Context (if available)

If memory-keeper is configured:
- Load recent high-priority items
- Show recent decisions and progress
- Flag any incomplete tasks or follow-ups

## 5. Summary

Provide a quick overview:
- Number of meetings today (if calendar available)
- Key priorities from memory
- Any items needing immediate attention

## Output Format

```markdown
# Morning Briefing - [Date]

## Today's Schedule
[Calendar events or "No calendar configured"]

## Recent Context
[Memory items or "No memory keeper configured"]

## Priorities
- [Priority 1]
- [Priority 2]

## Quick Stats
- Meetings: X (Y hours)
- Open items: Z
```

**Keep it scannable** - use bullet points, tables, and clear headers.
