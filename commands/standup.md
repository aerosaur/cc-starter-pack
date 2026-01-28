---
description: Generate standup update from recent work
---

Generate a concise standup update based on recent work activity. This creates a focused summary suitable for team standup meetings.

## 1. Gather Information

Collect recent work information from:

### Memory Keeper (if available)
- Search short-term memory for recent activity
- Look for items categorized as `progress`, `decision`, or `task`
- Filter for high-priority items

### Conversation History
- Review our recent conversations
- Extract completed work, key decisions, and progress

## 2. Structure the Standup Update

Create a concise update:

```markdown
# Standup Update - [Day, Month DD]

## Yesterday / Last Working Day
- [Specific accomplishment 1]
- [Specific accomplishment 2]

## Today's Plan
- [Specific task 1]
- [Specific task 2]

## Blockers
[None | Or list specific blockers]
```

## 3. Writing Guidelines

### For "Yesterday" Section:
- **Be specific**: "Implemented user authentication with JWT" not "worked on feature"
- **Include impact**: "Fixed bug affecting 10% of users" vs "fixed bug"
- **Keep it brief**: 2-4 bullet points maximum

### For "Today" Section:
- **Be actionable**: Clear, concrete tasks
- **Set realistic goals**: Don't overcommit
- **2-4 items maximum**

### For "Blockers" Section:
- **Be specific**: What exactly is blocking?
- **Include context**: Why is it blocking?
- Say "None" if no blockers

## 4. Format Options

Offer these formats:

**Markdown (Default)**
```
**Yesterday:** Implemented X, fixed Y
**Today:** Complete Z, review PRs
**Blockers:** None
```

**Slack-friendly (Compact)**
```
*Yesterday:* Implemented X, fixed Y
*Today:* Complete Z, review PRs
*Blockers:* None
```

## Notes

- Best run in the morning before standup
- Focus on outcomes, not activities
- Keep it concise - aim for 30-60 seconds when read aloud
