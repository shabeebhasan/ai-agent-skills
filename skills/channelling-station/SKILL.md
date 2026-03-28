---
name: Channelling Station Development
description: Build or update channelling sorting station backend and UI flows in IMSOFT. Use when implementing PreSort, PostQC, Rework, FinalSort, or related scan/confirm/delivery workflows from task specifications. Apply PHP 5.6 compatibility and keep scope aligned to the explicit task contract.
---

# Channelling Station Development Skill

Use this skill for channelling station work in IMSOFT.

## Scope Guard

- Follow the task specification literally.
- If the task only defines backend services/endpoints, implement only backend services/endpoints.
- Do not add station pages, JS, CSS, form models, permissions, or print/share hooks unless they are explicitly part of the task contract.
- Keep all new PHP compatible with PHP 5.6.

## Core Workflow

### 1. Read the Task Contract

Extract:

- station name / mode
- endpoint contracts
- required payload fields
- deterministic error codes
- status transitions
- dependencies on upstream tasks

### 2. Decide the Required Surface

For backend-only tasks, typical files are:

- `controllers/ChannellingController.php`
- `models/service/channelling/...`
- `models/service/auftrag/...`
- `models/dto/DtoAuftrag.php`
- `models/dao/...`
- `migrations/sql/...`

For full station tasks, additional files may include:

- `models/view/form/channelling/...`
- `views/channelling/...`
- `web/js/...`
- `web/css/...`
- functional/unit tests

### 3. Backend Implementation Rules

- Use T-spec source-of-truth services rather than duplicating routing logic.
- Return deterministic `error_code` values.
- Keep controllers thin and delegate to service methods.
- In station Services, call DAO lookups through `DbWorker::Instance(DaoClass::class)` rather than direct DAO ActiveRecord/query methods.
- Add PHPDoc to public station service methods so endpoint/service contracts stay explicit in legacy files.
- Use `ServiceAuftrag::setAuftragStatusValidated()` when the task requires validated transitions.
- Keep delivery-list behavior aligned with the task: transient station list, export, create-note, clear.

### 4. UI Implementation Rules

Only apply this section when the task explicitly requires UI.

- Reuse existing channelling station patterns and assets.
- Keep scanner-first interaction paths.
- Use deterministic IDs and endpoint wiring.

## Full Station Checklist

When the task includes the full station surface, verify:

```text
page route exists
scan endpoint exists
confirm endpoint exists
delivery list/export/create/clear endpoints exist
bootstrap config or equivalent view wiring exists
frontend renders required route/status metadata
tests cover happy path and deterministic errors
```

## Backend-Only Checklist

When the task is backend-only, verify:

```text
controller endpoints wired
service contract implemented
required option/status aliases added
validated status logic enforced where required
error contract deterministic
no out-of-scope UI or permission work added
PHP 5.6 compatibility reviewed
```
