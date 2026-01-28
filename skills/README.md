# Skills

Skills are domain-specific knowledge packages that Claude activates based on context. When you're working on frontend code, Claude loads frontend patterns. When you're writing documentation, it loads technical writing guidelines.

## How Skills Work

1. **Automatic Activation**: Claude reads skill descriptions and activates relevant ones based on your request
2. **Tool Access**: Each skill specifies which tools it can use
3. **Reference Docs**: Skills can include reference files with patterns and examples

## Included Skills

### Development

| Skill | Description |
|-------|-------------|
| `senior-frontend` | React, Next.js, TypeScript, Tailwind patterns |
| `backend-nodejs` | Node.js, Express, APIs, databases |
| `database-sql` | PostgreSQL, MySQL, Prisma, query optimization |
| `vanilla-web-dev` | HTML, CSS, JavaScript without frameworks |
| `chrome-extension-dev` | Chrome extension development with Manifest V3 |
| `testing-automation` | Jest, Vitest, Playwright, Cypress |

### Design & UI

| Skill | Description |
|-------|-------------|
| `design-principles` | Linear/Notion/Stripe-style UI patterns |
| `ui-design-system` | Design tokens, component documentation |
| `ux-researcher-designer` | User research, personas, journey mapping |

### Content & Documentation

| Skill | Description |
|-------|-------------|
| `technical-writing` | API docs, READMEs, ADRs, runbooks |
| `brief-writing` | PRDs, feature briefs, user stories |
| `note-taking` | Meeting notes, daily notes, templates |
| `seo-geo` | SEO optimization, keyword research |

### Automation & Tools

| Skill | Description |
|-------|-------------|
| `n8n-automation` | n8n workflow creation and debugging |
| `ai-prompting` | Prompt engineering for automation |
| `api-client-dev` | REST clients, OAuth, retry logic |
| `shell-scripting-macos` | Bash/zsh, macOS automation |

## Skill File Structure

Each skill is a folder containing at minimum a `skill.md` file:

```
skills/
├── my-skill/
│   ├── skill.md           # Required: Main skill definition
│   ├── references/        # Optional: Reference documentation
│   │   ├── patterns.md
│   │   └── examples.md
│   └── scripts/           # Optional: Helper scripts
│       └── analyzer.py
```

### skill.md Format

```markdown
---
name: my-skill
description: When to use this skill (used for activation matching)
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# My Skill

Main content and instructions...
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Skill identifier |
| `description` | Yes | When to activate (Claude matches this against your request) |
| `allowed-tools` | Yes | Tools this skill can use |

## Creating New Skills

### 1. Create Skill Folder

```bash
mkdir -p ~/.claude/skills/my-skill
```

### 2. Create skill.md

```markdown
---
name: my-skill
description: Expert guidance for [domain]. Use when [trigger conditions].
allowed-tools: Read, Grep, Glob, Edit, Write
---

# My Skill Name

## Overview
What this skill provides...

## Best Practices
Key guidelines...

## Patterns
Common patterns and examples...

## Anti-Patterns
What to avoid...
```

### 3. Add References (Optional)

Create `references/` folder with detailed documentation:
- `patterns.md` - Design patterns and code examples
- `workflow.md` - Step-by-step processes
- `best-practices.md` - Guidelines and recommendations

### 4. Add Scripts (Optional)

Create `scripts/` folder with helper tools:
- Analysis scripts
- Generators
- Validators

## Tips for Good Skills

1. **Clear Description**: The description determines when the skill activates
2. **Focused Scope**: One skill = one domain
3. **Actionable Content**: Include examples and patterns
4. **Reference Docs**: Put detailed info in references, not the main skill.md
5. **Tool Selection**: Only allow tools the skill actually needs

## Customization

Feel free to:
- Modify existing skills to match your preferences
- Delete skills you don't use
- Create new skills for your specific workflows
- Add company-specific patterns to reference docs

## Skill Activation Examples

| You Say | Skill Activated |
|---------|-----------------|
| "Help me build a React component" | `senior-frontend` |
| "Write tests for this function" | `testing-automation` |
| "Create a PRD for this feature" | `brief-writing` |
| "Optimize this SQL query" | `database-sql` |
| "Set up an n8n workflow" | `n8n-automation` |
