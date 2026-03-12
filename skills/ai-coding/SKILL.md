---
name: AI-Assisted Coding
description: Specification-driven development, systematic code generation, refactoring strategies, and modern development patterns
---

# AI-Assisted Coding Skill

Use this skill for writing production code efficiently using specification-driven approaches, systematic refactoring, and modern patterns.

## When to Activate

- Implementing from a T-spec or requirements document
- Writing new services, controllers, views, or JS modules
- Refactoring existing code to new patterns
- Generating tests from implementation

## Core Method: Spec → Plan → Code → Test

### Step 1: Parse the Specification

Extract these from any spec:

```
□ Input contract (what data comes in, from where)
□ Output contract (response shape, error codes)
□ State transitions (before → after)
□ Dependencies (upstream services, data sources)
□ Acceptance criteria (what makes this "done")
```

### Step 2: Create Implementation Plan

Before writing any code, produce:

1. **File list** — every file that will be created or modified
2. **Dependency order** — which files must exist before others
3. **Change size** — estimate lines per file (keep files focused)
4. **Test plan** — what scenarios to cover

### Step 3: Code Generation Patterns

#### Service Methods

```php
/**
 * Brief description of what this does.
 * @param string $input Description
 * @return array ['success' => 1, 'field' => value] | ['success' => 0, 'error_code' => 'CODE']
 */
public static function doSomething($input) {
    // 1. Validate / normalize input
    // 2. Load required data
    // 3. Apply business logic
    // 4. Return deterministic result
}
```

**Rules:**
- Static methods for stateless operations
- Return arrays with `success` flag, never throw for business errors
- Normalize all inputs early (trim, strtoupper, type cast)
- Every branch returns a response, never falls through

#### Controller Actions

```php
public function actionDoSomething() {
    $this->ensureAccess();                    // 1. Auth
    Yii::$app->response->format = 'json';     // 2. Format
    $form = new FormModel();                  // 3. Load + validate
    $form->load(Yii::$app->request->post(), '');
    if (!$form->validate()) {
        return ['success' => 0, 'error_code' => 'VALIDATION_FAILED'];
    }
    $result = Service::doSomething($form);    // 4. Delegate
    return $result;                           // 5. Return
}
```

#### JavaScript Modules

```javascript
(function ($) {
    'use strict';
    var $root = $('#module-root');
    if (!$root.length) return;

    // Parse config from DOM
    var config = JSON.parse($root.attr('data-config') || '{}');

    // State
    var state = { mode: 'idle' };

    // Private functions
    function render() { /* ... */ }
    function handleScan() { /* ... */ }

    // Event bindings
    $root.on('click', '#action-btn', handleScan);

    // Init
    render();
})(jQuery);
```

### Step 4: Refactoring Strategies

#### Extract and Delegate
When a method grows too large:
1. Identify self-contained blocks
2. Extract to private methods with clear names
3. Keep the orchestration method readable

#### Bridge Pattern for Breaking Changes
When changing a method signature:
```php
// Old method — add @deprecated, delegate to new
/** @deprecated Use newMethod() instead */
public function oldMethod($a, $b) {
    return $this->newMethod(['param_a' => $a, 'param_b' => $b]);
}

// New method — clean interface
public function newMethod(array $context) { /* ... */ }
```

#### CSS Migration
When moving from inline to shared:
1. Create shared file with new class names
2. Add backward-compatible aliases for old names
3. Update views to register shared file
4. Remove inline styles

## Code Quality Checklist

Before marking any implementation complete:

```
□ All inputs validated and normalized
□ All error paths return deterministic error codes
□ No business logic in controllers
□ No hardcoded values without config fallback
□ PHPDoc on all public methods
□ Consistent naming with project conventions
□ Tests written for happy path + error paths
□ No console.log or var_dump left in code
□ All DOM IDs unique and follow naming convention
□ All event handlers properly namespaced
```

## Yii2-Specific Patterns

| Pattern | Example |
|---------|---------|
| Config access | `Yii::$app->params['key']` with `isset()` guard |
| Session storage | `Yii::$app->session->get('key', default)` |
| URL generation | `Url::to(['controller/action', 'param' => 'value'])` |
| JSON response | `Yii::$app->response->format = Response::FORMAT_JSON` |
| RBAC check | `$user->hasPermission('permission_code')` |
| Flash messages | `Yii::$app->session->setFlash('type', 'message')` |
| Asset registration | `$this->registerJsFile('@web/js/file.js', ['depends' => [JqueryAsset::class]])` |
