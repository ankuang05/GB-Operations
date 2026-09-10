# Plan-Orchestrate Result

**Plan**: `docs/ROADMAP.md`
**Lang**: `typescript` *(explicit — the greenfield repo has no detectable markers yet; stack fixed by EDD §2.2)*
**ECC mode**: `plugin`
**Steps**: 24
**Scope**: all

> Generative only. Nothing below has been executed. Paste one line when you are ready to start that step.

---

## Steps overview

| # | Title | Sprint | Tags | Chain |
|---|---|---|---|---|
| 1 | Monorepo scaffold, tooling & CI | S0 | impl, build | `ecc:tdd-guide,ecc:build-error-resolver,ecc:typescript-reviewer` |
| 2 | Supabase provisioning, Drizzle & migration 001 | S1 | impl, db | `ecc:tdd-guide,ecc:database-reviewer,ecc:typescript-reviewer` |
| 3 | Supabase Auth flows & JWT tenant claims | S1 | impl, security | `ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer` |
| 4 | RLS policies & cross-tenant test suite | S1 | test, db, security | `ecc:tdd-guide,ecc:database-reviewer,ecc:security-reviewer` |
| 5 | tRPC setup & role-guarded middleware | S1 | impl, security | `ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer` |
| 6 | Design system, shared primitives & i18n | S1 | design, impl | `ecc:architect,ecc:tdd-guide,ecc:typescript-reviewer` |
| 7 | User administration UI | S1 | impl, security | `ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer` |
| 8 | Migration 002 — task tables | S2 | impl, db | `ecc:tdd-guide,ecc:database-reviewer,ecc:typescript-reviewer` |
| 9 | Recurrence engine & nightly materialization | S2 | impl | `ecc:tdd-guide,ecc:typescript-reviewer` |
| 10 | tRPC task procedures | S2 | impl | `ecc:tdd-guide,ecc:typescript-reviewer` |
| 11 | Web task authoring & template library | S2 | impl | `ecc:tdd-guide,ecc:typescript-reviewer` |
| 12 | Web today/tomorrow board | S2 | impl | `ecc:tdd-guide,ecc:typescript-reviewer` |
| 13 | Expo shell, navigation & auth flow | S3 | impl | `ecc:tdd-guide,ecc:typescript-reviewer` |
| 14 | "My Day" task list | S3 | impl | `ecc:tdd-guide,ecc:typescript-reviewer` |
| 15 | Offline SQLite mirror & outbox sync engine | S3 | design, impl | `ecc:architect,ecc:tdd-guide,ecc:typescript-reviewer` |
| 16 | Complete-task flow with photo evidence | S3 | impl, security | `ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer` |
| 17 | EAS build pipeline & TestFlight delivery | S3 | build | `ecc:build-error-resolver` |
| 18 | Completion aggregation & daily rollups | S4 | impl, db | `ecc:tdd-guide,ecc:database-reviewer,ecc:typescript-reviewer` |
| 19 | Manager visibility dashboards | S4 | impl | `ecc:tdd-guide,ecc:typescript-reviewer` |
| 20 | Employee self-view & Notice at Collection | S4 | impl, security | `ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer` |
| 21 | E2E suites — Playwright & Maestro | S4/S5 | test | `ecc:tdd-guide,ecc:e2e-runner` |
| 22 | Security hardening pass | S5 | security, review | `ecc:security-reviewer,ecc:typescript-reviewer,ecc:code-reviewer` |
| 23 | Performance pass | S5 | db, review | `ecc:database-reviewer,ecc:typescript-reviewer,ecc:code-reviewer` |
| 24 | Documentation & codemaps | S5 | docs | `ecc:doc-updater` |

---

## Step 1 — Monorepo scaffold, tooling & CI

