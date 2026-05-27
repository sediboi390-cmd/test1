---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions
inclusion: always
---

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, skip this skill.
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance a skill might apply to what you are doing, you ABSOLUTELY MUST invoke the skill.

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.

This is not negotiable. This is not optional. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>

## Instruction Priority

Superpowers skills override default system prompt behavior, but **user instructions always take precedence**:

1. **User's explicit instructions** (direct requests) — highest priority
2. **Superpowers skills** — override default system behavior where they conflict
3. **Default system prompt** — lowest priority

## How to Use Skills

The skills are available as steering files in `.kiro/steering/`. Read and follow them directly when relevant.

## The Rule

**Invoke relevant skills BEFORE any response or action.** Even a 1% chance a skill might apply means you should check it. If an invoked skill turns out to be wrong for the situation, you don't need to use it.

## Red Flags

These thoughts mean STOP — you're rationalizing:

| Thought | Reality |
|---------|---------|
| "This is just a simple question" | Questions are tasks. Check for skills. |
| "I need more context first" | Skill check comes BEFORE clarifying questions. |
| "Let me explore the codebase first" | Skills tell you HOW to explore. Check first. |
| "This doesn't need a formal skill" | If a skill exists, use it. |
| "I remember this skill" | Skills evolve. Read current version. |
| "This doesn't count as a task" | Action = task. Check for skills. |
| "The skill is overkill" | Simple things become complex. Use it. |

## Skill Priority

When multiple skills could apply, use this order:

1. **Process skills first** (brainstorming, debugging) — these determine HOW to approach the task
2. **Implementation skills second** — these guide execution

- "Let's build X" → brainstorming first, then implementation skills.
- "Fix this bug" → systematic-debugging first, then domain-specific skills.

## Available Skills

- `superpowers-brainstorming` — before any creative work or feature building
- `superpowers-writing-plans` — after design is approved, before coding
- `superpowers-executing-plans` — when executing an implementation plan inline
- `superpowers-subagent-driven-development` — when executing plans with subagents
- `superpowers-test-driven-development` — when implementing any feature or bugfix
- `superpowers-systematic-debugging` — when encountering any bug or unexpected behavior
- `superpowers-verification-before-completion` — before claiming work is done
- `superpowers-requesting-code-review` — after completing tasks or features
- `superpowers-receiving-code-review` — when receiving review feedback
- `superpowers-using-git-worktrees` — before starting feature work
- `superpowers-finishing-a-development-branch` — when implementation is complete
- `superpowers-dispatching-parallel-agents` — when facing 2+ independent tasks
