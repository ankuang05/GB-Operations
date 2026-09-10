# Engineering Design Document

**Product:** OpsLink — Workforce & Operations Platform
**Version:** 1.0 — 2026-09-09
**Author:** Andy Kuang
**Status:** Draft for stakeholder review
**Companion to:** [`PRD.md`](PRD.md)

---

## 1. Design goals and constraints

Every decision here is shaped by one dominant constraint: **two part-time engineers, ~25 hours per week combined.** That is roughly one-fifth of a conventional two-person team. The architecture is optimized for *hours saved*, not for elegance or scale we do not have.

| Goal | Why | Consequence |
|---|---|---|
| **Minimize surfaces we operate** | Nobody is on call | Managed services over self-hosted; one deployment target |
| **Type safety end to end** | Two people cannot manually catch integration bugs | Single TypeScript monorepo, types generated from the schema |
| **Web and iOS from shared logic** | Both surfaces are required (§2) | Shared packages for types, API client, validation, business rules |
| **Security enforced in the database** | App-layer bugs are inevitable at this staffing level | Postgres Row-Level Security as the backstop, not the only layer |
| **No dependency on GB's ERP for R1/R2** | The ERP is an unknown of unknown difficulty | Dynamics integration is isolated, late, and swappable |
| **Buy identity, storage, and realtime** | Auth alone is 4+ weeks to build correctly | Supabase |

**Non-goals for R1:** horizontal scale, microservices, multi-region, self-hosting, 99.99% uptime. GB has ~150 users in four buildings.

---

## 2. System architecture

### 2.1 The two-surface model

The CEO asked whether we can do both an iOS app and a website, where "the iOS app is the mobile version of the website, and harder tasks can be performed on the website." That is the right instinct, and it defines the architecture:

```
                    ┌───────────────────────────┐
                    │   Shared Backend (API)    │
                    │   Next.js Route Handlers  │
                    └────────────┬──────────────┘
                                 │
              ┌──────────────────┴──────────────────┐
              │                                     │
    ┌─────────▼──────────┐              ┌───────────▼──────────┐
    │  apps/web          │              │  apps/mobile         │
    │  Next.js 15        │              │  Expo / React Native │
    │                    │              │                      │
    │  FULL capability:  │              │  FLOOR capability:   │
    │  · task authoring  │              │  · My Day task list  │
    │  · dashboards      │              │  · complete + photo  │
    │  · reporting       │              │  · offline-first     │
    │  · admin           │              │  · clock in/out      │
    │  · bulk operations │              │  · pre-trip checks   │
    └─────────┬──────────┘              └───────────┬──────────┘
              │                                     │
              └──────────────┬──────────────────────┘
                             │
              ┌──────────────▼───────────────┐
              │  packages/  (shared TS)      │
              │  · types      · api-client   │
              │  · schema     · validation   │
              │  · core (business rules)     │
              └──────────────────────────────┘
```

**Critical clarification:** the iOS app is *not* a mirror of the website. It is a **deliberate subset**. The web is a superset handling everything hard — authoring, tables, bulk edits, analytics. The phone handles what is genuinely mobile: standing in a warehouse aisle, gloved, checking off work.

This is why we do **not** use a single cross-rendering codebase (`react-native-web`). Rendering a dense manager dashboard through React Native primitives is slow to build and worse to use. Two tailored frontends over one shared core is *less* total work than one codebase fighting two very different jobs.

### 2.2 Technology stack

| Layer | Choice | Why this and not the alternative |
|---|---|---|
| **Monorepo** | pnpm workspaces + Turborepo | Shared types across web/mobile/API without publishing packages |
| **Language** | TypeScript (strict) | One language across all surfaces; both engineers productive everywhere |
| **Web** | Next.js 15, App Router, React Server Components | Serves UI *and* API from one deployable; RSC removes most client state work |
| **Mobile** | Expo (React Native), EAS Build | No local Xcode toolchain to maintain; OTA updates skip App Store review for JS fixes — decisive at our staffing |
| **API** | Next.js Route Handlers + tRPC | End-to-end types with no codegen step; mobile calls the same procedures the web does |
| **Database** | Postgres (Supabase) | Relational data with hard tenant isolation via RLS |
| **ORM** | Drizzle | SQL-first, so RLS policies and queries stay coherent; no migration engine fighting Supabase |
| **Auth** | Supabase Auth | ~4 weeks of work we do not do; JWT claims carry role and tenant |
| **File storage** | Supabase Storage | Task-completion photos, pre-trip check evidence |
| **Styling** | Tailwind (web) + NativeWind (mobile) | One mental model for both surfaces |
| **Web hosting** | Vercel | Zero-config for Next.js; preview deploy per PR |
| **Testing** | Vitest (unit), Playwright (web E2E), Maestro (iOS E2E) | |
| **Errors / analytics** | Sentry + PostHog | The adoption metrics in PRD §9 come free |
| **CI** | GitHub Actions | Typecheck, lint, test, migration check on every PR |

