---
name: Software Architecture
description: System design, component decomposition, dependency analysis, and architectural decision-making for this legacy PHP 5.6 Yii application. Use when planning new modules, integrating features with existing layers, or deciding service/DAO/controller boundaries.
---

# Software Architecture Skill

Use this skill when designing new systems, decomposing features into components, or making architectural decisions.

## Project Constraints

- This codebase is a legacy PHP 5.6 environment. Architecture decisions must preserve PHP 5.6-compatible implementation patterns.
- Prefer additive service-layer changes over broad rewrites.
- Keep scope aligned with the requested task. Do not expand backend tasks into UI or permission systems unless the specification requires that integration.

## Workflow

### 1. Gather Context

Identify:

- what problem the feature solves
- who consumes it
- what data flows in and out
- what existing patterns already solve similar work
- what constraints exist: framework, database, performance, compatibility

Always read existing code first.

### 2. Decompose by Layer

Follow repository conventions:

- controller: thin input/output layer
- service: business orchestration and rules
- DAO: data access only
- DTO/form model: validation and transport where needed
- service-to-DAO access: use `DbWorker::Instance(DaoClass::class)` from Services instead of direct DAO AR/query calls

### 3. Map Dependencies

List:

- upstream dependencies
- downstream consumers
- side effects such as status changes or logs
- failure modes for each dependency

### 4. Define Contracts First

Specify:

- inputs
- outputs
- deterministic errors
- state transitions
- backward compatibility expectations
- PHP 5.6 implementation limits

### 5. Use the Project Resolution Pattern

Apply:

```text
runtime settings -> code constants/maps -> sensible defaults
```

Do not let missing configuration crash normal business flow unless the task explicitly requires a hard failure.

## Anti-Patterns to Avoid

- business logic in controllers
- generic error messages where deterministic codes are required
- broad rewrites for small task-scoped changes
- importing adjacent tasks into the same change
- introducing PHP 7+ syntax into legacy files
- creating new patterns where existing service/DAO/controller patterns already fit
- direct ActiveRecord access from Services when a DAO + `DbWorker` path should own the query
