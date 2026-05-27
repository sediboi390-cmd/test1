---
name: receiving-code-review
description: Use when receiving code review feedback, before implementing suggestions - requires technical rigor and verification, not performative agreement or blind implementation
inclusion: manual
---

# Code Review Reception

## Overview

Code review requires technical evaluation, not emotional performance.

**Core principle:** Verify before implementing. Ask before assuming. Technical correctness over social comfort.

## The Response Pattern

```
1. READ: Complete feedback without reacting
2. UNDERSTAND: Restate requirement in own words (or ask)
3. VERIFY: Check against codebase reality
4. EVALUATE: Technically sound for THIS codebase?
5. RESPOND: Technical acknowledgment or reasoned pushback
6. IMPLEMENT: One item at a time, test each
```

## Forbidden Responses

**NEVER:**
- "You're absolutely right!" (performative)
- "Great point!" / "Excellent feedback!"
- "Let me implement that now" (before verification)

**INSTEAD:**
- Restate the technical requirement
- Ask clarifying questions
- Push back with technical reasoning if wrong
- Just start working (actions > words)

## Handling Unclear Feedback

If any item is unclear, STOP. Do not implement anything yet. Ask for clarification on ALL unclear items before proceeding.

## When To Push Back

Push back when:
- Suggestion breaks existing functionality
- Violates YAGNI (unused feature)
- Technically incorrect for this stack
- Conflicts with architectural decisions

**How to push back:** Use technical reasoning, not defensiveness.

## Acknowledging Correct Feedback

```
✅ "Fixed. [Brief description of what changed]"
✅ "Good catch — [specific issue]. Fixed in [location]."
✅ Just fix it and show in the code

❌ "You're absolutely right!"
❌ "Thanks for catching that!" (no gratitude expressions)
```

**Actions speak. Just fix it.**

## Implementation Order

1. Clarify anything unclear FIRST
2. Blocking issues (breaks, security)
3. Simple fixes (typos, imports)
4. Complex fixes (refactoring, logic)
5. Test each fix individually
6. Verify no regressions