**Estimated run cost at pilot scale:** ~$45–95/month (Supabase Pro $25, Vercel Pro $20, Sentry/PostHog free tiers) plus Apple Developer $99/yr.

---

## 3. Data model

### 3.1 Core entities (R1)

```sql
organizations       -- tenant root. GB is row #1.
  id, name, slug, timezone, settings_json, created_at

sites               -- GB's four warehouses
  id, organization_id, name, address, geofence_center, geofence_radius_m

profiles            -- extends Supabase auth.users
  id (= auth.users.id), organization_id, site_id, role, full_name,
  phone, preferred_locale, is_active, created_at
  -- role ∈ ('ceo','manager','sales','warehouse','driver','customer')

task_templates      -- a reusable daily checklist
  id, organization_id, site_id, name, description,
  recurrence_rule,                  -- RFC 5545 RRULE
  default_assignee_role, requires_photo, created_by, is_active

tasks               -- one concrete assigned unit of work
  id, organization_id, site_id, template_id (nullable),
  title, description, priority, due_at,
  assigned_to_profile_id (nullable),  -- null = open to the site group
  assigned_to_role (nullable),
  status,                             -- pending | in_progress | completed | skipped | expired
  created_by, created_at

task_completions    -- append-only evidence, never updated
  id, task_id, organization_id, completed_by_profile_id,
  completed_at, note, photo_path,
  client_completed_at,                -- when the phone captured it (offline)
  sync_received_at, device_id

audit_log           -- NFR-7, append-only
  id, organization_id, actor_profile_id, action, entity_type,
  entity_id, before_json, after_json, occurred_at
```

### 3.2 Three decisions worth calling out

**Every table carries `organization_id`.** Not derived through joins — physically present on every row. This makes the RLS policy in §4.2 a single uniform auditable rule instead of a per-table puzzle.

**`task_completions` is append-only and separate from `tasks`.** A completion is *evidence*, and evidence that can be edited is not evidence. When this system is eventually cited in an employment dispute, the completion record must be immutable. `tasks.status` is a derived convenience; `task_completions` is the truth.

**Completions record both `client_completed_at` and `sync_received_at`.** An employee working in a Wi-Fi dead zone completes a task at 08:14 and syncs at 11:02. Storing only server time would show them completing three hours of work in one minute and make an honest employee look like they were gaming the system. Both timestamps are shown in the UI.

### 3.3 Later entities (named, not designed)

R3 adds `products`, `inventory_snapshots`, `customers` — populated read-only by the Dynamics ingest (§5), stamped with `source_system` and `synced_at`. R4 adds `pick_tickets`, `pick_lines`, `pick_scans`. R5 adds `orders`, `order_lines`. They are named here only so R1's schema does not paint them into a corner; they are not specified until their release.

---

## 4. Security design

### 4.1 Defense in depth

Three layers, because two part-time engineers will eventually write an application-layer bug:

1. **Route middleware** — is there a valid session at all?
2. **tRPC procedure guards** — does this role have permission for this operation?
3. **Postgres Row-Level Security** — *even if layers 1 and 2 are wrong*, can this JWT touch this row?

Layer 3 is the one that matters. It is enforced by the database, cannot be bypassed by an application bug, and is why NFR-4 says "row level, not only in application code."

### 4.2 The tenant isolation policy

Supabase Auth issues a JWT; we attach `organization_id` and `role` as custom claims at sign-in. Every table then carries the same shape of policy:

```sql
alter table tasks enable row level security;

create policy tenant_isolation on tasks
  using (organization_id = (auth.jwt() ->> 'organization_id')::uuid);
```

Role rules layer on top. Warehouse employees, for example, read only tasks assigned to them or to their site group:

```sql
create policy warehouse_reads_own on tasks for select
  using (
    organization_id = (auth.jwt() ->> 'organization_id')::uuid
    and (
      (auth.jwt() ->> 'role') in ('ceo','manager')
      or assigned_to_profile_id = auth.uid()
      or (assigned_to_profile_id is null and site_id = current_user_site())
    )
  );
```

**Test obligation:** PRD acceptance criterion 3 is verified by an automated cross-tenant suite running in CI on every PR — it creates two organizations and asserts every table, under every role, returns zero rows across the boundary. Written in Sprint 1, before any feature depends on it.