**Intent**: Stand up the pnpm + Turborepo workspace with `apps/web`, `apps/mobile`, and `packages/{types,core,api-client}`, plus the GitHub Actions pipeline that gates every later step.
**Tags**: impl, build
**Chain rationale**: `build-error-resolver` sits mid-chain to get the toolchain green across three workspaces before `typescript-reviewer` closes on config correctness.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:build-error-resolver,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-1] Scaffold a pnpm + Turborepo monorepo with apps/web (Next.js 15 App Router), apps/mobile (Expo), and packages/types, packages/core, packages/api-client. Add GitHub Actions running typecheck, lint, test and a migration dry-run. Acceptance: pnpm dev starts web and mobile locally; CI passes green on a trivial PR; strict TypeScript with no implicit any across all workspaces."
```

## Step 2 — Supabase provisioning, Drizzle & migration 001

**Intent**: Create local/staging/production Supabase projects, wire Drizzle, and land the first migration: `organizations`, `sites`, `profiles`, `audit_log`.
**Tags**: impl, db
**Chain rationale**: `database-reviewer` validates schema shape and migration reversibility before `typescript-reviewer` checks the generated types.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:database-reviewer,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-2] Provision Supabase projects for local, staging and production, wire Drizzle, and write migration 001 creating organizations, sites, profiles and audit_log per ENGINEERING-DESIGN.md section 3.1. Every table carries organization_id. Acceptance: migration applies and rolls back cleanly on an empty database; Drizzle types generate for all four tables; audit_log has no UPDATE or DELETE grant to the application role."
```

## Step 3 — Supabase Auth flows & JWT tenant claims

**Intent**: Email/password login, logout, password reset and session refresh across web and mobile, with `organization_id` and `role` attached as JWT custom claims.
**Tags**: impl, security
**Chain rationale**: `security-reviewer` closes the chain — this is the credential path and the source of the claims every RLS policy trusts.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-3] Implement Supabase Auth login, logout, password reset and session refresh shared by apps/web and apps/mobile, attaching organization_id and role as JWT custom claims at sign-in. Roles are ceo, manager, sales, warehouse, driver, customer. Acceptance: all six roles authenticate on both surfaces; claims are present and correct in the decoded JWT; no credential or token is ever written to logs or local storage in plaintext."
```

## Step 4 — RLS policies & cross-tenant test suite

**Intent**: Enable Row-Level Security on every table with a uniform tenant-isolation policy plus role-specific read rules, and build the automated suite that proves isolation holds.
**Tags**: test, db, security
**Chain rationale**: This is PRD acceptance criterion 3 — the one requirement that cannot be retrofitted. `e2e-runner` was dropped from the default `test` chain because isolation is verified by SQL-level integration tests, not browser flows; `security-reviewer` closes instead.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:database-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-4] Enable RLS on every table with a uniform organization_id isolation policy plus role-specific read rules per ENGINEERING-DESIGN.md section 4.2, and write a cross-tenant test suite that creates two organizations and asserts every table under every role returns zero rows across the boundary. Acceptance: 100 percent table coverage in the suite; the suite gates CI and blocks merge when red; warehouse role reads only its own or its site group tasks."
```

## Step 5 — tRPC setup & role-guarded middleware

**Intent**: Stand up tRPC over Next.js Route Handlers with procedure-level role guards, consumed identically by web and mobile.
**Tags**: impl, security
**Chain rationale**: The guard layer is authorization logic, so `security-reviewer` closes; `typescript-reviewer` first checks that inference survives the client boundary.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-5] Set up tRPC on Next.js Route Handlers with role-guarded procedure middleware, exported through packages/api-client so apps/web and apps/mobile call identical procedures. Acceptance: end-to-end type inference works from server procedure to mobile call site with no codegen step; a procedure guarded to manager rejects a warehouse JWT with 403; guard failures are audit-logged."
```

## Step 6 — Design system, shared primitives & i18n

**Intent**: Shared design tokens rendered through Tailwind on web and NativeWind on mobile, with English/Spanish i18n scaffolding from the start.
**Tags**: design, impl
**Chain rationale**: `architect` sets the token boundary between the two renderers before implementation; `planner` is deliberately omitted since this is an implementation step, not a planning one.

```bash
/ecc:orchestrate custom "ecc:architect,ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-6] Define shared design tokens consumed by Tailwind on apps/web and NativeWind on apps/mobile, build the base primitive set, and scaffold i18n with English and Spanish string extraction. Acceptance: tokens live in a single shared package and both surfaces consume them without duplication; every user-facing string is externalized with no hardcoded copy; mobile primitives meet a 44pt minimum tap target per NFR-2."
```

## Step 7 — User administration UI

**Intent**: Let a CEO or manager invite users, assign role and site, and deactivate accounts.
**Tags**: impl, security
**Chain rationale**: Privilege assignment is the classic escalation surface, so `security-reviewer` closes.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-7] Build the web user administration UI for inviting users, assigning role and site, and deactivating accounts, restricted to ceo and manager roles. Acceptance: an invited warehouse employee lands on a role-correct empty state on first login; a manager cannot grant a role above their own or assign a user to another organization; every administrative mutation writes to audit_log."
```

