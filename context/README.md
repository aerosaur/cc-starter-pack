# Context Files

This directory contains context files that Claude reads to understand your preferences and workflows.

## Files

### writing-style.md

Defines your communication style for when Claude sends messages on your behalf. Customize this to match how you write in:
- Slack messages
- Emails
- WhatsApp
- Other platforms

### current-session.md

Automatically updated with the current session timestamp. Helps Claude understand the time context and reference the correct date.

### tasks/

Task-specific workflow files:

- **communication.md** - Workflows for Slack, WhatsApp, and email messaging
- **git.md** - Git safety protocols and commit workflows
- **calendar.md** - Calendar operation workflows

## Usage

These files are referenced in CLAUDE.md and read by Claude when performing specific tasks. For example, before sending any message, Claude should read `writing-style.md` to match your tone.

## Customization

Edit these files to match your preferences. The more specific you are, the better Claude can adapt to your style.

### Example: writing-style.md

If you prefer formal communication:
```markdown
## Core Characteristics
- **Tone**: Professional and formal
- **Sentences**: Complete sentences with proper punctuation
- **Contractions**: Avoid (use "cannot" not "can't")
```

If you prefer casual communication:
```markdown
## Core Characteristics
- **Tone**: Casual and friendly
- **Sentences**: Short and punchy
- **Contractions**: Yes (can't, won't, I'd)
- **Lowercase starts**: Often skip capitalization
```

## Adding New Context Files

Create additional context files for specific situations:

```markdown
# meetings.md
Guidelines for meeting-related tasks...

# code-review.md
Preferences for code review comments...
```

Then reference them in CLAUDE.md under the appropriate section.
