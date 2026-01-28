# Advanced Techniques

## Chain-of-Thought for Automations

```
Analyze this support ticket using step-by-step reasoning.

Ticket: {{$json.ticket}}

Think through:
Step 1: Identify the main issue
Step 2: Check for urgency indicators
Step 3: Determine required expertise
Step 4: Assign priority score

Reasoning:
[Your analysis]

Final Answer (JSON only):
{
  "issue": "string",
  "urgency": "low/medium/high",
  "priority_score": "1-10"
}
```

## Self-Consistency Checking

```
Task: Classify this email
Email: {{$json.email}}

Provide THREE independent classifications:
Classification 1: [category]
Classification 2: [category]
Classification 3: [category]

Final Answer: [most common category]
Confidence: [0.0-1.0 based on agreement]
```

## Dynamic Prompt Assembly

```javascript
// In workflow Code node
let systemPrompt = "You are a customer service assistant.";

// Add context based on customer tier
if ($json.customer_tier === 'enterprise') {
  systemPrompt += " This is an enterprise customer. Prioritize their needs.";
}

// Add urgency context
if ($json.urgency_score > 7) {
  systemPrompt += " This is URGENT. Provide immediate solutions.";
}

return {
  system: systemPrompt,
  user: $json.message
};
```

---

# Model-Specific Optimization

## GPT-4 / GPT-4 Turbo

- Temperature: 0.1-0.3 for consistency
- Max tokens: Set conservatively
- Best for: Complex reasoning, multi-step analysis

## Claude (Anthropic)

- Temperature: 0.0-0.2 for automation
- Use system prompts for role definition
- Best for: Long-context analysis, detailed extraction

Formatting tip - use XML-style tags:
```xml
<input>{{$json.data}}</input>
<instructions>...</instructions>
<output_format>...</output_format>
```

## GPT-3.5 Turbo

- Temperature: 0.0-0.2
- More explicit instructions needed
- Best for: Simple classification, cost-effective operations
- Tips: Use more examples, be extra explicit, validate outputs rigorously

---

# Token Optimization

## Reduce Token Usage

### Abbreviate Instructions
```
❌ Verbose:
"Please carefully analyze the following customer message..."

✅ Concise:
"Classify sentiment: positive/negative/neutral
Input: {{$json.message}}"
```

### Use Token-Efficient Examples
- Keep examples minimal
- Show only key patterns

### Reduce Context
```javascript
// Instead of full document
const summary = $json.document.substring(0, 500);

// Instead of full history
const last3Messages = $json.history.slice(-3);
```

---

# Error Handling

## Handle Invalid Input

```
Input: {{$json.user_input}}

Before processing:
1. Is input empty? → Return: {"error": "empty_input"}
2. Is input too long? → Process first 1000 chars only
3. Is input wrong language? → Return: {"error": "unsupported_language"}

If valid, proceed with task.
```

## Specify Fallback Behavior

```
If you cannot determine [field] with high confidence:
- Set value to "UNKNOWN"
- Set confidence to 0.0
- Set requires_review to true
- Do not guess
```

---

# Common Pitfalls and Solutions

## Pitfall 1: Non-Parseable Output

**Problem:** AI adds explanatory text around JSON

**Solution:**
```
Respond with ONLY the JSON object. Do not include:
- Explanatory text
- Markdown formatting
- Code blocks
- Any text before or after JSON

Start with { and end with }
```

## Pitfall 2: Inconsistent Classifications

**Problem:** Same input, different outputs

**Solution:**
- Lower temperature to 0.0-0.1
- Add more specific examples
- Use explicit category lists
- Implement voting mechanism

## Pitfall 3: Hallucinated Data

**Problem:** AI invents information

**Solution:**
```
CRITICAL: Only extract information explicitly present.
If a field cannot be found, set it to null.
Do NOT infer, assume, or guess values.
```

---

# Production Best Practices

## Monitoring

```javascript
// Log prompt performance
{
  "prompt_id": "email_classifier_v2",
  "input_length": $json.input.length,
  "tokens_used": response.usage,
  "confidence": response.confidence,
  "success": true/false
}
```

## Version Control

```
Prompt Version: v2.3
Last Updated: 2024-10-15
Changes: Added handling for edge case XYZ
Performance: 94% accuracy on test set
```

## Graceful Degradation

```javascript
try {
  response = await callGPT4(prompt);
} catch (error) {
  response = await callClaude(prompt);
}

if (!response || response.confidence < 0.5) {
  response = rulesBasedFallback(input);
}
```
