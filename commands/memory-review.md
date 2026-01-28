---
description: Review and manage memory keeper items
---

Review memory keeper items, identify stale data, suggest migrations from short to long-term memory, and generate usage statistics.

## 1. Get Memory Status

Start by getting an overview of both memory stores:

```bash
# Get short-term memory status
mcp__memory-keeper-short__context_status

# Get long-term memory status
mcp__memory-keeper-long__context_status
```

## 2. Analyze Short-Term Memory

### Retrieve All Items
```bash
# Get all short-term memory items with metadata
mcp__memory-keeper-short__context_get
  limit: 100
  includeMetadata: true
  sort: "created_desc"
```

### Categorize Items
Organize items by:
- **Age**:
  - Today (< 24 hours)
  - This week (< 7 days)
  - This month (< 30 days)
  - Old (> 30 days)
- **Category**: task, decision, progress, note, error, warning
- **Priority**: high, normal, low
- **Channel**: Group by project/feature channels

### Identify Items for Action

**Items to Move to Long-Term:**
- High-priority decisions with long-term impact
- Completed solutions that are reference-worthy
- Working implementations and configurations
- Important gotchas and limitations
- Architectural decisions (should be in long-term)

**Items to Archive/Delete:**
- Completed tasks (> 30 days old)
- Resolved errors/warnings
- Temporary notes no longer relevant
- Duplicate information
- Obsolete information

**Items to Keep in Short-Term:**
- Active work items
- Recent decisions (< 7 days)
- Current project progress
- Recent errors needing resolution

## 3. Generate Review Report

Create a structured review with these sections:

```markdown
# Memory Keeper Review - [Date]

## Overview

### Short-Term Memory
- **Total Items**: X
- **Oldest Item**: [Date]
- **Newest Item**: [Date]

### Long-Term Memory
- **Total Items**: Y

---

## Statistics

### By Category
| Category | Count | % |
|----------|-------|---|
| task | X | Y% |
| decision | X | Y% |
| progress | X | Y% |
| note | X | Y% |
| error | X | Y% |

### By Priority
| Priority | Count | % |
|----------|-------|---|
| high | X | Y% |
| normal | X | Y% |
| low | X | Y% |

### By Channel (Top 10)
| Channel | Count |
|---------|-------|
| [channel-1] | X |
| [channel-2] | Y |

### By Age
| Age Range | Count |
|-----------|-------|
| < 24 hours | X |
| 1-7 days | Y |
| 8-30 days | Z |
| > 30 days | N |

---

## Recommended Actions

### 1. Move to Long-Term Memory

**Architectural Decisions:**
- [ ] `[key]` - [brief description] (created: [date])

**Working Solutions:**
- [ ] `[key]` - [brief description] (created: [date])

### 2. Archive/Delete (Stale Items)

**Completed Tasks (>30 days):**
- [ ] `[key]` (created: [date])

**Resolved Errors:**
- [ ] `[key]` (created: [date])

### 3. Items Needing Attention

**Unresolved Errors:**
- `[key]` - [description]

**Incomplete Tasks:**
- `[key]` - [description]

---

## Next Steps

1. **Immediate** (Do now):
   - [ ] Migrate X high-priority items to long-term
   - [ ] Delete Y obviously stale items

2. **This Week**:
   - [ ] Complete migration of all identified items
   - [ ] Clean up stale data

3. **Ongoing**:
   - [ ] Run /memory-review weekly
```

## 4. Interactive Actions

After generating the report, offer interactive options:

1. **Migrate selected items to long-term memory**
2. **Delete stale items**
3. **View specific category or channel**
4. **Generate migration plan**
5. **Export memory data**

## 5. Best Practices

### When to Move to Long-Term
- Permanent solutions and working code
- Architectural decisions with rationale
- System configurations that won't change often
- Important gotchas discovered
- Reference implementations

### When to Keep in Short-Term
- Active work in progress
- Recent decisions (< 7 days)
- Temporary experiments
- Current session context

### When to Delete
- Completed tasks (> 30 days old)
- Resolved errors
- Obsolete information
- Duplicate items
