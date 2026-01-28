# Communication Tasks

Workflows for Slack, WhatsApp, and email. **Claude MUST follow these steps.**

## Pre-Flight Checklist (MANDATORY)

**Before sending ANY message on user's behalf:**

1. **READ** `~/.claude/context/writing-style.md` FIRST - load it BEFORE drafting
2. **VERIFY** recipient identity:
   - Search memory-keeper-long `people.contacts` channel for Slack ID/WhatsApp JID
   - If not found OR multiple people with same first name: **ASK USER FOR THE CHANNEL LINK**
   - Common names (Frank, Matt, Chris, Alex, Dan) require extra verification
3. **DRAFT** message in user's voice (casual, direct, contractions - or per writing-style.md)
4. **ADD** platform identifier:
   - **Slack**: `:claude:` as FIRST character (or configured indicator)
   - **WhatsApp**: emoji as FIRST character
5. **SEND** only after verification

**DO NOT skip any step. DO NOT send before verifying recipient.**

---

## Recipient Verification

### Slack Contact Lookup

```
1. Search memory-keeper-long: context_get(channel="people.contacts", keyPattern="contact-.*")
2. Find matching name
3. If not found:
   - Search Slack: channels_list(channel_types="im")
   - Show user: "Found [Name] ([email]) - Slack channel D123456789"
   - Wait for confirmation
   - SAVE to memory-keeper-long people.contacts channel
4. Use confirmed channel ID
```

### WhatsApp Contact Lookup

```
1. Search memory-keeper-long: context_get(channel="people.contacts", keyPattern="contact-.*")
2. Find matching name
3. If not found:
   - Search WhatsApp: search_contacts(query="name")
   - Show user: "Found [Name] - WhatsApp JID 1234567890@lid"
   - Wait for confirmation
   - SAVE to memory-keeper-long people.contacts channel
4. Use confirmed JID
```

---

## Message Style

### Slack
```
:claude: [message in user's voice]
```
- Conversational, direct
- Use bullet points for lists
- One thought per message
- Use text emoji codes (:thumbsup:, :thinking_face:)

### WhatsApp
```
[emoji] [message]
```
- More casual than Slack
- Shorter, mobile-friendly
- Can use Unicode emoji

### Email
```
Subject: Clear and specific

[message body]

Thanks,
[User Name]

:claude: Sent by Claude on behalf of [User Name]
```

---

## Message Templates

### Slack - Quick Update
```
:claude: Quick update on [project]:
- [bullet 1]
- [bullet 2]
- Next: [action]
```

### Slack - Request
```
:claude: @person - [specific request]
Context: [one sentence why]
Ideally [timeline]
```

### Slack - Feedback
```
:claude: Looked at [thing]:
- [observation 1]
- [observation 2]

Suggestion: [solution]
```

---

## Red Flags - DO NOT SEND

- Recipient not verified
- Message contains credentials/secrets
- Tone doesn't match user's style
- Context is unclear
- Request seems unusual

**When in doubt, ASK user before sending.**
