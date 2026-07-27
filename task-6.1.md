# Module 6 — Task 6.1: Core Domain & Repository Layer

**Status: Frozen** (originally frozen after Task 6.1 review; re-confirmed
frozen after the Task 6.2 cleanup documented in `task-6.2.md`).

## Goal (per roadmap)

Build the foundation: domain contracts and interfaces only. No UI, no
API, no database wiring, no application wiring.

## What was delivered

**`@market-sentinel/core` additions** (existing package, extended):

| File | Change |
|---|---|
| `entities/category.ts` | New — `Category`, `CreateCategoryInput`, `UpdateCategoryInput` |
| `entities/keyword.ts` | Extended — `KeywordGroup`, `CreateKeywordGroupInput`, `UpdateKeywordGroupInput` |
| `entities/source.ts` | Extended — `NotifyMode`, `ManagedSource`, `CreateSourceInput`, `UpdateSourceInput` |
| `interfaces/category-repository.ts` | New — `CategoryRepository` |
| `interfaces/keyword-group-repository.ts` | New — `KeywordGroupRepository` |
| `interfaces/source-manager-repository.ts` | New — `SourceManagerRepository` (see design note below) |
| `interfaces/source-repository.ts` | One doc-comment line added; **no shape change** |
| `index.ts` | Extended to export all of the above |

**New package `@market-sentinel/source-manager`:**

| File | Purpose |
|---|---|
| `src/validation/source-schema.ts` | Zod schema mirroring `CreateSourceInput`/`UpdateSourceInput`, including the two cross-field rules (`digestWindowSec` required when `notifyMode === 'digest'`; `extractionConfig` shape required when `sourceType === 'html_listing'`) |
| `src/validation/category-schema.ts` | Zod schema for categories |
| `src/validation/keyword-group-schema.ts` | Zod schema for keyword groups |
| `src/interfaces/source-service.ts` | `SourceService` contract — **no implementation yet** |
| `src/interfaces/category-service.ts` | `CategoryService` contract — **no implementation yet** |
| `src/interfaces/keyword-group-service.ts` | `KeywordGroupService` contract — **no implementation yet** |
| `test/validation/*.test.ts` | 40 tests across the three schemas |
| `test/interfaces/service-shapes.test.ts` | 3 tests — hand-written in-memory fakes proving the three service interfaces are usable, not just type-correct |

## Key design decisions

1. **`SourceManagerRepository` is a separate interface from
   `SourceRepository`, not an extension of it.** `packages/db` already
   has a frozen, implemented `PrismaSourceRepository` for the Scheduler
   (Module 4). Adding required CRUD methods directly to
   `SourceRepository` would break that class's type-checking — a change
   to a frozen module, which this task's own constraints rule out. A
   second interface adds exactly the five methods the Source Manager
   needs (`findById`, `list`, `create`, `update`, `delete`) without
   touching anything Module 4 owns.

2. **`KeywordGroupRepository` is separate from `KeywordRepository`.**
   `KeywordRepository.getAllKeywords()` (Module 3) has no reason to know
   about group boundaries; the Keyword Engine has no reason to do CRUD
   on groups. Two interfaces keep each consumer depending on exactly
   what it uses.

3. **Service interfaces (`SourceService`/`CategoryService`/
   `KeywordGroupService`) live in `packages/source-manager`, not
   `core`.** Nothing outside this package is expected to import them
   directly — a future Dashboard (Module 7) would talk to the Source
   Manager over its HTTP API, not by importing package internals.

4. **Flagged, not resolved:** the roadmap's Task 6.3 line "assign
   keyword groups to source" has no supporting schema relation yet
   (`Source`↔`KeywordGroup` isn't modeled in Phase 1). `KeywordGroupService`
   deliberately has no assignment method — this needs a schema decision
   before Task 6.3, not a unilateral guess.

## Verification

- `packages/core` type-checks cleanly (`tsc --noEmit`).
- `packages/source-manager/src` type-checks cleanly against
  `@market-sentinel/core` and `zod`.
- **43/43 tests pass** via `node --test` (23 source-schema, 8
  category-schema, 9 keyword-group-schema, 3 service-interface shape
  tests).
- Diffed byte-for-byte against the Module 4 baseline: `packages/db`,
  `packages/scheduler`, `packages/fetchers`, `packages/event-bus`,
  `packages/change-detection`, `packages/keyword-engine`, and
  `project_status.md` are all untouched.

## Freeze confirmation

As of this cleanup pass (see `task-6.2.md`), every file in this table is
confirmed either unchanged since Task 6.1's original freeze, or — in the
one case where it briefly wasn't (`interfaces/source-manager-repository.ts`,
edited during Task 6.2) — restored to be byte-for-byte identical to its
original Task 6.1 content. **Task 6.1 is fully frozen.**