### 4.3 Sensitive data handling

- **Passwords** never touch our code; Supabase Auth owns hashing and reset flows.
- **Photos** live in a private bucket, served via short-lived signed URLs, never public paths.
- **PII** in R1 is limited to `full_name` and `phone`. No SSN, no employee home address, no payment data anywhere until R5 — and card data then goes to a PCI-compliant processor, never our database.
- **Audit log** is append-only; the application role holds no `UPDATE` or `DELETE` grant on it.

---

## 5. Microsoft Dynamics integration strategy

### 5.1 What we know and don't know

GB's data lives in a Microsoft Dynamics installation that, per the CEO, is "really really old based on the interface." That most likely means **Dynamics GP, Dynamics NAV (pre-2013), or Dynamics SL** — all on-premise, all SQL Server backed, none with a modern REST API. It is emphatically *not* Dynamics 365 Business Central, which would have been the easy case.

**We are designing under uncertainty, so the design makes the uncertainty cheap.**

### 5.2 Three principles

**Principle 1 — R1 and R2 require zero Dynamics data.** This is the single most important scheduling decision in the project. The integration is the highest-variance work in the plan — it could take two weeks or four months and we cannot yet tell which — so nothing GB needs first is allowed to depend on it. If Dynamics access never materializes, R1 and R2 still ship and still deliver the CEO's stated priority.

**Principle 2 — Read-only, one-way.** We never write to Dynamics. It remains the system of record for inventory and financials; we are a read replica with an operations layer on top. Writing to a 20-year-old ERP that runs a working business is how you take a working business offline.

**Principle 3 — The adapter interface is the contract, and the CSV fallback is a first-class implementation.**

```ts
interface ERPAdapter {
  fetchProducts(since?: Date): Promise<ProductRecord[]>;
  fetchInventory(siteId: string): Promise<InventoryRecord[]>;
  fetchCustomers(since?: Date): Promise<CustomerRecord[]>;
  fetchPickTickets(since: Date): Promise<PickTicketRecord[]>;
}
```

Three implementations, in ascending order of how much GB cooperation they need:

| Implementation | Mechanism | Requires from GB |
|---|---|---|
| **`CsvDropAdapter`** | GB drops nightly exports to SFTP; we watch and ingest | Someone clicking Export nightly, or a scheduled job in Dynamics |
| **`SqlServerReadAdapter`** | Direct read-only SQL against a replica | A read-only DB user and network access |
| **`BusinessCentralAdapter`** | OData v4 / REST | Only if GB ever migrates to modern Dynamics |

**We build `CsvDropAdapter` first, unconditionally.** It works regardless of which version GB runs, needs no database credentials, no VPN, no consultant, and no security review of an inbound connection to their ERP. It converts R3 from "unknown, possibly impossible" into "two weeks." If direct SQL access later appears, we swap the implementation behind the same interface with no changes above it.

### 5.3 Ingestion pipeline

```
GB Dynamics ──nightly export──▶ SFTP drop ──▶ Ingest job ──▶ staging tables
                                                                  │
                                                    validate · diff · reconcile
                                                                  │
                                                                  ▼
                                                       OpsLink Postgres
                                                  (source_system='dynamics',
                                                        synced_at=…)
```

Rules: ingest is **idempotent** — re-running a file changes nothing. Rows upsert on the ERP's natural key. A validation failure quarantines that row and alerts, rather than aborting the batch. Every synced row is stamped with `source_system` and `synced_at`, so nobody ever has to guess whether a number is ours or theirs.

### 5.4 Long-term: the strangler-fig path

The CEO wants to eventually move off Dynamics. This architecture sets up the path without committing to it:

1. **R3 — Shadow.** We read from Dynamics; Dynamics stays authoritative for everything.
2. **R4–R5 — Own operations.** Tasks, picks, and orders originate in OpsLink. Dynamics still owns inventory and financials.
3. **Future — Selective cutover.** Domain by domain, OpsLink becomes authoritative, with Dynamics receiving a reconciliation feed until it can be retired.

We are not scheduling step 3. We are making sure we have not made it impossible.

---

## 6. Offline-first design (NFR-1)

The hardest technical requirement in R1, and the one that most determines whether the crew adopts the app.

**Model:** the mobile client holds a local SQLite mirror of *today's* tasks — never full history, bounded by design. Reads are always local, so the list opens instantly with no spinner. Writes append to a durable outbox and replay on reconnect.

```
User taps "Complete"
  → write to local SQLite            (instant, optimistic UI)
  → append to outbox                 (client_completed_at + device_id)
  → background sync drains the outbox when connectivity returns
  → server assigns sync_received_at, returns the canonical row
  → local mirror reconciled
```

