---
name: AI-Assisted Coding
description: Specification-driven development, systematic code generation, refactoring strategies, and implementation workflow for this PHP 5.6 legacy project. Use when implementing from task specs, adding services/controllers/DAO code, refactoring within existing conventions, or preparing project-aligned test plans.
---

# AI-Assisted Coding Skill

Use this skill for writing production code efficiently using specification-driven approaches and repository-native patterns.

## Project-Specific Constraints

- This repository targets PHP 5.6. Do not introduce PHP 7+ syntax.
- Do not add `\Throwable`, scalar type hints, return types, typed properties, `??`, or other newer shorthand.
- Prefer explicit `isset()`, ternaries, and `catch (\Exception $e)`.
- Stay inside the active task scope. If the task is backend-only, do not add UI, assets, or permission layers unless the spec requires them.

## Core Method: Spec -> Plan -> Code -> Test

### Step 1: Parse the Specification

Extract:

```text
input contract
output contract
state transitions
dependencies
acceptance criteria
```

### Step 2: Create an Implementation Plan

Before writing code, identify:

1. File list
2. Dependency order
3. Change size
4. Test plan

### Step 3: Use Project-Aligned Code Patterns

#### Service Methods

```php
/**
 * Brief description.
 * @param mixed $input Description
 * @return array
 */
public static function doSomething($input) {
    // 1. Normalize input
    // 2. Load dependencies/data
    // 3. Apply business rules
    // 4. Return deterministic result
}
```

Rules:

- Prefer static methods for stateless operations.
- Normalize inputs early.
- Return deterministic arrays for business outcomes.
- Keep orchestration readable and extract helpers when blocks become independent.
- Add concise PHPDoc for public methods you introduce or materially change, including `@param` and `@return`.

#### Controller Actions

```php
public function actionDoSomething() {
    Yii::$app->response->format = Response::FORMAT_JSON;
    $payload = Yii::$app->request->post();
    return ServiceExample::doSomething($payload);
}
```

Rules:

- Keep controllers thin.
- Validate/normalize request data before delegating when the endpoint contract needs it.
- Do not move business logic into controllers.

#### Refactoring Strategy

- Extract and delegate instead of rewriting large files.
- Keep existing callers working.
- Add bridge methods only when compatibility requires them.

Example:

```php
/** @deprecated Use newMethod() instead. */
public function oldMethod($a, $b) {
    return $this->newMethod([
        'param_a' => $a,
        'param_b' => $b,
    ]);
}

public function newMethod($context) {
    // ...
}
```

## Code Quality Checklist

Before marking work complete:

```text
all inputs validated and normalized
all error paths deterministic
no business logic in controllers
no hardcoded values without fallback strategy
no PHP 7+ syntax introduced
consistent naming with project conventions
tests or a concrete test plan provided
no debug leftovers
```

## Yii / Project Patterns

- Config access: `isset(Yii::$app->params['key']) ? Yii::$app->params['key'] : $default`
- JSON response: `Yii::$app->response->format = Response::FORMAT_JSON`
- Session access: `Yii::$app->session->get('key', $default)`
- RBAC check: `$user->hasPermission('permission_code')`
- Service layer: `models/service/`
- DAO layer: `models/dao/`
- In Services, load DAO operations via `DbWorker::Instance(DaoClass::class)` instead of calling DAO ActiveRecord/query methods directly.

## PHP 5.6 Review Pass

Before finishing any PHP change, scan for:

```bash
rg -n "Throwable|\\?\\?|function .*:\\s|public .*\\$.*:|private .*\\$.*:" path/to/files
```

Then manually review any new method signatures for legacy-incompatible syntax.