## Step 8 — Migration 002: task tables

**Intent**: Add `task_templates`, `tasks`, and the append-only `task_completions` table carrying both client and server timestamps.
**Tags**: impl, db
**Chain rationale**: `database-reviewer` validates the append-only constraint and index plan; `typescript-reviewer` closes on generated types.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:database-reviewer,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-8] Write migration 002 adding task_templates, tasks and task_completions per ENGINEERING-DESIGN.md section 3.1, with task_completions append-only and carrying both client_completed_at and sync_received_at. Acceptance: no UPDATE or DELETE grant exists on task_completions for the application role; RLS policies from step 4 extend to all three tables; indexes support querying today's tasks by site and by assignee."
```

## Step 9 — Recurrence engine & nightly materialization

**Intent**: RFC 5545 RRULE evaluation turning templates into concrete dated tasks via a nightly job.
**Tags**: impl
**Chain rationale**: Pure business logic in `packages/core` with dense edge cases — TDD first, `typescript-reviewer` closes.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-9] Implement an RFC 5545 RRULE recurrence engine in packages/core plus the nightly job that materializes task_templates into dated task rows per site timezone. Acceptance: templates materialize correctly across a US daylight-saving transition in both directions; re-running the job for the same date is idempotent and creates no duplicates; a deactivated template stops materializing without affecting existing tasks."
```

## Step 10 — tRPC task procedures

**Intent**: The task API surface — create, read, update, assign, bulk create, complete — shared by both clients.
**Tags**: impl
**Chain rationale**: This is the contract between web and mobile; `typescript-reviewer` closes on the shared types.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-10] Implement tRPC task procedures for create, read, update, assign, bulk create and complete, with role guards from step 5 applied. Completing a task inserts into task_completions rather than mutating history. Acceptance: a warehouse role can complete only tasks assigned to them or their site group; bulk create of 50 tasks completes in a single transaction; every mutation writes a before and after snapshot to audit_log."
```

## Step 11 — Web task authoring & template library

**Intent**: The manager's primary workflow — build tomorrow's list in under five minutes.
**Tags**: impl
**Chain rationale**: Straight feature work against an existing API; the reviewer closes on component structure and form correctness.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-11] Build the web task authoring form supporting one-off and recurring tasks assigned to a person or a site group with due time and priority, plus a reusable template library to save, reuse and edit checklists. Acceptance: a manager builds a full next-day task list for one warehouse in under 5 minutes measured with a real GB manager; recurring tasks preview their next 5 occurrences before saving; the form is fully usable in Spanish."
```

## Step 12 — Web today/tomorrow board

**Intent**: The manager's at-a-glance view of assigned work with drag-to-reassign.
**Tags**: impl
**Chain rationale**: Interaction-heavy UI; the reviewer closes on render performance and state handling.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-12] Build the web today/tomorrow task board grouped by assignee with drag-to-reassign. Acceptance: reassignment persists optimistically and rolls back visibly on server rejection; the board renders 200 tasks without dropped frames; reassignment is also reachable through a non-drag control for keyboard and screen-reader users."
```

## Step 13 — Expo shell, navigation & auth flow

**Intent**: The iOS app skeleton — two tabs only, per ADR-006.
**Tags**: impl
**Chain rationale**: Foundational mobile work; the reviewer closes on navigation typing and session handling.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-13] Build the Expo app shell with navigation, the shared auth flow from step 3, and exactly two tabs, My Day and Profile, per ADR-006. Acceptance: session persists across cold start and refreshes silently before expiry; unauthenticated launch lands on login with no flash of authenticated content; no placeholder or disabled tabs are visible. Out of scope: chat, ordering and tools tabs, which ship in later releases."
```

## Step 14 — "My Day" task list

