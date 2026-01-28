# PRD & Feature Brief Templates

## Product Requirements Document (PRD)

### Full Template

```markdown
# [Product/Feature Name] - PRD

**Status:** 🔵 Draft / 🟡 In Review / 🟢 Approved
**Last Updated:** YYYY-MM-DD
**Author:** @PM Name

## Executive Summary
**TL;DR:** [2-3 sentences: What, why, and impact]

**Quick Stats:**
- Target Launch: [Quarter/Date]
- Estimated Effort: [S/M/L/XL]
- Impact: [High/Medium/Low]
- Priority: [P0/P1/P2]

## Problem Statement

### The Problem
[Describe the user problem or business opportunity]

**Who experiences this?**
- Primary users: [Description]
- Affected population: [Size]

**Why does this matter?**
- User impact: [How it affects users]
- Business impact: [Financial/strategic impact]
- Urgency: [Why now?]

### Evidence
**Quantitative:**
- [Metric 1]: Current → Target
- User complaints: [Number]

**Qualitative:**
- User research: [Summary]
- Customer quotes: "[Direct quote]"

## Goals and Success Metrics

### Objectives
1. **Primary Goal:** [What we're achieving]
2. **Secondary Goals:** [Additional benefits]

### Key Results
1. **[Metric Name]**
   - Baseline: [Current]
   - Target: [Goal] by [Date]
   - Measurement: [How]

## Proposed Solution

### Solution Overview
[High-level description]

**Core Functionality:**
1. [Capability 1]
2. [Capability 2]

**Key Differentiators:**
- [What makes this better]

### User Stories

**Story:**
As a [user type]
I want to [action]
So that [benefit]

**Acceptance Criteria:**
- [ ] Given [context], when [action], then [result]
- [ ] Given [context], when [action], then [result]

## Scope

### In Scope ✅
1. [Feature 1] - [Description]
2. [Feature 2] - [Description]

### Out of Scope ❌
1. [Feature A] - [Why deferred]

### Trade-offs
**Decision:** [What was chosen]
- Options: [A vs B]
- Rationale: [Why]

## Technical Considerations

### Requirements
- [Requirement 1]
- [Requirement 2]

### Performance
- Page load: < [X] seconds
- API response: < [Y] ms

### Security
- Authentication: [Requirements]
- Data encryption: [Requirements]

## Dependencies

| Team | Dependency | Owner | Due Date | Status |
|------|------------|-------|----------|--------|
| [Team] | [Need] | @Name | YYYY-MM-DD | 🟢/🟡/🔴 |

## Risks and Mitigation

**Risk 1:** [Description]
- Likelihood: High/Medium/Low
- Impact: High/Medium/Low
- Mitigation: [Strategy]
- Owner: @Name

## Implementation Plan

### Timeline
| Milestone | Date | Owner |
|-----------|------|-------|
| Design complete | YYYY-MM-DD | @Designer |
| Dev complete | YYYY-MM-DD | @Eng Lead |
| Launch | YYYY-MM-DD | @PM |

## Success Criteria

### Launch Criteria
- [ ] All P0 stories complete
- [ ] Zero critical bugs
- [ ] Performance benchmarks met
- [ ] Security review passed

### Validation Plan
**Week 1:**
- Monitor: [Metrics]
- Goal: [Threshold]
```

---

## Feature Brief (Lightweight)

Use when you need a quick, scannable spec for smaller features.

```markdown
# [Feature Name] - Brief

**Author:** @PM | **Date:** YYYY-MM-DD

## 🎯 Problem
[2-3 sentences]

**User pain:** [Specific pain]
**Business impact:** [Why it matters]

## 📊 Evidence
- [Data point 1]
- [User quote]

## ✨ Solution
[What we're building]

**Core functionality:**
1. [Key capability 1]
2. [Key capability 2]

## 🎯 Success Metrics
- **Primary:** [Metric] from [X] to [Y]

## 👥 Users
**Primary:** [Persona]
**Scenario:** [When they need this]

## ✅ Scope
**In:** [Items]
**Out:** [Items with reasons]

## 📅 Timeline
- Design: [Dates]
- Development: [Dates]
- Launch: [Target]
```

---

## When to Use Which

| Scenario | Use |
|----------|-----|
| New product/major feature | Full PRD |
| Quick enhancement | Feature Brief |
| Bug fix with scope | Feature Brief |
| Strategic initiative | Full PRD + Executive Brief |
| Design exploration | Design Brief |
