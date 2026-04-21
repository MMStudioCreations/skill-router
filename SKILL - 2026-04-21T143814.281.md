---
name: universal-skill-router
description: Use at the start of any session, or when a prompt arrives without an explicit skill named. Reads the installed skill library at runtime, matches prompt intent against skill descriptions, resolves prerequisite chains, and outputs the correct skill sequence with a ready-to-execute trigger — without requiring the user to know any skill names. Triggers on any new session opening, any task description, any vague request. If the user describes what they want to do and no skill is specified, this router runs first.
---

# Universal Skill Router

## Core Principle

No hardcoded skill names. No hardcoded domains. The router builds its routing map at runtime by reading the installed skill library — so it works for any user with any skill set.

**Iron Rule:** Route, don't execute. This skill selects and sequences skills. It does not do the work.

---

## Step 1: Build the Runtime Skill Index

At the start of every routing decision, scan the installed skill directories and extract the `name` and `description` fields from each `SKILL.md` frontmatter.

**Standard skill locations to scan (check all that exist):**
```
~/.claude/skills/          # Claude Code personal skills
~/.agents/skills/          # Codex / agent skills
/mnt/skills/user/          # Platform-mounted user skills
/mnt/skills/public/        # Platform-mounted public skills
./skills/                  # Project-local skills
```

**Extraction pattern:** Read the YAML frontmatter block (between `---` delimiters) and pull `name` and `description`. Store as a live index:

```
skill-name → description text
```

This index is the routing table. It is built fresh each session — never cached, never assumed.

**If no skill directories are found:** Tell the user which paths were checked and ask them to confirm their skill install location before proceeding.

---

## Step 2: Classify the Prompt

Read the user's prompt and extract:

1. **Primary intent** — what they want to accomplish (build, fix, audit, write, launch, plan, etc.)
2. **Domain signals** — nouns that suggest a category (feature, bug, security, copy, launch, content, deck, spec, test, etc.)
3. **Artifact signals** — references to existing files (spec, plan, context doc, codebase, PR, etc.)
4. **Constraint signals** — words that imply order dependencies (first, before, after, existing, new, already have)

Do not match to skills yet. Just extract signals.

---

## Step 3: Score Skills Against the Prompt

For each skill in the runtime index, score relevance against the extracted signals:

**Scoring rules:**
- Description contains a direct keyword match from the prompt → high relevance
- Description contains a synonym or related concept → medium relevance
- Description's "Use when..." clause matches the situation → high relevance
- Description's "Invoke for..." or trigger phrases match → high relevance
- Description explicitly says NOT to use in this situation → exclude

**Select the top 1–3 skills by relevance score.**

If two skills are close in score and serve different purposes (e.g. one is a process gate, one is an implementation skill), both may be correct — resolve via prerequisite chain in Step 4.

---

## Step 4: Resolve the Prerequisite Chain

Many skills only work correctly when a prior skill has produced an artifact. Before confirming a route, check the chain:

**Universal prerequisite patterns** (infer from skill descriptions, not hardcoded):

| If the top skill's description mentions... | Check whether... |
|---|---|
| "spec", "requirements", "before touching code" | A spec/requirements file already exists |
| "plan", "implementation plan", "task plan" | A plan file already exists |
| "context doc", "product context", "marketing context" | A context document already exists |
| "after all tasks complete", "once implementation is done" | Implementation is actually complete |
| "before claiming done", "before committing" | Work has been verified |

**If a prerequisite is missing:** Insert the prerequisite skill at the front of the sequence. State clearly: *"Before [target skill], we need [prerequisite]. Running that first."*

**Chains resolve left to right.** Check each skill in the sequence for its own prerequisites before confirming the full chain.

---

## Step 5: Handle Ambiguity

**If intent is unclear** (prompt is vague, could match 3+ skills equally):
→ Ask one question: *"Are you trying to [option A] or [option B]?"* — two options max, derived from top matches.

**If domain is unclear** (no project/codebase/context signals in prompt):
→ Ask one question: *"What are you working on?"*

