# Prompt Patterns for Common Tasks

## Pattern 1: Classification

```
You are a text classifier. Classify the following input into exactly one category.

Categories: [spam, inquiry, complaint, feedback, other]

Input: {{$json.message}}

Rules:
- Respond with ONLY the category name
- If uncertain, use "other"
- Be case-sensitive

Classification:
```

## Pattern 2: Structured Extraction

```
Extract structured information from the following text. If a field cannot be determined, use null.

Text: {{$json.content}}

Extract to this JSON format:
{
  "customer_name": "string or null",
  "email": "valid email or null",
  "inquiry_type": "one of: [sales, support, partnership, other]",
  "urgency_indicators": ["array", "of", "strings"]
}

JSON Response:
```

## Pattern 3: Content Generation

```
Generate a professional email response based on the following:

Recipient Type: {{$json.recipient_type}}
Context: {{$json.context}}
Tone: {{$json.tone}}

Requirements:
- Length: 50-150 words
- Include specific next steps
- End with clear call-to-action
- Do not use salutation (added automatically)
- Do not include signature (added automatically)

Email Body:
```

## Pattern 4: Sentiment Analysis

```
Analyze the sentiment of this message and provide a score.

Message: {{$json.message}}

Respond in this format:
SENTIMENT: [positive/neutral/negative]
SCORE: [number from -1.0 to 1.0]
CONFIDENCE: [number from 0.0 to 1.0]
KEY_EMOTION: [joy/anger/fear/sadness/surprise/trust/disgust/neutral]

Do not include explanations.
```

## Pattern 5: Data Enrichment

```
Enrich the following company data with additional context.

Company Name: {{$json.company_name}}
Website: {{$json.website}}

Based on common knowledge, provide:
{
  "market_segment": "enterprise/mid-market/smb",
  "likely_tech_stack": ["array"],
  "estimated_revenue_range": "range or unknown"
}

Respond ONLY with JSON. Use "unknown" for uncertain fields.
```

## Component Templates

### ROLE Definition
```
You are a professional email classifier specialized in customer support routing.
You are an SEO content analyzer with expertise in EEAT evaluation.
You are a data quality auditor for CRM systems.
```

### CONTEXT Provision
```
Context:
- User is a premium customer (account age: 2 years)
- Previous interaction: bug report filed yesterday
- Current ticket status: awaiting response
```

### TASK Description
```
Your task is to:
1. Analyze the customer email
2. Determine the primary intent
3. Identify urgency level
4. Extract key entities
5. Recommend routing destination
```

### OUTPUT Format
```
Respond ONLY with valid JSON in this exact format:
{
  "intent": "one of: [bug, feature, billing, cancellation]",
  "urgency": "one of: [low, medium, high, critical]",
  "confidence": "0.0 to 1.0"
}

Do not include any text before or after the JSON.
```

### CONSTRAINTS
```
Constraints:
- If input is unclear, set intent to "general_inquiry"
- Urgency "critical" only if explicitly mentioned
- Confidence below 0.6 should route to "human_review"
- Maximum processing time: 5 seconds
```

### EXAMPLES (Few-Shot)
```
Example 1:
Input: "My payment failed and I can't access my account!"
Output: {"intent": "billing", "urgency": "high", "confidence": 0.95}

Example 2:
Input: "How do I export data?"
Output: {"intent": "feature", "urgency": "low", "confidence": 0.90}
```
