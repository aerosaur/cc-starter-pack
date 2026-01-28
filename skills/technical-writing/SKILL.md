---
name: technical-writing
description: Expert guidance for writing clear, concise technical documentation including API docs, README files, architecture decisions (ADRs), runbooks, user guides, and inline code documentation. Covers documentation structure, style guides, diagrams, and collaboration. Use when creating documentation, improving existing docs, or establishing documentation standards.
allowed-tools: Read, Grep, Glob, Edit, Write
---

# Technical Writing

Complete toolkit for creating clear, maintainable technical documentation that serves both developers and end-users.

## When to Use This Skill

- Creating README files or project documentation
- Writing API documentation (OpenAPI/Swagger)
- Creating Architecture Decision Records (ADRs)
- Writing runbooks and operational guides
- Documenting code (JSDoc, TSDoc)
- Establishing documentation standards

## Core Capabilities

| Category | Includes |
|----------|----------|
| **Doc Types** | README, API docs, ADRs, runbooks, user guides, changelogs |
| **Standards** | OpenAPI/Swagger, JSDoc/TSDoc, Keep a Changelog |
| **Diagrams** | Mermaid (flowcharts, sequence, architecture) |
| **Tools** | Markdown, Docusaurus, VitePress, Vale |

## Writing Principles

1. **Be concise** - Remove unnecessary words
2. **Use active voice** - "The system sends" not "The email is sent"
3. **Be specific** - "Increases performance by 50%" not "Better performance"
4. **Use examples** - Show, don't just tell
5. **Structure hierarchically** - Use headings and lists
6. **Keep consistent** - Same terms for same concepts

## Quick Reference

### README Structure
```
# Project Name
Brief description

## Overview
## Features
## Installation
## Quick Start
## Documentation
## Development
## Contributing
## License
```

### ADR Structure
```
# ADR-NNN: [Decision Title]

## Status
## Context
## Decision
## Consequences (Positive/Negative/Neutral)
## Implementation Notes
## Related
## Date
```

### Runbook Structure
```
# Runbook: [Procedure Name]

## Overview
## Prerequisites
## Impact
## Step-by-Step Procedure
## Rollback
## Troubleshooting
## Contacts
```

### Comment Guidelines
- Explain WHY, not WHAT
- Document business logic
- Mark workarounds with bug references
- Use TODO with context and ticket numbers

## Detailed Documentation

- [README & API Docs](references/readme-api-docs.md) - Templates for README, OpenAPI, JSDoc
- [ADRs & Runbooks](references/adrs-runbooks.md) - Decision records and operational guides
- [Style & Maintenance](references/style-maintenance.md) - Writing style, diagrams, review checklists
