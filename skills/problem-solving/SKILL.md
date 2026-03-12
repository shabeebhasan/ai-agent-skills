---
name: Problem Solving
description: Debugging workflows, root cause analysis, performance optimization, and systematic troubleshooting
---

# Problem Solving Skill

Use this skill when debugging issues, diagnosing failures, optimizing performance, or resolving unexpected behavior.

## When to Activate

- Something that was working is now broken
- A feature behaves differently than expected
- Performance degradation is reported
- Error messages appear without clear cause
- Tests are failing unexpectedly

## Debugging Workflow

### Step 1: Reproduce

Before investigating, confirm:

```
□ Can I reproduce the issue?
□ What are the exact steps to trigger it?
□ What is the expected behavior?
□ What is the actual behavior?
□ When did it start? (recent changes? deployment?)
```

### Step 2: Isolate

Narrow the scope using binary search:

```
Full system → Which layer? (frontend / controller / service / DB)
Layer → Which file? (check recent changes with git log)
File → Which function? (add logging or breakpoints)
Function → Which line? (trace data flow)
```

**Quick isolation tools:**
```bash
# What changed recently?
git log --oneline -20
git diff HEAD~5 -- path/to/suspect/file.php

# Where is this called from?
grep -rn "functionName" models/ controllers/ --include="*.php"

# What's the actual value?
# Add temporary: Yii::warning(json_encode($variable), __METHOD__);
```

### Step 3: Root Cause Analysis

Ask the **5 Whys**:

```
Problem: Device shows wrong sorting target
1. Why? → sortingResolve() returned ST_CN_REF instead of ST_ISO_REP
2. Why? → functional_layer was 'functional' instead of 'limited'
3. Why? → normalizeLayer() received null from BoD
4. Why? → BoD query didn't join function_test table
5. Why? → Missing LEFT JOIN after recent refactor
→ Root cause: Query change in commit abc123
```

### Step 4: Fix

Apply the minimum change that resolves the root cause:

- **Don't fix symptoms** — fix the actual cause
- **Don't refactor during a fix** — make it work first, improve later
- **Add a test for the failure case** — prevent regression

### Step 5: Verify

```
□ Original issue is resolved
□ Fix doesn't break other functionality
□ Existing tests still pass
□ New test covers the failure case
□ No debug code left (console.log, var_dump, etc.)
```

## Common Issue Patterns

### PHP / Yii2

| Symptom | Likely Cause | Check |
|---------|-------------|-------|
| 500 error, no details | Exception in non-debug mode | Check `runtime/logs/app.log` |
| JSON returns HTML | Missing `response->format = JSON` | Controller action setup |
| Empty model on load | Wrong `load()` formName | `$form->load($data, '')` for no prefix |
| RBAC always denies | Permission not in `rbac/` config | Check `AuthManager` assignments |
| Session data lost | Missing `session->set()` | Verify session key spelling |
| Pjax not updating | Wrong Pjax ID or event | Match `#pjaxId` between trigger and widget |

### JavaScript / jQuery

| Symptom | Likely Cause | Check |
|---------|-------------|-------|
| Click handler not firing | Dynamic element, no delegation | Use `$root.on('click', selector, fn)` |
| AJAX returns 403 | Missing CSRF token | Include `_csrf` in POST data |
| State not updating | Object reference not updated | Ensure state mutation, not reassignment |
| Focus not moving | Element hidden or not in DOM | Check `display:none`, timing with `setTimeout` |
| Multiple event fires | Handler bound multiple times | Use `.off().on()` or namespace events |

### CSS

| Symptom | Likely Cause | Check |
|---------|-------------|-------|
| Styles not applied | CSS not registered | Verify `registerCssFile()` in view |
| Specificity conflict | External CSS overrides | Use more specific selectors or `!important` sparingly |
| Layout broken on resize | Missing `flex-wrap` or `overflow` | Add responsive breakpoints |

## Performance Optimization

### Identify Bottlenecks

```
1. Database: Slow queries → Add indexes, reduce N+1
2. PHP: Heavy computation → Cache results, lazy load
3. Network: Large payloads → Paginate, compress
4. Frontend: Excessive DOM updates → Batch renders, debounce
```

### Quick Wins

```php
// N+1 → Eager loading
$models = Model::find()->with(['relation'])->all();

// Repeated queries → Cache
$result = Yii::$app->cache->getOrSet('key', function() {
    return expensive_query();
}, 3600);

// Large result sets → Pagination
$query->limit(50)->offset($page * 50);
```

### Frontend Performance

```javascript
// Debounce rapid events (search/scroll)
var debounceTimer;
$input.on('input', function () {
    clearTimeout(debounceTimer);
    debounceTimer = setTimeout(function () { doSearch(); }, 300);
});

// Batch DOM updates
var html = '';
items.forEach(function (item) { html += renderItem(item); });
$container.html(html); // One DOM write, not N
```

## Escalation Checklist

If you can't resolve within 30 minutes of investigation:

```
□ Document what you've tried
□ Narrow to the smallest reproducible case
□ Check if it's a known framework/library issue
□ Search for similar issues in project history
□ Prepare a clear problem statement for the user
```