**Intent**: What a warehouse employee sees when they open the app in the morning.
**Tags**: impl
**Chain rationale**: The core adoption surface; the reviewer closes on list performance and accessibility.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-14] Build the My Day list showing today's tasks ordered by priority and due time, with clear overdue emphasis and a visible completed count. Acceptance: the list renders from the local mirror with no loading spinner on cold start; overdue tasks are distinguishable without relying on color alone; all tap targets are at least 44pt per NFR-2 and usable wearing warehouse gloves."
```

## Step 15 — Offline SQLite mirror & outbox sync engine

**Intent**: The highest-risk item in R1 — the app must work in Wi-Fi dead zones or the crew stops opening it.
**Tags**: design, impl
**Chain rationale**: `architect` fixes the sync boundary and conflict rule before implementation; the reviewer closes on the async and error paths where sync bugs actually live.

```bash
/ecc:orchestrate custom "ecc:architect,ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-15] Implement the local SQLite mirror of today's tasks and the durable outbox sync engine per ENGINEERING-DESIGN.md section 6, recording client_completed_at at capture and sync_received_at on the server. Conflict rule is completion-wins. Acceptance: a completion made in airplane mode appears on the manager dashboard within 60 seconds of reconnecting; the outbox survives app termination and replays on next launch; replaying the same outbox entry twice creates exactly one completion."
```

## Step 16 — Complete-task flow with photo evidence

**Intent**: Marking work done, with a photo or note as the evidence record.
**Tags**: impl, security
**Chain rationale**: Photo storage means private buckets and signed URLs, so `security-reviewer` closes.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-16] Build the complete-task flow with camera capture, an optional note, and optimistic UI, uploading to a private Supabase Storage bucket through the outbox from step 15. Acceptance: photos are never served from a public path and are reachable only via short-lived signed URLs; capture works offline and uploads on reconnect; both client and server completion timestamps are displayed to the employee and the manager."
```

## Step 17 — EAS build pipeline & TestFlight delivery

**Intent**: Ship builds to real devices from Sprint 3 on, so App Store review is never a surprise in February.
**Tags**: build
**Chain rationale**: Pure toolchain and signing work, gated by its own success criterion — a build reaching a device. No reviewer needed.

```bash
/ecc:orchestrate custom "ecc:build-error-resolver" "[Plan: docs/ROADMAP.md#step-17] Configure the EAS build pipeline with signing credentials and automated TestFlight internal distribution, plus OTA update channels for staging and production. Acceptance: a build reaches TestFlight and installs on a physical iPhone; an OTA JS-only update reaches an installed build without App Store review; staging and production builds are independently addressable and clearly labeled in-app."
```

## Step 18 — Completion aggregation & daily rollups

**Intent**: The query layer behind every dashboard number.
**Tags**: impl, db
**Chain rationale**: Aggregation performance is a database problem first — `database-reviewer` sits mid-chain, `typescript-reviewer` closes.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:database-reviewer,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-18] Build completion aggregation queries and materialized daily rollups for per-site and per-person on-time completion rates. Every aggregate must be traceable to its underlying task rows per RM-2. Acceptance: dashboard queries return in under 200ms at 150 users and 90 days of history; rollups are recomputed idempotently; no aggregate exposes any presence or duration metric, per ADR-004."
```

## Step 19 — Manager visibility dashboards

**Intent**: The screen that answers the CEO's original question.
**Tags**: impl
**Chain rationale**: Feature work over the step 18 query layer; the reviewer closes on data-fetching and render cost.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-19] Build the manager dashboards: a live per-warehouse completion board, per-person history with evidence drill-down, and an overdue queue with a reassign action. Acceptance: the dashboard loads in under 2 seconds on a 4G connection per NFR-5; every displayed number drills through to the specific tasks behind it; the overdue queue updates without a manual refresh."
```

## Step 20 — Employee self-view & Notice at Collection

