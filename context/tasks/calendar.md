# Calendar Tasks

Workflows for Google Calendar. **Claude MUST follow these steps.**

## Pre-Flight Checklist

**Before ANY calendar operation:**

1. **GET** current time: Use `mcp__time__get_current_time` or check current-session.md
2. **VERIFY** timezone: Check user's configured timezone in CLAUDE.md
3. **CONFIRM** attendees: Check memory-keeper-long people.contacts for email addresses

---

## Creating Events

### Required Information

Before creating, ASK if not provided:
- Event title
- Date and time (with timezone)
- Duration
- Attendees (if any)
- Description (optional)

### Creation Workflow

1. **Parse** date/time to ISO format
2. **Convert** to user's timezone if needed
3. **Verify** no conflicts at that time
4. **Create** event with:
   - User as attendee (always include)
   - Clear title
   - Accurate timezone
5. **Confirm** event was created

---

## Viewing Events

### Today's Events
```
list-events(calendarId="primary", timeMin="today start", timeMax="today end")
```

### This Week
```
list-events(calendarId="primary", timeMin="week start", timeMax="week end")
```

### Specific Date
```
list-events(calendarId="primary", timeMin="date start", timeMax="date end")
```

---

## Modifying Events

### Adding Attendees
1. Get existing event details
2. Add new attendee email
3. Update event
4. **NOTE**: modify_event may not reliably add attendees - verify after

### Changing Time
1. Get existing event
2. Confirm new time with user
3. Update event
4. Verify change applied

### Canceling Events
1. Confirm with user before deleting
2. Note if attendees need to be notified
3. Delete event

---

## Free/Busy Checks

Before scheduling meetings:
1. Check user's availability
2. Check attendees' availability if possible
3. Suggest available slots

---

## Known Issues

- `modify_event` doesn't always add attendees reliably - verify and retry if needed
- Always include user as attendee when creating events
- Timezone handling requires explicit conversion

---

## Time Zone Reference

Common timezone identifiers:
- US Pacific: `America/Los_Angeles`
- US Mountain: `America/Denver`
- US Central: `America/Chicago`
- US Eastern: `America/New_York`
- UK: `Europe/London`
- Central Europe: `Europe/Paris`
- Australia Sydney: `Australia/Sydney`