**Conflict policy — deliberately simple.** Completions are append-only and effectively conflict-free: if two people complete the same group task, both are recorded and the manager sees both. We do **not** build CRDTs or three-way merge. The only real conflict is a manager deleting a task someone completed offline; that resolves as *completion wins* — the task is restored as completed and the manager is notified. Five lines of rule, not a subsystem.

**Explicit non-goal:** offline task *authoring*. Managers create tasks on the web, where they have connectivity. Offline writes are limited to completion, which is the only thing that actually happens in a dead zone.

---

## 7. Architecture Decision Records

### ADR-001 — Two tailored frontends over one cross-rendered codebase
**Status:** Accepted
**Context:** Both a website and an iOS app are required, and the CEO wants the website to handle "harder tasks."
**Decision:** Separate `apps/web` (Next.js) and `apps/mobile` (Expo), sharing `packages/*` for types, API client, validation, and business rules. Not `react-native-web`.
**Rationale:** The two surfaces have genuinely different jobs — dense data authoring versus gloved one-handed checkoff. A single cross-rendered codebase optimizes for neither and costs more hours fighting layout primitives than the second frontend costs to write. Sharing the *logic* captures nearly all the duplication savings; sharing the *rendering* captures little and costs much.
**Consequences:** Two UI codebases. Mitigated by keeping mobile a deliberate subset (§2.1) and all business rules in `packages/core`.

### ADR-002 — Multi-tenant from day one
**Status:** Accepted
**Context:** The CEO explicitly said "keep this app generalized and not specifically dedicated to one company," but GB is the only customer for the foreseeable future.
**Decision:** Every table carries `organization_id` from the first migration; RLS enforces isolation from Sprint 1.
**Rationale:** Retrofitting multi-tenancy is one of the most expensive changes in software — it touches every query, every policy, and every existing row. Adding a column and a uniform policy at the start costs about a sprint-day. Doing it later costs months. That asymmetry justifies it even at 10% odds of a second customer.
**Consequences:** Slightly more ceremony in every query. Accepted.

### ADR-003 — CSV drop as the primary ERP integration, not the fallback
**Status:** Accepted
**Context:** GB runs an unidentified, very old on-premise Dynamics installation with no known API and no confirmed IT owner.
**Decision:** Build `CsvDropAdapter` first; treat direct SQL access as an optimization behind the same `ERPAdapter` interface.
**Rationale:** Variance on direct integration is enormous — version, network access, credentials, and a consultant who may not exist. A nightly export file works against every version of Dynamics ever shipped, needs no credentials into GB's ERP, and requires no security review of an inbound connection to their production system. It converts the riskiest item in the plan into a bounded one.
**Consequences:** Data is up to 24 hours stale in R3. Acceptable — inventory lookup and catalog do not need real-time accuracy for GB's use case.

### ADR-004 — Measure work output, never bodily presence
**Status:** Accepted · **Stakeholder decision required**
**Context:** The CEO asked for features addressing 30+ minute bathroom breaks, on-the-clock chatting, and "AI to keep employees on track."
**Decision:** The platform measures task completion, on-time rates, pre-trip compliance, pick accuracy, and cycle time. It does **not** measure break duration, idle time, indoor location, or message content. AI surfaces operational anomalies and drafts task lists; it never scores or ranks individuals. Full policy: PRD §7.
**Rationale:** Two reasons, either sufficient alone. **Legal:** California's duty-free break requirements (Labor Code §226.7, IWC Wage Orders), FEHA/ADA protection of restroom frequency as a medical characteristic, two-party consent for communications (Penal Code §632), and CPRA ADMT obligations landing inside our R6 window together make presence-monitoring a liability GB does not currently have. **Product:** output metrics answer the CEO's actual question better — "14 of 16 tasks completed, 2 overdue" is more useful to a supervisor than "44 minutes in the bathroom," and it is a conversation they can have without counsel present.
**Consequences:** We are declining to build two features as literally described. GB should have employment counsel review PRD §7 before R2 ships clock-in. **This ADR should be explicitly accepted or rejected by the CEO before Sprint 4.**

### ADR-005 — Buy identity rather than build it
**Status:** Accepted
**Context:** Six roles, multi-tenant, password reset, session management, eventual SSO.
**Decision:** Supabase Auth for credentials and sessions; roles and tenancy as JWT claims; RLS for enforcement.
**Rationale:** Correct auth is ~4 weeks of our ~25 hr/week capacity — over 15% of the entire R1 budget — spent on a solved, commoditized, high-blast-radius problem. Buying it converts a month of work and a permanent security liability into a day of configuration.
**Consequences:** Supabase becomes a hard dependency. Mitigated because Supabase is standard Postgres — the data stays portable even if the auth layer someday needs replacing.

