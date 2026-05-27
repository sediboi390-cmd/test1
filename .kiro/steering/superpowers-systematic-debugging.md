---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
inclusion: manual
---

# Systematic Debugging

## Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

## The Four Phases

### Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

1. **Read Error Messages Carefully** — Don't skip past errors. Read stack traces completely.
2. **Reproduce Consistently** — Can you trigger it reliably? What are the exact steps?
3. **Check Recent Changes** — What changed that could cause this? Git diff, recent commits.
4. **Gather Evidence in Multi-Component Systems** — Add diagnostic instrumentation at each component boundary. Run once to gather evidence showing WHERE it breaks.
5. **Trace Data Flow** — Where does the bad value originate? Keep tracing up until you find the source. Fix at source, not at symptom.

### Phase 2: Pattern Analysis

1. **Find Working Examples** — Locate similar working code in same codebase.
2. **Compare Against References** — Read reference implementations completely, not just skimming.
3. **Identify Differences** — List every difference between working and broken, however small.

### Phase 3: Hypothesis and Testing

1. **Form Single Hypothesis** — State clearly: "I think X is the root cause because Y"
2. **Test Minimally** — Make the SMALLEST possible change to test hypothesis. One variable at a time.
3. **Verify Before Continuing** — Did it work? Yes → Phase 4. Didn't work? Form NEW hypothesis.

### Phase 4: Implementation

1. **Create Failing Test Case** — Simplest possible reproduction. Use `superpowers-test-driven-development`.
2. **Implement Single Fix** — Address the root cause. ONE change at a time.
3. **Verify Fix** — Test passes? No other tests broken?
4. **If Fix Doesn't Work** — STOP. Count how many fixes tried. If ≥ 3: question the architecture. Don't attempt Fix #4 without architectural discussion.

## Red Flags — STOP and Follow Process

- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- Proposing solutions before tracing data flow
- "One more fix attempt" (when already tried 2+)

**All of these mean: STOP. Return to Phase 1.**

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too. |
| "Emergency, no time for process" | Systematic debugging is FASTER than thrashing. |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause. |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem. |

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, check changes, gather evidence | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |
