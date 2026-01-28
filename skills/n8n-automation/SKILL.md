---
name: n8n Workflow Automation
description: Expert guidance for creating, debugging, and optimizing n8n workflows. Use when the user mentions n8n, workflows, automation, creating automations, debugging workflows, workflow errors, API integrations, or needs help with n8n expressions, nodes, triggers, or data transformations.
allowed-tools: Read, Grep, Glob
---

# n8n Workflow Automation Skill

## Overview
This skill provides comprehensive guidance for creating, debugging, and optimizing n8n workflows. n8n is an open-source workflow automation tool that connects services and automates tasks through a visual interface.

## When to Use This Skill
- Creating new n8n workflows from scratch
- Debugging existing n8n workflows
- Optimizing workflow performance
- Implementing error handling and retry logic
- Working with n8n expressions and data transformation
- Integrating APIs and services into n8n workflows

## Core Concepts

### Workflow Architecture
n8n workflows consist of:
- **Trigger Nodes**: Initiate workflow execution (webhook, schedule, manual)
- **Action Nodes**: Perform operations (HTTP requests, database queries, AI calls)
- **Logic Nodes**: Control flow (IF, Switch, Merge, Split)
- **Data Transformation Nodes**: Manipulate data (Code, Function, Set)

### Execution Model
- Workflows execute node by node
- Each node receives input data from previous nodes
- Data flows as JSON objects
- Nodes can process items individually or in batches

## Best Practices

### 1. Workflow Design Principles

#### Start with Clear Requirements
- Define inputs, outputs, and expected behavior
- Map out the workflow logic before building
- Identify error scenarios and edge cases

#### Use Descriptive Node Names
```
Bad:  HTTP Request, HTTP Request 1, HTTP Request 2
Good: Fetch User Data, Update CRM Record, Send Slack Notification
```

#### Implement Proper Error Handling
```
Workflow Structure:
Trigger → Try → Action Nodes → Success Path
              ↓
         Error Trigger → Error Handler → Alert/Log
```

### 2. Data Management

#### Expression Syntax
n8n uses expressions to reference data:
```javascript
// Access input data
{{ $json.fieldName }}

// Access specific item
{{ $node["Node Name"].json.field }}

// Access all items
{{ $items }}

// Environment variables
{{ $env.API_KEY }}

// Execution metadata
{{ $now }}
{{ $workflow.id }}
{{ $execution.id }}
```

#### Common Data Transformations
```javascript
// String manipulation
{{ $json.email.toLowerCase() }}
{{ $json.name.trim() }}
{{ $json.text.split(',') }}

// Date handling
{{ $now.toISO() }}
{{ DateTime.fromISO($json.date).plus({ days: 7 }) }}

// Conditional logic
{{ $json.status === 'active' ? 'yes' : 'no' }}

// Array operations
{{ $json.items.length }}
{{ $json.items.map(item => item.id) }}
{{ $json.items.filter(item => item.active) }}
```

### 3. Node-Specific Best Practices

#### HTTP Request Node
```json
{
  "method": "POST",
  "url": "https://api.example.com/endpoint",
  "authentication": "predefinedCredential",
  "sendHeaders": true,
  "headerParameters": {
    "Content-Type": "application/json"
  },
  "sendBody": true,
  "bodyParameters": {
    "key": "={{ $json.value }}"
  },
  "options": {
    "timeout": 30000,
    "retry": {
      "enabled": true,
      "maxRetries": 3,
      "waitBetweenRetries": 1000
    }
  }
}
```

**Tips:**
- Always set appropriate timeouts
- Enable retries for flaky APIs
- Use authentication credentials, not hardcoded tokens
- Handle rate limits with Wait nodes

#### Code Node (JavaScript)
```javascript
// Process all items
for (const item of items) {
  // Transform data
  item.json.processedAt = new Date().toISOString();
  item.json.fullName = `${item.json.firstName} ${item.json.lastName}`;

  // Add computed fields
  item.json.isValid = validateEmail(item.json.email);
}

return items;

// Helper function
function validateEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

**Tips:**
- Use Code node for complex transformations
- Keep logic modular and documented
- Return the items array
- Use console.log() for debugging
- Handle null/undefined values

## Common Patterns

### Pattern 1: Webhook to Database
```
Webhook Trigger
→ Validate Input (IF node)
→ Transform Data (Code node)
→ Insert into Database (Postgres/MySQL node)
→ Send Success Response (Respond to Webhook)
```

### Pattern 2: Scheduled Data Sync
```
Schedule Trigger (daily at 2 AM)
→ Fetch from Source API
→ Transform Data
→ Compare with Existing (Code node)
→ Update Changed Records
→ Send Summary Email
```

### Pattern 3: AI-Enhanced Workflow
```
Trigger (Customer email)
→ Fetch Context Data
→ AI Prompt (ChatGPT/Claude)
→ Parse AI Response
→ Route Based on Intent (Switch)
→ Execute Appropriate Action
→ Log to Database
```

## Debugging Techniques

### 1. Use Execution History
- Check workflow executions in n8n UI
- Review input/output data for each node
- Identify where data transformation fails

### 2. Add Sticky Notes
Document complex logic and edge cases inline.

### 3. Test with Sample Data
```javascript
// Create test trigger with sample data
{
  "id": 1,
  "email": "test@example.com",
  "status": "active",
  "createdAt": "2024-01-01T00:00:00Z"
}
```

### 4. Progressive Testing
- Test each node individually
- Build workflow incrementally
- Use "Execute Node" feature

### 5. Logging Strategy
```javascript
// Add logging nodes before/after critical operations
console.log('Processing item:', item.json.id);
console.log('API response:', response);
```

## Common Issues and Solutions

### Issue 1: "Cannot read property of undefined"
**Cause:** Accessing nested data that doesn't exist
**Solution:**
```javascript
// Bad
{{ $json.user.address.city }}

// Good
{{ $json.user?.address?.city ?? 'N/A' }}
```

### Issue 2: Workflow Times Out
**Cause:** Long-running operations or external API delays
**Solutions:**
- Increase workflow timeout in settings
- Split into multiple workflows with webhooks
- Implement async processing patterns

### Issue 3: Data Not Passing Between Nodes
**Cause:** Node configuration or execution order
**Solutions:**
- Check node connections
- Verify "Execute Once" vs "Execute for Each Item"
- Use Merge node for parallel branches

## Integration-Specific Tips

### Google Sheets
- Use append for adding rows
- Use update for modifying existing data
- Consider batch operations for large datasets

### Slack
- Use Block Kit for rich messages
- Handle rate limits (1 message/second)
- Use threads for related messages

### OpenAI/Claude
- Set appropriate temperature values
- Implement token counting
- Cache responses when possible
- Use system prompts effectively
- Handle rate limits and retries

## Quick Reference

### Most Used Nodes
1. **HTTP Request** - API calls
2. **Code** - Custom JavaScript
3. **IF** - Conditional logic
4. **Set** - Data transformation
5. **Merge** - Combine data streams
6. **Split In Batches** - Process large datasets
7. **Wait** - Delay execution
8. **Error Trigger** - Handle workflow errors

When working with n8n workflows, always prioritize:
1. Clear workflow organization
2. Robust error handling
3. Security best practices
4. Performance optimization
5. Comprehensive testing
