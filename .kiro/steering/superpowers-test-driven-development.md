---
name: test-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code
inclusion: manual
---

# Test-Driven Development (TDD)

## Overview

Write the test first. Watch it fail. Write minimal code to pass.

**Core principle:** If you didn't watch the test fail, you don't know if it tests the right thing.

## The Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

Write code before the test? Delete it. Start over. No exceptions.

## Red-Green-Refactor

### RED - Write Failing Test

Write one minimal test showing what should happen. Requirements:
- One behavior
- Clear name describing the behavior
- Uses real code (no mocks unless unavoidable)

### Verify RED - Watch It Fail (MANDATORY, never skip)

```bash
npm test path/to/test.test.ts
```

Confirm the test fails because the feature is missing (not because of typos).

### GREEN - Minimal Code

Write the simplest code to pass the test. Don't add features, refactor other code, or "improve" beyond the test.

### Verify GREEN - Watch It Pass (MANDATORY)

```bash
npm test path/to/test.test.ts
```

Confirm all tests pass with pristine output.

### REFACTOR - Clean Up

After green only: remove duplication, improve names, extract helpers. Keep tests green. Don't add behavior.

### Repeat

Next failing test for next feature.

## Common Rationalizations — All Wrong

| Excuse | Reality |
|--------|---------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests passing immediately prove nothing. |
| "Already manually tested" | Ad-hoc ≠ systematic. No record, can't re-run. |
| "Deleting X hours is wasteful" | Sunk cost fallacy. Keeping unverified code is technical debt. |
| "TDD will slow me down" | TDD is faster than debugging production bugs. |

## Red Flags — STOP and Start Over

- Code before test
- Test written after implementation
- Test passes immediately (you're testing existing behavior)
- Rationalizing "just this once"
- "Keep as reference" or "adapt existing code" (delete means delete)

## Verification Checklist

Before marking work complete:

- [ ] Every new function/method has a test
- [ ] Watched each test fail before implementing
- [ ] Each test failed for expected reason
- [ ] Wrote minimal code to pass each test
- [ ] All tests pass
- [ ] Output pristine (no errors, warnings)

## When Stuck

| Problem | Solution |
|---------|----------|
| Don't know how to test | Write wished-for API. Write assertion first. |
| Test too complicated | Design too complicated. Simplify interface. |
| Must mock everything | Code too coupled. Use dependency injection. |

## Final Rule

```
Production code → test exists and failed first
Otherwise → not TDD
```