### ADR-006 — Ship a two-tab app, not a five-tab shell
**Status:** Accepted
**Context:** The CEO specified a five-page structure modeled on OMELINK. R1 fills only part of it.
**Decision:** R1 ships My Day and Profile. Other tabs appear as their features become real.
**Rationale:** Adoption is the top risk in the PRD register. Three dead tabs on first launch teach the crew the app is unfinished and not worth opening, and first impressions with a warehouse workforce are hard to reverse. The five-tab target is unchanged; only its reveal is sequenced.
**Consequences:** The app looks less ambitious in early demos. Worth it.

---

## 8. Testing and quality strategy

At 25 hours/week we cannot afford broad manual QA, so testing concentrates where failure is expensive rather than spreading evenly:

| Layer | Tool | Target | What it protects |
|---|---|---|---|
| Cross-tenant isolation | Vitest + Supabase test harness | **100% of tables** | PRD acceptance criterion 3 — the non-negotiable one |
| Business rules (`packages/core`) | Vitest | ≥80% | Recurrence, due-time, permission resolution |
| API procedures | Vitest integration | ≥70% | The contract between web and mobile |
| Web critical paths | Playwright | 5 flows | Login, create task, assign, dashboard, deactivate user |
| iOS critical paths | Maestro | 3 flows | Login, complete task offline, sync |
| Everything else | Manual, pre-release | — | |

**CI gate on every PR:** typecheck, lint, unit tests, cross-tenant suite, migration dry-run. A red cross-tenant suite blocks merge unconditionally.

---

## 9. Deployment and environments

| Environment | Purpose | Database | Mobile |
|---|---|---|---|
| **Local** | Development | Supabase local (Docker) | Expo Go |
| **Preview** | Per-PR review | Shared preview project, seeded | — |
| **Staging** | Pilot rehearsal, migration testing | Own Supabase project | TestFlight internal |
| **Production** | GB live | Own Supabase project, PITR enabled | App Store |

**Release process:** trunk-based development on `main` with short-lived branches. **Both engineers review every PR** — that is the bus-factor mitigation, not a formality. Web deploys continuously to staging on merge; production is a manual promote. Mobile ships through EAS to TestFlight weekly from Sprint 3 onward, so App Store review is never a surprise at the end. JS-only fixes go out as OTA updates without review.

**Backups:** Supabase point-in-time recovery on production, **with one restore rehearsed during Sprint 5**. A backup nobody has restored is not a backup.

---

## 10. Appendix A — Dynamics discovery checklist

*Hand this to whoever administers GB's Dynamics system. Answers unblock D1 and D2 and determine whether R3 is two weeks or three months. Needed by Sprint 2.*

**Identify the system**
1. Exact product and version — from Help → About in the Dynamics client ("Dynamics GP 2013", "Dynamics NAV 2009 R2", "Dynamics SL 2011", …).
2. Where does it run — a server in a GB office, a hosting provider, or a consultant's data center?
3. Which SQL Server version backs it?
4. Roughly how many products, customers, and orders per month does it hold?

**Access**
5. Who administers it — internal staff, an outside consultant, or nobody currently?
6. Is there an existing reporting or read-only database user, or a reporting replica?
7. Can a scheduled export job be added, and can it write to a folder or SFTP location?
8. Is any integration or export already running (accounting, EDI, a website)?

**Data shape** — for each of Products, Inventory, Customers, Pick Tickets:
9. Which screen or report displays it today?
10. Can someone export one sample as CSV or Excel? *(Redact pricing and customer names if needed — we need column names and formats, not real values.)*
11. What is the natural key — item number, customer ID, ticket number?
12. What date format does the export use?

**Constraints**
13. Any maintenance window when the database must not be queried?
14. Any contractual or vendor restriction on third-party access?
15. Is there a support contract with the Dynamics vendor, and is it current?

**The minimum viable answer:** if the only question GB can answer is #7 — *yes, we can produce a nightly CSV export* — then R3 proceeds on schedule via `CsvDropAdapter`. Everything else is optimization.

---

## 11. Related documents

- **Product Requirements** — [`docs/PRD.md`](PRD.md)
- **Roadmap & Delivery Plan** — [`docs/ROADMAP.md`](ROADMAP.md)
- **Build Orchestration** — [`docs/ORCHESTRATION.md`](ORCHESTRATION.md)
