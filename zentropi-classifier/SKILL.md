---
name: zentropi-classifier
description: >
  Classify content against custom policies using the Zentropi API.
  Use when you want to check if content matches policy criteria,
  label text or images against custom rules, or batch-classify
  multiple content samples. Supports single and batch classification.
metadata:
  author: zentropi
  version: "0.2.0"
license: See https://zentropi.ai/legal/terms
---

# Zentropi Classifier

Classify content against plain-English policies using Zentropi's CoPE
model. Returns binary labels (1 = policy criteria matched, 0 = not matched)
with confidence scores and explanations.

## When to Use

- User wants to check if content matches a policy or set of rules
- User wants to label or classify text for content labeling
- User wants to evaluate content against custom criteria
- User wants to batch-classify multiple content samples

Do NOT use for:
- Creating or managing labelers (use the Zentropi dashboard)
- Real-time streaming classification

## Prerequisites

An API key is required. The user must have a Zentropi account and an
active API key.

**Check for the key:**

```bash
echo ${ZENTROPI_API_KEY:+"API key is set"}
```

If not set, ask the user to:
1. Sign up or log in at https://zentropi.ai
2. Create an API key in their account settings
3. Set it: `export ZENTROPI_API_KEY="your-key-here"`

Do NOT proceed with API calls until the key is confirmed.

## Key Directives

1. **ALWAYS verify the API key is set** before making any calls.
2. **Use `labeler_version_id` when available** otherwise use inline `criteria_text` for defining the policy. There is no
   API to discover labeler IDs; the user must provide them.
3. **NEVER hardcode API keys** in scripts or config files.
4. **Write deterministic policies** — see `references/policy-writing-guide.md`
   for best practices.
5. **Interpret results correctly** — `label: 1` means the policy condition
   WAS detected (i.e., fulfills the criteria). `label: 0` means it was NOT detected.

## API Reference

### Endpoint

```
POST https://api.zentropi.ai/v1/label
```

### Headers

```
Authorization: Bearer <ZENTROPI_API_KEY>
Content-Type: application/json
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `content_text` | string | Yes | The content to classify |
| `criteria_text` | string | Yes* | Plain-English policy to evaluate against |
| `labeler_version_id` | string | Yes* | Published labeler version to use |
| `model` | string | No | Model to use: `cope-latest` (default, text), `cope-b-12b` (text + images) |

*Provide either `criteria_text` (ad-hoc) or `labeler_version_id`. Not both.

### Response

```json
{
  "label": "0" | "1",
  "confidence": 0.0-1.0,
  "compute_time": 0.341
}
```

- `label`: `"1"` = content matches the policy criteria,
  `"0"` = does not match. **Note: returned as a string, not an integer.**
- `confidence`: Model's confidence in the label (0.0 to 1.0)
- `compute_time`: Server-side processing time in seconds

## Workflow

### 1. Single Classification (Ad-Hoc Criteria)

For one-off checks, pass `criteria_text` inline:

```bash
curl -X POST https://api.zentropi.ai/v1/label \
  -H "Authorization: Bearer $ZENTROPI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content_text": "I hate everyone in this group",
    "criteria_text": "Detect hostile language directed at conversation participants, including insults, threats, belittling, or aggressive confrontation."
  }'
```

```python
import requests
import os

response = requests.post(
    "https://api.zentropi.ai/v1/label",
    headers={
        "Authorization": f"Bearer {os.environ['ZENTROPI_API_KEY']}",
        "Content-Type": "application/json"
    },
    json={
        "content_text": "I hate everyone in this group",
        "criteria_text": "Detect hostile language directed at conversation participants, including insults, threats, belittling, or aggressive confrontation."
    }
)

