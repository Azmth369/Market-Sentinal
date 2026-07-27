# Module 6 — Handoff

## Current module

Module 6 — Source Manager.

## Frozen tasks

* Task 6.1 — Core Domain \& Repository Layer (re-confirmed frozen after
the Task 6.2 cleanup; `core/src/interfaces/source-manager-repository.ts`
is byte-for-byte identical to its original Task 6.1 state).
* Task 6.2 — CRUD API (Sources), pending your final green-light per
`docs/module-6/task-6.2.md`.

## Completed tasks

* Task 6.1 — domain entities, repository interfaces
(`CategoryRepository`, `KeywordGroupRepository`,
`SourceManagerRepository`), service interfaces, Zod schemas.
* Task 6.2 — `SourceManagerService`, `PrismaSourceManagerRepository`,
`SourceNotFoundError`/`SourceCategoryReferenceError`, `ValidationError`.
Freeze violation from this task (error classes edited directly into
the frozen Task 6.1 interface file) has been found and fixed — see
`docs/module-6/task-6.2.md`.

## Outstanding architectural decisions

* **"Assign keyword groups to source" (Task 6.3):** no `Source`↔
`KeywordGroup` relation exists in the Phase 1 schema. Needs a schema
decision (join table vs. staying global) before Task 6.3 can implement
this roadmap line item.
* **`SourceCategoryReferenceError` HTTP status:** 400 or 404, once
Task 6.5 builds the HTTP layer.
* **Delete idempotency:** `deleteSource` on an already-deleted `id`
currently throws `SourceNotFoundError` rather than no-op succeeding.
Confirm this is the intended behavior before Task 6.5 exposes it over
HTTP.
* **Service interface placement:** `SourceService`/`CategoryService`/
`KeywordGroupService` live in `packages/source-manager`, not `core`,
on the assumption a future Dashboard (Module 7) will talk to Source
Manager over HTTP rather than importing it directly. Confirm this
still matches intent before Task 6.5.

## Next task

Task 6.3 — Categories \& Keyword Groups (Category CRUD, Keyword Group
CRUD, assign category to source, assign keyword groups to source — the
last item blocked on the schema decision above). Not started.

## Known constraints

* This sandbox has no network access: `pnpm install` cannot run, so
`zod`, `@prisma/client`, and other external deps aren't installed
here. Verification has relied on `tsc --noEmit` (with temporary,
unshipped type stubs where needed) and `node --test` via manually
symlinked workspace packages. Files depending on `zod` fail to load
at runtime in this sandbox for that reason — confirmed pre-existing,
not caused by any change in this session.
* No repository implementation of `CategoryRepository` or
`KeywordGroupRepository` exists yet — only `SourceManagerRepository`
has a Prisma implementation so far (Task 6.2).
* No HTTP routes exist yet for any Source Manager operation — that's
Task 6.5.
* Per the roadmap, `project\_status.md` and section numbering are not to
be updated until all of Module 6 (6.1–6.5) is complete.