**Intent**: The transparency guarantees that make this an accountability tool rather than a surveillance tool.
**Tags**: impl, security
**Chain rationale**: This step implements a legal and policy position (RM-3, RM-4), so `security-reviewer` closes on access-control equivalence between the two views.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-20] Build the employee self-view rendering that employee's own metrics from the same components and queries the manager view uses, plus a Notice at Collection screen shown and acknowledged at first login in English and Spanish. Acceptance: employee and manager views of the same person are provably identical with no hidden fields; acknowledgement is recorded with a timestamp; no metric anywhere measures presence, duration or location, per ADR-004."
```

## Step 21 — E2E suites: Playwright & Maestro

**Intent**: Automate the five web and three iOS flows that must never break.
**Tags**: test
**Chain rationale**: `e2e-runner` is the validator for this step and gates it directly; no additional reviewer is required.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:e2e-runner" "[Plan: docs/ROADMAP.md#step-21] Build the Playwright suite covering login, create task, assign, dashboard and deactivate user, plus the Maestro suite covering login, complete task offline and sync. Acceptance: all eight flows pass against a seeded staging environment; the offline Maestro flow genuinely disables the network rather than mocking it; both suites run in CI and quarantine flaky tests rather than failing the build."
```

## Step 22 — Security hardening pass

**Intent**: Pre-pilot audit before real employee data enters the system.
**Tags**: security, review
**Chain rationale**: Audit-only step. `security-reviewer` leads on the threat surface, `code-reviewer` provides a second independent pass.

```bash
/ecc:orchestrate custom "ecc:security-reviewer,ecc:typescript-reviewer,ecc:code-reviewer" "[Plan: docs/ROADMAP.md#step-22] Run the pre-pilot security pass: re-audit every RLS policy against the step 4 suite, verify signed URL expiry on storage, scan dependencies for known vulnerabilities, and confirm no secret or token is logged. Acceptance: zero high or critical dependency findings; every table's RLS policy is confirmed present and correct; no PII appears in Sentry payloads or application logs."
```

## Step 23 — Performance pass

**Intent**: Meet NFR-5 and keep the app feeling instant on the floor.
**Tags**: db, review
**Chain rationale**: The dominant cost is query shape, so `database-reviewer` leads; `code-reviewer` closes on client-side render and startup cost.

```bash
/ecc:orchestrate custom "ecc:database-reviewer,ecc:typescript-reviewer,ecc:code-reviewer" "[Plan: docs/ROADMAP.md#step-23] Profile and optimize dashboard aggregate queries and mobile cold start against a seeded dataset of 150 users and 90 days of history. Acceptance: the manager dashboard loads in under 2 seconds on simulated 4G per NFR-5; mobile cold start to a rendered My Day list is under 2 seconds; no N+1 query pattern remains in any dashboard endpoint."
```

## Step 24 — Documentation & codemaps

**Intent**: Leave the repo navigable — the real mitigation for a two-person bus factor.
**Tags**: docs
**Chain rationale**: Single-purpose documentation step; `doc-updater` owns it end to end.

```bash
/ecc:orchestrate custom "ecc:doc-updater" "[Plan: docs/ROADMAP.md#step-24] Generate codemaps under docs/CODEMAPS covering apps/web, apps/mobile and packages, write the README with local setup and environment variables, and document the runbook for migrations, backup restore and OTA release. Acceptance: a new engineer reaches a running local environment from the README alone; every ADR in ENGINEERING-DESIGN.md is reflected in the codemap structure; the backup restore runbook has been executed at least once."
```

---

## Batch execution

Steps in dependency order. Let each finish before starting the next — later steps build on earlier handoffs.