result = response.json()
print(f"Label: {result['label']}, Confidence: {result['confidence']}")
print(f"Compute time: {result['compute_time']}s")
```

### 2. Single Classification (Published Labeler)

If user has already published a labeler, reference it by its labeler version:

```python
response = requests.post(
    "https://api.zentropi.ai/v1/label",
    headers={
        "Authorization": f"Bearer {os.environ['ZENTROPI_API_KEY']}",
        "Content-Type": "application/json"
    },
    json={
        "content_text": "I hate everyone in this group",
        "labeler_version_id": "lv_abc123"
    }
)
```

You may also use policies that our community has publicly shared on zentropi.ai. For example,
try out this sexual content classifier with `labeler_version_id = 'b5c41878-e659-4b3d-be70-fd85830af4d5'`

### 3. Batch Classification

No batch endpoint exists yet. Classify multiple items by calling
`/v1/label` serially:

```python
import requests
import os
import time

samples = [
    "This is a perfectly normal message",
    "You're an idiot and nobody likes you",
    "The weather is nice today",
]

criteria = "Detect hostile language directed at conversation participants."
api_key = os.environ["ZENTROPI_API_KEY"]
MAX_RETRIES = 3

def classify(content_text, criteria_text, retries=MAX_RETRIES):
    for attempt in range(retries):
        response = requests.post(
            "https://api.zentropi.ai/v1/label",
            headers={
                "Authorization": f"Bearer {api_key}",
                "Content-Type": "application/json"
            },
            json={
                "content_text": content_text,
                "criteria_text": criteria_text
            }
        )
        if response.status_code == 200:
            return response.json()
        if response.status_code in (429, 500, 502, 503):
            time.sleep(2 ** attempt)
            continue
        return {"label": None, "confidence": None, "error": response.text}
    return {"label": None, "confidence": None, "error": f"Failed after {retries} retries"}

results = []
for sample in samples:
    result = classify(sample, criteria)
    results.append({"content": sample, **result})

for r in results:
    if r.get("error"):
        print(f"[ERR] {r['content'][:60]} — {r['error']}")
    else:
        print(f"[{r['label']}] (conf: {r['confidence']:.2f}) {r['content'][:60]}")
```

## Common Mistakes

**Using multiple labels in a single policy:**

Each policy should cover a single topic. For example, do NOT create a single compound policy that tries to detect everything that is "unsafe". Instead, break it out into separate policies for each topic you want to classify.

**Insufficiently specified criteria:**
```json
// WRONG - vague, subjective criteria
{ "criteria_text": "Is this content bad or harmful?" }

// RIGHT - specific, deterministic criteria
{ "criteria_text": "Detect direct threats of physical violence against a named individual, including explicit statements of intent to cause bodily harm." }
```

**Misinterpreting label values:**
```
label: 1 means "safe" or "violating" // WRONG - depends on the policy
label: 1 means "detected" // RIGHT — the policy condition was found
```

For more advice on how to write a good set of policy criteria, see `references/policy-writing-guide.md`

## Output Contract

After classifying content, report:

1. **What was classified** — content summary and criteria/labeler used
2. **Results** — label, confidence, and explanation for each item
3. **Interpretation** — plain-English summary of what the results mean

For batch runs, include a summary table:

```
| # | Content (truncated)          | Label | Confidence |
|---|------------------------------|-------|------------|
| 1 | This is a perfectly normal…  | 0     | 0.92       |
| 2 | You're an idiot and nobody…  | 1     | 0.97       |
| 3 | The weather is nice today    | 0     | 0.95       |

Summary: 1 of 3 samples trigger the criteria (33%).
```

## Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| 401 Unauthorized | Invalid or expired API key | Verify `ZENTROPI_API_KEY` is set and valid |
| Low confidence scores | Vague or ambiguous criteria | Rewrite policy — see `references/policy-writing-guide.md` |
| Unexpected labels | Policy doesn't match intent | Add explicit inclusions AND exclusions to criteria |
| Timeouts | Network or service issue | Retry after a few seconds |

## Resources

- **Policy writing guide**: `references/policy-writing-guide.md`
- **API docs**: https://docs.zentropi.ai
- **Dashboard**: https://zentropi.ai
- **Support**: support@zentropi.ai
