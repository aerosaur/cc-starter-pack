---
name: ai-prompting
description: Expert guidance on crafting effective AI prompts specifically for automation workflows (n8n, Zapier, Make, etc.). Use when the user mentions AI prompts, prompt engineering, automation prompts, GPT, Claude, OpenAI, language models, LLMs, improving AI responses, structured output, or needs help with prompt design, consistency, or integration of AI in workflows.
allowed-tools: Read, Grep, Glob
---

# AI Prompting for Automations

Expert guidance for crafting reliable, production-ready AI prompts for automation workflows.

## When to Use This Skill

- Creating AI prompts for workflow automations
- Debugging inconsistent AI responses
- Optimizing prompt performance and token usage
- Implementing structured output from AI models
- Building context-aware automation workflows

## Core Principles

### 1. Determinism Over Creativity
- Use low temperature (0.0-0.3)
- Provide explicit output formats
- Avoid open-ended instructions

### 2. Error Prevention
- Define behavior for missing data
- Specify fallback responses
- Handle unexpected formats

### 3. Parseable Outputs
- Request specific formats (JSON, CSV, YAML)
- Use delimiters and markers
- Enable automated extraction

## Prompt Architecture

```
[ROLE] + [CONTEXT] + [TASK] + [FORMAT] + [CONSTRAINTS] + [EXAMPLES]
```

| Component | Purpose | Example |
|-----------|---------|---------|
| ROLE | Define expertise | "You are a text classifier" |
| CONTEXT | Provide background | Customer tier, history |
| TASK | Clear instructions | "Classify into categories" |
| FORMAT | Output structure | JSON schema |
| CONSTRAINTS | Boundaries | "If unclear, use 'other'" |
| EXAMPLES | Few-shot learning | Input/output pairs |

## Quick Reference

### Model Settings

| Model | Task | Temperature | Max Tokens |
|-------|------|-------------|------------|
| GPT-4 | Classification | 0.1-0.2 | 100-500 |
| GPT-4 | Generation | 0.5-0.7 | 500-2000 |
| Claude | Extraction | 0.0-0.1 | 500-1500 |
| GPT-3.5 | Classification | 0.0-0.1 | 50-200 |

### Quick Templates

**Classification:**
```
Classify: {{input}}
Categories: [A, B, C]
Format: Single letter
```

**JSON Extraction:**
```
Extract from: {{input}}
Format: {{schema}}
Rules: Use null for missing
Response: JSON only
```

## Key Rule

A good automation prompt works reliably 10,000 times, not brilliantly once.

## Detailed Documentation

- [Prompt Patterns](references/patterns.md) - Classification, extraction, generation patterns
- [Advanced Techniques](references/techniques.md) - Chain-of-thought, model optimization, error handling
