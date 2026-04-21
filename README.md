# Universal Skill Router

**A meta-skill that routes any prompt to the right skill — automatically.**

Works with Claude Code, Codex, Gemini CLI, and any agent that uses the [skills spec](https://agentskills.io/specification). No configuration. No hardcoded skill names. Adapts to your installed skill set at runtime.

---

## The Problem

You have 50 skills installed. You type a prompt. Now you have to remember which skill handles it, whether a prerequisite skill needs to run first, and in what order.

That's cognitive overhead that defeats the purpose of having skills.

## The Solution

The Universal Skill Router reads your installed skill library, scores every skill against your prompt, resolves dependency chains, and outputs the exact trigger to run — ready to paste or execute.

```
You:    "my auth endpoint is returning 500 errors"

Router: MATCHED SKILLS:  systematic-debugging → secure-code-guardian → verification-before-completion
        REASONING:       Bug report → debug root cause first; auth surface → security review after fix; completion gate before claiming done
        PREREQUISITE:    None

        TRIGGER:
        "Run systematic-debugging on the auth endpoint 500 errors. Trace the failure through the request lifecycle before proposing any fix."

        Next after this: secure-code-guardian — reviews the auth surface once root cause is identified
```

---

## How It Works

**No hardcoded knowledge.** The routing table is built fresh every session by scanning your `SKILL.md` files and reading their `description` fields. Add or remove skills — the router adapts automatically.

**Five-step process:**

1. **Build runtime index** — scans all standard skill directories, extracts `name` + `description` from every installed skill
2. **Classify prompt** — extracts intent, domain signals, artifact references, and ordering constraints
3. **Score skills** — matches description trigger phrases against the prompt; excludes skills that explicitly say "not for this"
4. **Resolve prerequisite chain** — detects when a gate skill must run first (spec before plan, plan before execution, etc.) and inserts it automatically
5. **Output route + trigger** — structured routing block with a context-aware trigger prompt, ready to execute

---

## Installation

### Option 1: Clone into your skills directory

```bash
# Claude Code
git clone https://github.com/MMStudioCreations/skill-router.git \
  ~/.claude/skills/universal-skill-router

# Codex / agent skills
git clone https://github.com/MMStudioCreations/skill-router.git \
  ~/.agents/skills/universal-skill-router

# Project-local
git clone https://github.com/MMStudioCreations/skill-router.git \
  ./skills/universal-skill-router
```

### Option 2: Manual copy

Download `SKILL.md` and place it in a folder named `universal-skill-router` inside your skills directory.

### Activate automatic routing (recommended)

Add to your `CLAUDE.md` or `AGENTS.md`:

```markdown
At the start of every session, and whenever a task is described without
an explicit skill named, invoke the universal-skill-router skill first.
```

---

## Compatibility

| Platform | Works? | Skill location |
|---|---|---|
| Claude Code | ✅ | `~/.claude/skills/` |
| OpenAI Codex | ✅ | `~/.agents/skills/` |
| Gemini CLI | ✅ | `~/.gemini/antigravity/skills/` |
| Project-local | ✅ | `./skills/` |
| Any agent using the [skills spec](https://agentskills.io/specification) | ✅ | Varies |

---

## What Makes This Different

Most skill routers are **recommendation engines** — they suggest skills but leave sequencing and prerequisite resolution to you.

This router:
- **Resolves full chains**, not just single matches — knows that `writing-plans` needs a spec first, that `executing-plans` needs a plan, that `verification-before-completion` closes every session
- **Infers prerequisites from descriptions** — no hardcoded dependency graph; if a skill's description says "before touching code" or "required prior to," the router classifies it as a gate and inserts it automatically
- **Outputs an executable trigger**, not just a skill name — the trigger prompt includes all context from your original message so you can run it immediately
- **Works on any skill set** — tested against engineering, marketing, governance, and mixed skill libraries

---

## Requirements

Your skills need a `description` field in their YAML frontmatter:

```yaml
---
name: my-skill
description: Use when [trigger conditions]. Invoke for [specific situations].
---
```

Skills following the [agentskills.io specification](https://agentskills.io/specification) work out of the box. The router degrades gracefully on thin descriptions by reading the first section of the skill body.

---

## Example Routing Scenarios

| Prompt | Routed to |
|---|---|
| "add bulk delete to the app" | `brainstorming` → `feature-forge` → `writing-plans` |
| "fix the scan pipeline bug" | `systematic-debugging` → `verification-before-completion` |
| "write the landing page copy" | `product-marketing-context` [gate] → `copywriting` + `ai-seo` |
| "create the Q2 board deck" | `writing-plans` → `pptx` → `verification-before-completion` |
| "security audit the auth layer" | `secure-code-guardian` |
| "I'm done with the feature" | `verification-before-completion` → `finishing-a-development-branch` |
| "continue" | *(infers position from session context, outputs next skill)* |

---

## Contributing

Issues and PRs welcome. If you find a routing failure pattern — a prompt that routes to the wrong skill, or a prerequisite chain that doesn't resolve correctly — open an issue with the prompt, your installed skills, and the expected vs actual route.

---

## License

MIT

---

*Built by [MMStudioCreations](https://github.com/MMStudioCreations)*
