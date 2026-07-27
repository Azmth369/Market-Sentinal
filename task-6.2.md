# Module 6 — Task 6.2: CRUD API (Sources)

**Status: Frozen**, following the cleanup documented below.

## Goal (per roadmap)

Make sources manageable: Create, Update, Delete, Get, List. Validation
and error handling included. CRUD operations only — no templates
(Task 6.4), no HTTP route registration (Task 6.5).

## What was delivered

**`@market-sentinel/core` additions:**

|Item|Original location (as shipped)|Corrected location (after cleanup)|
|-|-|-|
|`SourceNotFoundError`|`interfaces/source-manager-repository.ts` (edited directly into the frozen Task 6.1 file)|`errors/source-manager-errors.ts` (new, dedicated file)|
|`SourceCategoryReferenceError`|same|same|

Thrown by `PrismaSourceManagerRepository`: `SourceNotFoundError` when
`update`/`delete` targets a nonexistent `id`; `SourceCategoryReferenceError`
when `create`/`update` references a `categoryId` that doesn't exist
(translating Prisma's `P2025`/`P2003` into stable domain types, the same
pattern as `DuplicateContentError` in `event-repository.ts`).

**`@market-sentinel/db` addition** (new file, `PrismaSourceRepository`
untouched):

* `src/repositories/source-manager-repository.ts` —
`PrismaSourceManagerRepository implements SourceManagerRepository`.
Maps the full `Source` row to `ManagedSource`. Catches Prisma's
`P2025`/`P2003` and re-throws the two domain errors above.
* `src/index.ts`, `src/repositories/index.ts` — extended to export
`PrismaSourceManagerRepository`.

**`@market-sentinel/source-manager` additions:**

* `src/errors.ts` — `ValidationError` (generic, reusable by Task 6.3's
`CategoryService`/`KeywordGroupService`) + `toValidationError()`,
converting a zod `ZodError` into structured `{ path, message }` issues.
* `src/services/source-manager-service.ts` — `SourceManagerService implements SourceService` (Task 6.1's interface). Validates with Task
6.1's zod schemas, delegates persistence to an injected
`SourceManagerRepository` (dependency inversion — never imports
`@market-sentinel/db`/Prisma directly). Does not catch or re-wrap
`SourceNotFoundError`/`SourceCategoryReferenceError` — those already
are the right thing for a caller to catch.
* `src/index.ts` — extended to export `SourceManagerService` and the
error utilities.
* `package.json` — test script glob widened to also pick up
`test/\*\*/\*.test.ts` (new nested test directories).

## Error handling model

|Failure|Thrown by|Type|Likely future HTTP status|
|-|-|-|-|
|Input fails a zod rule|`SourceManagerService`|`ValidationError`|400|
|`id` doesn't exist (update/delete)|`PrismaSourceManagerRepository`|`SourceNotFoundError`|404|
|`categoryId` doesn't exist (create/update)|`PrismaSourceManagerRepository`|`SourceCategoryReferenceError`|400 or 404 (open question)|

`getSource`/`listSources` never throw for "nothing found" — a missing
single source is a `null` return; an empty list is an empty array.

**Delete is not idempotent, by design:** `deleteSource` on an
already-deleted `id` throws `SourceNotFoundError` rather than silently
succeeding, so a caller always knows whether their call actually removed
something. Flagged as worth confirming before Task 6.5 exposes this over
HTTP.

## Freeze violation found during review, and the fix

Task 6.2 was reviewed as functionally sound, but the review also found
that it had **directly edited `core/src/interfaces/source-manager-repository.ts`**
— a file frozen at the end of Task 6.1 — adding the two error classes
and a few doc-comment lines into it, rather than introducing them in a
new file. This is a process violation independent of the edit's own
merits: once a task is frozen, its files are not supposed to change as
a side effect of later work.

**Cleanup applied** (this pass):

1. Moved `SourceNotFoundError` and `SourceCategoryReferenceError`,
verbatim, into a new file: `core/src/errors/source-manager-errors.ts`.
2. Restored `core/src/interfaces/source-manager-repository.ts` to be
byte-for-byte identical to its original Task 6.1 content (confirmed
via diff against the Task 6.1 snapshot).
3. Updated `core/src/index.ts`'s export statement for the two error
classes to source from the new file.
4. No changes were needed in `packages/db` or
`packages/source-manager`'s test suite — both import the error
classes via `@market-sentinel/core`'s package root (`from '@market-sentinel/core'`), never via a direct path into the interface
file, so relocating the classes' *internal* home doesn't change the
package's *external* API surface at all.
5. Updated `packages/source-manager/README.md`'s Task 6.2 section to
describe the corrected file location, and added a "Post-hoc cleanup"
note explaining what happened and what changed.

**No functional/behavioral change resulted from this cleanup.** The two
error classes have the exact same names, constructors, messages, and
`instanceof` behavior as before — only which file defines them changed.

## Verification

* `@market-sentinel/core`: `tsc --noEmit` — clean, both before and after
the cleanup.
* `@market-sentinel/source-manager`: test suite re-run before and after
the cleanup, offline (`node --test` via `tsx`, using manually-linked
workspace packages since this sandbox has no network access to run
`pnpm install`). Identical results both times: 3 tests pass
(service-interface shape tests + the fake-repository round-trip
tests), 4 test files fail to even load due to `Cannot find module 'zod'` — confirmed this exact failure exists identically in the
*unmodified* Task 6.2 snapshot too, i.e. it's a pre-existing sandbox
limitation (no network access to install `zod`), not something this
cleanup caused or could have fixed.
* `@market-sentinel/db`: `source-manager-repository.ts` untouched by
this cleanup; not re-verified beyond confirming no import in it needed
to change.

## Confirmed unmodified by this cleanup

`packages/scheduler`, `packages/fetchers`, `packages/event-bus`,
`packages/change-detection`, `packages/keyword-engine`,
`packages/notifications`, `project\_status.md` — none touched by either
Task 6.2 or this cleanup pass.

## Freeze confirmation

`core/src/interfaces/source-manager-repository.ts` is now, again,
byte-for-byte identical to its original Task 6.1 state. **Task 6.1 is
fully frozen.** Task 6.2's actual deliverable (CRUD service, Prisma
repository, error types, tests) is unchanged in behavior and is now
itself ready to freeze, with the process violation corrected.

