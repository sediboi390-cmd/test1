---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session
inclusion: manual
---

# Subagent-Driven Development

Execute plan by dispatching fresh subagent per task, with two-stage review after each: spec compliance review first, then code quality review.

**Core principle:** Fresh subagent per task + two-stage review (spec then quality) = high quality, fast iteration

**Continuous execution:** Do not pause to check in between tasks. Execute all tasks from the plan without stopping unless: BLOCKED, genuine ambiguity, or all tasks complete.

## The Process

### Setup
1. Read plan file once, extract ALL tasks with full text
2. Note context needed for each task
3. Create a task list with all tasks

### Per Task
1. Dispatch implementer subagent with full task text + context
2. Answer any questions the subagent raises
3. Dispatch spec compliance reviewer subagent
4. If spec issues found → implementer fixes → re-review
5. Dispatch code quality reviewer subagent
6. If quality issues found → implementer fixes → re-review
7. Mark task complete

### Completion
After all tasks: dispatch final code reviewer for entire implementation, then use `superpowers-finishing-a-development-branch`.

## Handling Implementer Status

- **DONE:** Proceed to spec compliance review
- **DONE_WITH_CONCERNS:** Read concerns before proceeding. Address correctness/scope concerns first.
- **NEEDS_CONTEXT:** Provide missing context and re-dispatch
- **BLOCKED:** Assess blocker. Provide more context, use more capable model, break into smaller pieces, or escalate to human.

## Review Order (CRITICAL)

**ALWAYS: spec compliance first, THEN code quality.**

Never start code quality review before spec compliance is ✅.

## Red Flags

**Never:**
- Start implementation on main/master branch without explicit user consent
- Skip reviews (either stage)
- Proceed with unfixed issues
- Move to next task while reviews have open issues
- Start code quality review before spec compliance is approved
