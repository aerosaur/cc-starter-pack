# User Stories, Design Briefs & Technical Briefs

## User Story Writing

### Standard Format
```
As a [user type]
I want to [action]
So that [benefit]

Acceptance Criteria:
- [ ] Given [context], when [action], then [result]
- [ ] Given [context], when [action], then [result]

Priority: P0/P1/P2
Story Points: [Number]
```

### Enhanced Format (With Context)
```
As a [user type]
When [situation]
I want to [action]
So that [benefit]
Because [underlying need]

Acceptance Criteria:
- [ ] Given I'm a logged-in user
      When I click "Export"
      Then CSV downloads with all data

- [ ] Given CSV downloads
      When I open in Excel
      Then columns match dashboard labels
      And dates are Excel-compatible

Priority: P1
Points: 3
```

### User Story Tips
- One action per story
- User-facing benefit, not implementation
- Testable acceptance criteria
- Include edge cases in criteria

---

## Design Brief

```markdown
# Design Brief: [Feature]

**Designer:** @Name | **PM:** @Name | **Date:** YYYY-MM-DD

## Project Overview
**What:** [What we're designing]
**Why:** [Problem we're solving]

## Target Users
**Primary Persona:**
- Role: [Description]
- Tech savvy: [Level]
- Context: [When they use this]

## Design Requirements

### Must Have (P0)
- [ ] [Requirement 1]
- [ ] [Requirement 2]

### Should Have (P1)
- [ ] [Requirement 1]

## Design Principles
1. **[Principle]:** [Why it matters]
2. **[Principle]:** [Why it matters]

## Constraints
**Technical:** [What this means for design]
**Accessibility:** WCAG Level [A/AA]
**Platform:** [Desktop/Mobile requirements]

## Success Metrics
- [Metric 1]: [Target]
- [Metric 2]: [Target]

## Deliverables
- [ ] User flows
- [ ] Wireframes
- [ ] High-fidelity mockups
- [ ] Design specs

## Timeline
| Milestone | Date |
|-----------|------|
| Low-fi review | YYYY-MM-DD |
| High-fi review | YYYY-MM-DD |
| Handoff | YYYY-MM-DD |
```

---

## Technical Brief

```markdown
# Technical Brief: [Feature]

**PM:** @Name | **Tech Lead:** @Name | **Date:** YYYY-MM-DD

## What We're Building
[High-level description]

## Why
**User Problem:** [Problem]
**Success Metrics:** [Targets]

## Functional Requirements

### Core Functionality
1. **[Feature]**
   - [Behavior description]
   - [Edge cases]

## Non-Functional Requirements

### Performance
- Response time: [Requirement]
- Concurrent users: [Number]

### Security
- Authentication: [Requirements]
- Authorization: [Who can do what]

### Compliance
- [GDPR/HIPAA]: [Requirements]

## API Requirements

**Endpoint:** [Name]
```
Method: GET/POST
Path: /api/v1/resource

Request: {
  "field": "type"
}

Response: {
  "field": "type"
}
```

## Data Requirements

**Entity:** [Name]
- field1: [type, required/optional]
- field2: [type, required/optional]

## Edge Cases
1. **[Case]**
   - Scenario: [Description]
   - Expected: [How to handle]

## Testing Requirements
- Unit tests: [Coverage needed]
- E2E tests: [Critical flows]
```

---

## Brief Comparison

| Aspect | Design Brief | Technical Brief |
|--------|--------------|-----------------|
| **Audience** | Designers | Engineers |
| **Focus** | User experience | Implementation |
| **Details** | Personas, flows | APIs, data models |
| **Constraints** | Visual, accessibility | Performance, security |
| **Deliverables** | Mockups, specs | Architecture, tests |
