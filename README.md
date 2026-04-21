# Universal Skill Router

A meta-skill that routes any prompt to the correct skill sequence — automatically, without requiring the user to know skill names.

## How It Works

1. **Scans your installed skills at runtime** — reads `name` and `description` from every `SKILL.md` frontmatter in your skills directory
2. **Classifies prompt intent** — extracts what the user wants to do, what domain it's in, and what artifacts already exist
3. **Scores skills against the prompt** — matches description trigger phrases and keywords to the prompt
4. **Resolves prerequisite chains** — checks whether a gate skill must run first (e.g. a spec before a plan, a plan before execution)
5. **Outputs a ready-to-run trigger** — the exact prompt to start the right skill, with full context from the user's original message

## Installation

```
your-skills-dir/
  universal-skill-router/   ← drop this folder here
    SKILL.md
    CLAUDE.md
    README.md
  your-other-skill/
    SKILL.md
  ...
```

Copy `CLAUDE.md` to your project root (or merge into your existing `CLAUDE.md`) to activate automatic routing.

## Works With Any Skill Set

No hardcoded skill names. No hardcoded domains. The routing table is built fresh from your actual installed skills at the start of every session. Add or remove skills — the router adapts automatically.

## Requirements

- Skills must have a `description` field in their YAML frontmatter
- Standard `SKILL.md` structure (frontmatter between `---` delimiters)
- Compatible with Claude Code, Codex, Gemini CLI, and any agent that supports the skills spec

## What It Doesn't Do

- Does not execute work — routes to the skill that does
- Does not override explicit skill invocations from the user
- Does not cache between sessions — always reads live skill state
