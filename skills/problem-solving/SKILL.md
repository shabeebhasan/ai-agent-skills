---
name: Problem Solving
description: Debugging workflows, root cause analysis, performance optimization, and systematic troubleshooting for this legacy PHP 5.6 Yii codebase. Use when behavior is broken, tests fail, responses differ from spec, or recent changes need diagnosis.
---

# Problem Solving Skill

Use this skill when debugging issues, diagnosing failures, or resolving unexpected behavior.

## Project-Specific Debugging Checks

Always include a PHP 5.6 compatibility pass:

- check for accidental `\Throwable`
- check for PHP 7+ syntax such as `??`, return types, scalar types, or typed properties
- check new helper signatures for legacy-incompatible type hints
- prefer `rg` for searches across `models/`, `controllers/`, `views/`, and `web/js/`

## Debugging Workflow

### Step 1: Reproduce

Confirm:

```text
exact trigger steps
expected behavior
actual behavior
whether the issue is deterministic
whether it started after recent changes
```

### Step 2: Isolate

Narrow the scope:

```text
system -> layer -> file -> method -> line
```

Useful commands:

```bash
git log --oneline -20
git diff HEAD~5 -- path/to/file.php
rg -n "functionName|error_code|target_code" models controllers views web/js
```

### Step 3: Find Root Cause

- Trace data flow.
- Identify the first wrong value, not the last visible symptom.
- Check whether the issue is contract drift, config drift, or compatibility drift.

### Step 4: Fix Minimally

- Fix the actual cause.
- Avoid opportunistic refactors during a bug fix.
- Preserve backward compatibility.

### Step 5: Verify

```text
original issue resolved
no obvious regressions introduced
deterministic errors still mapped correctly
no debug leftovers
PHP 5.6 compatibility preserved
```

## Common Project Failure Patterns

- JSON endpoint returns HTML: response format not set to JSON
- Deterministic error missing: service threw or returned ad hoc message
- Parse error after small edit: PHP 7+ syntax introduced into PHP 5.6 code
- Status update rejected: wrong precondition or validated transition rule
- Route mismatch: duplicated decision logic instead of reusing canonical service
