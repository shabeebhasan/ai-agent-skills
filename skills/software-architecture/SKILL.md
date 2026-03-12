---
name: Software Architecture
description: System design, component decomposition, dependency analysis, and architectural decision-making for scalable applications
---

# Software Architecture Skill

Use this skill when designing new systems, decomposing features into components, analyzing dependencies, or making architectural decisions.

## When to Activate

- User asks to "design", "architect", or "plan" a feature or system
- New module/service needs to integrate with existing codebase
- Refactoring requires understanding component boundaries
- Performance or scalability concerns arise

## Workflow

### 1. Context Gathering

Before designing anything, gather:

```
□ What problem does this solve?
□ Who are the consumers (API callers, UI, other services)?
□ What data flows in and out?
□ What existing patterns does the project use?
□ What are the constraints (framework, DB, performance)?
```

**Always read existing code first.** Search for:
- Similar features already implemented
- Service layer patterns in `models/service/`
- Controller patterns in `controllers/`
- View/JS patterns in `views/` and `web/js/`

### 2. Component Decomposition

Break the feature into layers following project convention:

```
┌─────────────────────────────────────────────┐
│  Controller (thin)                          │
│  → Validate input                           │
│  → Delegate to service                      │
│  → Format response                          │
├─────────────────────────────────────────────┤
│  Service (business logic)                   │
│  → Orchestrate operations                   │
│  → Apply business rules                     │
│  → Return deterministic results             │
├─────────────────────────────────────────────┤
│  DAO (data access)                          │
│  → ActiveRecord models                      │
│  → Query builders                           │
│  → No business logic                        │
├─────────────────────────────────────────────┤
│  Form Model (input validation)              │
│  → Scenarios and rules                      │
│  → Input sanitization                       │
│  → Not business logic                       │
└─────────────────────────────────────────────┘
```

### 3. Dependency Analysis

Map dependencies before writing code:

```
Feature X
├── depends on: [list upstream services/data]
├── consumed by: [list downstream callers]
├── side effects: [status changes, notifications, logs]
└── failure modes: [what happens when each dependency fails]
```

### 4. Interface Contract Design

Define the contract before implementation:

- **Input**: exact fields, types, validation rules
- **Output**: success response shape, error response shape
- **Error codes**: deterministic, mappable to user-facing messages
- **State transitions**: from_status → to_status with preconditions

### 5. Resolution Pattern

Apply the project's 3-tier resolution for configurable values:

```
1. Runtime config (Yii::$app->params)     — operator can change
2. Code constants / hardcoded maps         — developer controls
3. Sensible defaults                       — always works
```

Never let missing configuration crash the system.

### 6. Backward Compatibility Strategy

When modifying existing code:

- **Bridge methods**: New method with new signature + old method delegates to new
- **CSS aliases**: New class names + old class names point to same styles
- **API versioning**: New fields added, old fields never removed
- **Deprecation markers**: PHPDoc `@deprecated` with migration instructions

### 7. Documentation

Every architectural decision produces:
- Implementation plan (artifact) with file-level change descriptions
- Dependency diagram if >3 components involved
- Error taxonomy for all new error codes
- Test coverage plan

## Anti-Patterns to Avoid

| Anti-Pattern | Instead Do |
|---|---|
| Business logic in controllers | Move to service layer |
| Generic error messages | Use deterministic error codes |
| Hardcoded values without fallback config | Use 3-tier resolution |
| New patterns when existing ones work | Match project conventions |
| Full file rewrites for small changes | Targeted edits only |
| Untested code | Every feature gets tests |
