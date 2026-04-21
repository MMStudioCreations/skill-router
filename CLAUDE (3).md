# CLAUDE.md — Universal Skill Router

## Automatic Routing

At the start of every session, and whenever a task or request is described
without an explicit skill named, invoke the `universal-skill-router` skill first.

Do not respond to the user's prompt directly until the router has:
1. Built the runtime skill index from installed skills
2. Classified the prompt intent
3. Resolved the skill sequence and prerequisite chain
4. Output the route and trigger

## Exceptions (skip the router)

- User explicitly names a skill: "run feature-forge on X" → go directly to that skill
- User is mid-session and already executing a skill sequence → stay in that sequence
- Conversational message with no task intent: greetings, clarifying questions, status checks

## Skill Priority (when router selects multiple)

1. Process gates first (skills whose descriptions say "before", "required prior to", "first")
2. Implementation skills second
3. Completion gates last (verification, finishing)
