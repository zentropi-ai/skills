# Zentropi Skills

Agent skills for building systems powered by the
[Zentropi](https://zentropi.ai) classification engine. They give AI agents the superpower of fast, accurate, flexible, content labeling.

These skills follow the [Agent Skills](https://github.com/anthropics/agent-skills)
open standard.  They teach AI coding agents how to integrate Zentropi into applications and workflows— from writing policies to
optimizing performance to deploying live.

## Skills

| Skill | Description |
|-------|-------------|
| [zentropi-labeler](./skills/zentropi-labeler/) | Create custom classifiers and label content against them using the Zentropi API. |

## Quick Start

### 0. Installation

**Automatic installation:**

Just ask your agent: "Install the zentropi skills at https://github.com/zentropi-ai/skills"

**Skills.sh package manager:**

Another fast way to install is with [skills.sh](https://skills.sh):

```bash
# Install every skill in this repo
npx skills add https://github.com/zentropi-ai/skills

# …or install just the zentropi-labeler skill
npx skills add https://github.com/zentropi-ai/skills --skill zentropi-labeler
```

**Claude Code plugin:**

This repo is also a [Claude Code](https://code.claude.com) plugin marketplace.
Add it once, then install the labeler plugin:

```
/plugin marketplace add zentropi-ai/skills
/plugin install zentropi-labeler@zentropi
```

**Manual installation:**

Copy the skill directory into your agent's skills folder. The agent
reads `SKILL.md` on demand and follows the instructions to build
labelers, write policies, classify content, and handle errors.

### 1. Get an API key

Sign up at [zentropi.ai](https://zentropi.ai) and create an API key.

```bash
export ZENTROPI_API_KEY="your-key-here"
```

### 2. Classify content

Ask your agent to label content against your own criteria:

> Label w/ Zentropi if this is funny: "I tried to catch fog. I mist."

The agent will write a basic policy, call the Zentropi API, and return results:

```
Label: 1 (criteria matched), Confidence: 0.60
```

### 3. Write better policies

Simple one-line criteria work for prototyping, but production policies
should follow the [CoPE policy format](./skills/zentropi-labeler/references/policy-writing-guide.md)
— a structured template with defined terms, inclusion criteria,
exclusion criteria, and examples.

You may also use policies that our community has publicly shared on zentropi.ai. For example,
try out this sexual content labeler with `labeler_version_id = 'b5c41878-e659-4b3d-be70-fd85830af4d5'`

## What is CoPE?

[CoPE](https://huggingface.co/zentropi-ai/cope-b-a4b) (Content Policy Evaluator) is Zentropi's latest classification model.
You write policies in plain English describing what to detect, and CoPE
returns a binary label (match / no match) with a confidence score. No
training data, no fine-tuning — just describe what you're looking for.

- **Text and visual content** — use `cope-b-a4b` for text; `cope-b-a4b-mm` for
  multimodal (subscriber only)
- **Token context** — 256k tokens (combined policy + content)
- **Sub-second inference** — typical response times under 1s; higher
  rate limits for subscribers

Learn more: [CoPE model card](https://huggingface.co/zentropi-ai/cope-b-a4b)

## Resources

- [Zentropi Platform](https://zentropi.ai) — create labelers, manage API keys
- [API Documentation](https://zentropi.ai/support)
- [Policy Writing Guide](./skills/zentropi-labeler/references/policy-writing-guide.md)
- [CoPE-B Model Card](https://huggingface.co/zentropi-ai/cope-b-a4b)

## License

See [Zentropi Terms of Service](https://zentropi.ai/legal/terms).