```bash
/ecc:orchestrate custom "ecc:tdd-guide,ecc:build-error-resolver,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-1] Scaffold a pnpm + Turborepo monorepo with apps/web (Next.js 15 App Router), apps/mobile (Expo), and packages/types, packages/core, packages/api-client. Add GitHub Actions running typecheck, lint, test and a migration dry-run. Acceptance: pnpm dev starts web and mobile locally; CI passes green on a trivial PR; strict TypeScript with no implicit any across all workspaces."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:database-reviewer,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-2] Provision Supabase projects for local, staging and production, wire Drizzle, and write migration 001 creating organizations, sites, profiles and audit_log per ENGINEERING-DESIGN.md section 3.1. Every table carries organization_id. Acceptance: migration applies and rolls back cleanly on an empty database; Drizzle types generate for all four tables; audit_log has no UPDATE or DELETE grant to the application role."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-3] Implement Supabase Auth login, logout, password reset and session refresh shared by apps/web and apps/mobile, attaching organization_id and role as JWT custom claims at sign-in. Roles are ceo, manager, sales, warehouse, driver, customer. Acceptance: all six roles authenticate on both surfaces; claims are present and correct in the decoded JWT; no credential or token is ever written to logs or local storage in plaintext."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:database-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-4] Enable RLS on every table with a uniform organization_id isolation policy plus role-specific read rules per ENGINEERING-DESIGN.md section 4.2, and write a cross-tenant test suite that creates two organizations and asserts every table under every role returns zero rows across the boundary. Acceptance: 100 percent table coverage in the suite; the suite gates CI and blocks merge when red; warehouse role reads only its own or its site group tasks."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-5] Set up tRPC on Next.js Route Handlers with role-guarded procedure middleware, exported through packages/api-client so apps/web and apps/mobile call identical procedures. Acceptance: end-to-end type inference works from server procedure to mobile call site with no codegen step; a procedure guarded to manager rejects a warehouse JWT with 403; guard failures are audit-logged."
/ecc:orchestrate custom "ecc:architect,ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-6] Define shared design tokens consumed by Tailwind on apps/web and NativeWind on apps/mobile, build the base primitive set, and scaffold i18n with English and Spanish string extraction. Acceptance: tokens live in a single shared package and both surfaces consume them without duplication; every user-facing string is externalized with no hardcoded copy; mobile primitives meet a 44pt minimum tap target per NFR-2."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-7] Build the web user administration UI for inviting users, assigning role and site, and deactivating accounts, restricted to ceo and manager roles. Acceptance: an invited warehouse employee lands on a role-correct empty state on first login; a manager cannot grant a role above their own or assign a user to another organization; every administrative mutation writes to audit_log."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:database-reviewer,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-8] Write migration 002 adding task_templates, tasks and task_completions per ENGINEERING-DESIGN.md section 3.1, with task_completions append-only and carrying both client_completed_at and sync_received_at. Acceptance: no UPDATE or DELETE grant exists on task_completions for the application role; RLS policies from step 4 extend to all three tables; indexes support querying today's tasks by site and by assignee."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-9] Implement an RFC 5545 RRULE recurrence engine in packages/core plus the nightly job that materializes task_templates into dated task rows per site timezone. Acceptance: templates materialize correctly across a US daylight-saving transition in both directions; re-running the job for the same date is idempotent and creates no duplicates; a deactivated template stops materializing without affecting existing tasks."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-10] Implement tRPC task procedures for create, read, update, assign, bulk create and complete, with role guards from step 5 applied. Completing a task inserts into task_completions rather than mutating history. Acceptance: a warehouse role can complete only tasks assigned to them or their site group; bulk create of 50 tasks completes in a single transaction; every mutation writes a before and after snapshot to audit_log."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-11] Build the web task authoring form supporting one-off and recurring tasks assigned to a person or a site group with due time and priority, plus a reusable template library to save, reuse and edit checklists. Acceptance: a manager builds a full next-day task list for one warehouse in under 5 minutes measured with a real GB manager; recurring tasks preview their next 5 occurrences before saving; the form is fully usable in Spanish."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-12] Build the web today/tomorrow task board grouped by assignee with drag-to-reassign. Acceptance: reassignment persists optimistically and rolls back visibly on server rejection; the board renders 200 tasks without dropped frames; reassignment is also reachable through a non-drag control for keyboard and screen-reader users."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-13] Build the Expo app shell with navigation, the shared auth flow from step 3, and exactly two tabs, My Day and Profile, per ADR-006. Acceptance: session persists across cold start and refreshes silently before expiry; unauthenticated launch lands on login with no flash of authenticated content; no placeholder or disabled tabs are visible. Out of scope: chat, ordering and tools tabs, which ship in later releases."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-14] Build the My Day list showing today's tasks ordered by priority and due time, with clear overdue emphasis and a visible completed count. Acceptance: the list renders from the local mirror with no loading spinner on cold start; overdue tasks are distinguishable without relying on color alone; all tap targets are at least 44pt per NFR-2 and usable wearing warehouse gloves."
/ecc:orchestrate custom "ecc:architect,ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-15] Implement the local SQLite mirror of today's tasks and the durable outbox sync engine per ENGINEERING-DESIGN.md section 6, recording client_completed_at at capture and sync_received_at on the server. Conflict rule is completion-wins. Acceptance: a completion made in airplane mode appears on the manager dashboard within 60 seconds of reconnecting; the outbox survives app termination and replays on next launch; replaying the same outbox entry twice creates exactly one completion."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-16] Build the complete-task flow with camera capture, an optional note, and optimistic UI, uploading to a private Supabase Storage bucket through the outbox from step 15. Acceptance: photos are never served from a public path and are reachable only via short-lived signed URLs; capture works offline and uploads on reconnect; both client and server completion timestamps are displayed to the employee and the manager."
/ecc:orchestrate custom "ecc:build-error-resolver" "[Plan: docs/ROADMAP.md#step-17] Configure the EAS build pipeline with signing credentials and automated TestFlight internal distribution, plus OTA update channels for staging and production. Acceptance: a build reaches TestFlight and installs on a physical iPhone; an OTA JS-only update reaches an installed build without App Store review; staging and production builds are independently addressable and clearly labeled in-app."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:database-reviewer,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-18] Build completion aggregation queries and materialized daily rollups for per-site and per-person on-time completion rates. Every aggregate must be traceable to its underlying task rows per RM-2. Acceptance: dashboard queries return in under 200ms at 150 users and 90 days of history; rollups are recomputed idempotently; no aggregate exposes any presence or duration metric, per ADR-004."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer" "[Plan: docs/ROADMAP.md#step-19] Build the manager dashboards: a live per-warehouse completion board, per-person history with evidence drill-down, and an overdue queue with a reassign action. Acceptance: the dashboard loads in under 2 seconds on a 4G connection per NFR-5; every displayed number drills through to the specific tasks behind it; the overdue queue updates without a manual refresh."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:typescript-reviewer,ecc:security-reviewer" "[Plan: docs/ROADMAP.md#step-20] Build the employee self-view rendering that employee's own metrics from the same components and queries the manager view uses, plus a Notice at Collection screen shown and acknowledged at first login in English and Spanish. Acceptance: employee and manager views of the same person are provably identical with no hidden fields; acknowledgement is recorded with a timestamp; no metric anywhere measures presence, duration or location, per ADR-004."
/ecc:orchestrate custom "ecc:tdd-guide,ecc:e2e-runner" "[Plan: docs/ROADMAP.md#step-21] Build the Playwright suite covering login, create task, assign, dashboard and deactivate user, plus the Maestro suite covering login, complete task offline and sync. Acceptance: all eight flows pass against a seeded staging environment; the offline Maestro flow genuinely disables the network rather than mocking it; both suites run in CI and quarantine flaky tests rather than failing the build."
/ecc:orchestrate custom "ecc:security-reviewer,ecc:typescript-reviewer,ecc:code-reviewer" "[Plan: docs/ROADMAP.md#step-22] Run the pre-pilot security pass: re-audit every RLS policy against the step 4 suite, verify signed URL expiry on storage, scan dependencies for known vulnerabilities, and confirm no secret or token is logged. Acceptance: zero high or critical dependency findings; every table's RLS policy is confirmed present and correct; no PII appears in Sentry payloads or application logs."
/ecc:orchestrate custom "ecc:database-reviewer,ecc:typescript-reviewer,ecc:code-reviewer" "[Plan: docs/ROADMAP.md#step-23] Profile and optimize dashboard aggregate queries and mobile cold start against a seeded dataset of 150 users and 90 days of history. Acceptance: the manager dashboard loads in under 2 seconds on simulated 4G per NFR-5; mobile cold start to a rendered My Day list is under 2 seconds; no N+1 query pattern remains in any dashboard endpoint."
/ecc:orchestrate custom "ecc:doc-updater" "[Plan: docs/ROADMAP.md#step-24] Generate codemaps under docs/CODEMAPS covering apps/web, apps/mobile and packages, write the README with local setup and environment variables, and document the runbook for migrations, backup restore and OTA release. Acceptance: a new engineer reaches a running local environment from the README alone; every ADR in ENGINEERING-DESIGN.md is reflected in the codemap structure; the backup restore runbook has been executed at least once."
```

---

## Related documents

- **Product Requirements** — [`docs/PRD.md`](PRD.md)
- **Engineering Design** — [`docs/ENGINEERING-DESIGN.md`](ENGINEERING-DESIGN.md)
- **Roadmap & Delivery Plan** — [`docs/ROADMAP.md`](ROADMAP.md)
