# Zentropi Skills

Agent skills for building systems powered by the
[Zentropi](https://zentropi.ai) classification engine.

These skills follow the [Agent Skills](https://github.com/anthropics/agent-skills)
open standard. They teach AI coding agents how to integrate Zentropi's
CoPE model into applications — from writing classification policies to
calling the API to building labeling pipelines.

## Skills

| Skill | Description |
|-------|-------------|
| [zentropi-classifier](./zentropi-classifier/) | Classify content against custom policies using the Zentropi API. Single and batch classification, policy writing guidance, and integration patterns. |

## Quick Start

### 0. Installation

Each skill directory contains a `SKILL.md` that AI agents read to learn
the capability. To install a skill:

**Automatic installation:**
Just ask your agent: "Install the zentropi skills at https://github.com/zentropi-ai/skills"

**Manual installation:**
Copy the skill directory into your agent's skills folder. The agent
reads `SKILL.md` on demand and follows the instructions to classify
content, write policies, and handle errors.

### 1. Get an API key

Sign up at [zentropi.ai](https://zentropi.ai) and create an API key.

```bash
export ZENTROPI_API_KEY="your-key-here"
```

### 2. Classify content

Ask your agent to classify content against your own criteria:

> Classify w/ Zentropi if this is funny: "I tried to catch fog. I mist."

The agent will write a basic policy, call the Zentropi API, and return results:

```
Label: 1 (criteria matched), Confidence: 0.60
```

### 3. Write better policies

Simple one-line criteria work for prototyping, but production policies
should follow the [CoPE policy format](./zentropi-classifier/references/policy-writing-guide.md)
— a structured template with defined terms, inclusion criteria,
exclusion criteria, and examples.

You may also use policies that our community has publicly shared on zentropi.ai. For example,
try out this sexual content classifier with `labeler_version_id = 'b5c41878-e659-4b3d-be70-fd85830af4d5'`

## What is CoPE?

[CoPE](https://huggingface.co/zentropi-ai/cope-a-9b) (Content Policy Evaluator) is Zentropi's classification model.
You write policies in plain English describing what to detect, and CoPE
returns a binary label (match / no match) with a confidence score. No
training data, no fine-tuning — just describe what you're looking for.

- **Text and images** — use `cope-a-9b` for text; `cope-b-12b` for
  multimodal (subscriber only)
- **Token context** — 8k tokens for `cope-a-9b`, 128k tokens for
  `cope-b-12b` (combined policy + content)
- **Sub-second inference** — typical response times under 1s; higher
  rate limits for subscribers

Learn more: [CoPE model card](https://huggingface.co/zentropi-ai/cope-a-9b)

## Resources

- [Zentropi Dashboard](https://zentropi.ai) — create labelers, manage API keys
- [API Documentation](https://zentropi.ai/support)
- [Policy Writing Guide](./zentropi-classifier/references/policy-writing-guide.md)
- [CoPE Model Card](https://huggingface.co/zentropi-ai/cope-a-9b)

## License

See [Zentropi Terms of Service](https://zentropi.ai/legal/terms).