**If a skill is explicitly named by the user:**
→ Skip Steps 2–3. Go directly to Step 4 (prerequisite check) then Step 5 (output).

**Never ask more than one question before routing.** Make a best-guess route if needed and state the assumption.

---

## Step 6: Output the Route

```
MATCHED SKILLS:  [skill-name-1] → [skill-name-2] → [skill-name-3]
REASONING:       [one sentence per skill explaining why it matched]
PREREQUISITE:    [None | Missing: describe what's needed]

TRIGGER:
"[Exact prompt ready to start the first skill — include all context from the user's original message]"

Next after this: [skill-name-2] — [one-line description of what it produces]
```

Then ask: *"Ready to run this? Or adjust anything?"*

---

## Step 7: Adapt the Trigger

The trigger prompt is not generic — it incorporates everything from the user's original message:

- Specific feature, task, or deliverable names they mentioned
- Tech stack or tools they referenced
- People, stakeholders, or audiences mentioned
- Files, paths, or artifacts they referenced
- Any constraints or requirements they stated

**Bad trigger (generic):**
> "Run feature-forge on the new feature."

**Good trigger (context-aware):**
> "Run feature-forge on the bulk delete feature for the card collection app. Users need to select multiple cards and apply: delete, update condition, update set, export. Stack: Cloudflare Workers + D1. Save spec to /specs/bulk-delete.spec.md"

The trigger should be complete enough that pasting it into a new session would produce the right result without additional context.

---

## Multi-Skill Session Handling

**Single intent, single skill:** Output one route, one trigger.

**Single intent, sequential skills needed:** Output the full chain. After each skill completes, remind the user of the next step: *"Step 1 done. Next: [skill-name] — ready to continue?"*

**Multiple independent intents in one prompt:** Split into separate routes. List both. Ask which to run first.

**Ongoing session ("continue", "next step", "what's next"):** Infer position from conversation context. Identify which skill in the chain just completed, output the next one.

---

## Handling Skill Description Quality Variance

| Description pattern | How to handle |
|---|---|
| Starts with "Use when..." | High-confidence match. Direct extraction. |
| Starts with "Conducts..." / "Analyzes..." | Infer trigger from nouns/verbs. Match on task type. |
| Very short (< 20 words) | Read first section of SKILL.md body to supplement. |
| Mentions other skills ("see also X") | Flag as potential chain members. |
| Says "MUST use" / "REQUIRED before" | Treat as a process gate — fires before intended skill. |

---

## What This Skill Does Not Do

- Does not read the full body of every skill (only frontmatter descriptions, supplemented only when description is thin)
- Does not execute any work
- Does not override explicit user skill invocations
- Does not cache the skill index between sessions — always rebuilt fresh
- Does not assume skill locations — scans all standard paths

---

## Red Flags — Routing Failures to Avoid

| Signal | Problem | Correct behavior |
|---|---|---|
| Routing to an implementation skill when a prerequisite gate exists | Prerequisite artifact missing, skill produces incomplete output | Check for process gates first, insert if needed |
| Routing to the same skill the user just ran | Likely should be the *next* skill in the chain | Infer chain position from context |
| Multiple skills with equal relevance, picking arbitrarily | Wrong skill selected silently | Ask one clarifying question |
| User prompt contains a file path or existing artifact | Might change the prerequisite check outcome | Note the artifact, factor into chain resolution |
| Vague prompt, no clarifying question asked | Wrong route with false confidence | Ask before routing |
| Skill description says "not for X" and prompt is X | Wrong match | Exclude that skill, rescore remaining |

---

## Installation

Drop this folder into your skills directory alongside your other skills. The router reads from the same directory it lives in — no configuration needed.

```
your-skills-dir/
  universal-skill-router/
    SKILL.md          ← this file
  brainstorming/
    SKILL.md
  feature-forge/
    SKILL.md
  ... (your other skills)
```

Once installed, invoke it at the start of any session by name, or set it as a default entry point in your `CLAUDE.md` / `AGENTS.md`:

```markdown
# CLAUDE.md
At the start of every session, and whenever a task is described without
an explicit skill named, invoke the universal-skill-router skill first.
```
