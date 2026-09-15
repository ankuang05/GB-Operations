# How It's Built

**Product:** OpsLink — iOS operations app
**Version:** 3.0 — 2026-09-14
**Author:** Andy Kuang
**Companion to:** [PRODUCT.md](PRODUCT.md)

> This is the technical document, but it's written to be readable. If you're not an engineer, you can skip to [§7 The decisions we made](#7-the-decisions-we-made) — that section explains every significant choice in plain language, including what each one costs us.

---

## 1. The one constraint that shapes everything

**Two part-time engineers, about 25 hours a week between them.** That's roughly a quarter of a normal two-person team.

Every choice below optimizes for *hours saved*, not for elegance, and not for scale we don't have.

| Goal | Why it matters here | What it forces |
|---|---|---|
| **Run as little infrastructure as possible** | Nobody is on call at 2am | Managed services, not servers we maintain |
| **Let the database enforce security** | Two part-time people *will* write an app bug eventually | Postgres Row-Level Security as the backstop |
| **One app, not two** | Scope was narrowed to iOS | A single SwiftUI codebase that adapts to iPhone and iPad |
| **Keep the rules out of the app** | A website might come later | Business logic lives in Postgres, not in Swift |
| **Don't depend on GB's ERP early** | It's an unknown of unknown difficulty | Dynamics work is isolated, late, and swappable |
| **Buy login, don't build it** | Doing auth correctly is 4+ weeks | Supabase Auth |

**Explicitly not goals for the MVP (Phases 1–2):** horizontal scale, microservices, multi-region, self-hosting, 99.99% uptime. GB has ~150 people in four buildings.

---

## 2. The shape of the system

### 2.1 The whole picture

```
                    ┌─────────────────────────────────┐
                    │       OpsLink iOS App           │
                    │       Swift 6 + SwiftUI         │
                    │                                 │
                    │   iPhone layout    iPad layout  │
                    │   ───────────      ───────────  │
                    │   My Day           Task builder │
                    │   Complete + photo Dashboards   │
                    │   Offline-first    Admin        │
                    │   Clock in/out     Reporting    │
                    └───────────────┬─────────────────┘
                                    │
                         talks directly to
                                    │
                    ┌───────────────▼─────────────────┐
                    │          Supabase               │
                    │                                 │
                    │  Postgres  ← the rules live here│
                    │    · Row-Level Security         │
                    │    · constraints + triggers     │
                    │  Auth      ← logins, sessions   │
                    │  Storage   ← completion photos  │
                    │  Edge Fns  ← nightly jobs       │
                    └─────────────────────────────────┘
                                    ▲
                                    │ (later, if we want it)
                    ┌───────────────┴─────────────────┐
                    │   A website would plug in here  │
                    │   and inherit every rule free   │
                    └─────────────────────────────────┘
```

**The important thing about that diagram** is the bottom box. There is no custom API server in the middle, and that's deliberate. All the rules live in Postgres. The iOS app is one client. A website would be a second client of the same rules, not a reimplementation of them.

### 2.2 One app, two layouts

There is a single app in the App Store. When you log in, it reads your **role** (what you're allowed to touch) and your **job tags** (what's useful to you) and builds itself accordingly.

| Role + tag | iPhone | iPad |
|---|---|---|
| **Employee** | My Day, Profile | Same, wider |
| **Employee** · `driver` | Pre-trip check, deliveries, Profile | Same, wider |
| **Employee** · `receiver` | Receiving checks *(Phase 3+, pending the process walkthrough)* | Same, wider |
| **Management** · site scope | Today's board, quick reassign | **Full task builder, dashboards, admin** |
| **Management** · org scope *(the CEO)* | Summary numbers | **All warehouses, trends, reporting** |
| **Customer** | Catalog, cart, order history *(Phase 5)* | Same, wider |

SwiftUI handles this natively — the same views adapt through size classes and `NavigationSplitView`, which gives a sidebar-plus-detail layout on iPad and a stack of screens on iPhone. We write the screen once.

**What we are *not* doing:** shipping two apps, or building an iPad-only manager app. One binary, one review process, one thing to install.

### 2.3 The stack

| Layer | What we use | Why this and not the alternative |
|---|---|---|
| **Language** | Swift 6, strict concurrency | Native. The whole product is one platform now |
| **UI** | SwiftUI, iOS 17+ | Adapts iPhone↔iPad for free; far less layout code than UIKit |
| **Local storage** | GRDB (SQLite) | The offline mirror and outbox need predictable, inspectable SQL. SwiftData is newer and less proven for a sync queue |
| **Backend** | Supabase — Postgres, Auth, Storage, Edge Functions | One managed service covers the database, logins, file storage, and scheduled jobs |
| **API** | **None.** `supabase-swift` talks to Postgres directly | Row-Level Security does the enforcing. A middle tier would be code we write, test, and host for no added safety |
| **Server-side jobs** | Supabase Edge Functions (TypeScript) on a schedule | Only two jobs exist in the MVP: materialize recurring tasks nightly, expire overdue ones |
| **Migrations** | Plain `.sql` files, Supabase CLI | Version-controlled, reviewable, no ORM fighting the RLS policies |
| **Testing (app)** | Swift Testing + XCUITest | Apple's own tooling, runs in Xcode and CI |
| **Testing (database)** | Vitest against a local Supabase | Runs on a cheap Linux CI runner instead of a slow macOS one — matters when it runs on every single change |
| **CI** | GitHub Actions | Linux runner for database and migration checks, macOS runner for the app build |
| **Distribution** | TestFlight → App Store Connect | Standard Apple path |
| **Crashes / usage** | Sentry + PostHog | The adoption numbers in [PRODUCT.md §10](PRODUCT.md#10-how-well-know-it-worked) come free |

**Running cost at trial size:** about **$25–50/month** (Supabase Pro $25; Sentry and PostHog free tiers) plus Apple Developer $99/year. Dropping the website removed the web hosting bill entirely.

---

## 3. The data

### 3.1 What we store in the MVP (Phases 1–2)

```sql
organizations       -- one row per company. GB is row #1.
  id, name, slug, timezone, settings_json, created_at

sites               -- GB's four warehouses
  id, organization_id, name, address,
  geofence_center, geofence_radius_m

profiles            -- a person, attached to a Supabase login
  id (= auth.users.id), organization_id, site_id, role, scope, job_tags,
  full_name, phone, preferred_locale, is_active, created_at
  -- role     is one of: management | employee | customer
  -- scope    is one of: site | organization      (the CEO is organization-scoped)
  -- job_tags is a set:  picker | stager | qc | driver | receiver | buyer | sales | office
  -- site_id is the home warehouse; ignored when scope = organization

task_templates      -- a reusable checklist, e.g. "morning cold-storage round"
  id, organization_id, site_id, name, description,
  recurrence_rule,                 -- standard calendar repeat format (RFC 5545)
  default_assignee_role, requires_photo, created_by, is_active

tasks               -- one concrete piece of work, assigned
  id, organization_id, site_id, template_id (may be empty),
  title, description, priority, due_at,
  assigned_to_profile_id (may be empty),   -- empty = open to the whole site
  assigned_to_role (may be empty),
  status,   -- pending | in_progress | completed | skipped | expired
  created_by, created_at

task_completions    -- proof that it happened. Never edited, never deleted.
  id, task_id, organization_id, completed_by_profile_id,
  completed_at, note, photo_path,
  client_completed_at,   -- when the phone recorded it (possibly offline)
  sync_received_at,      -- when the server heard about it
  device_id

audit_log           -- who changed what, when. Never edited, never deleted.
  id, organization_id, actor_profile_id, action, entity_type,
  entity_id, before_json, after_json, occurred_at
```

### 3.2 Four choices worth explaining

**Every table carries `organization_id` directly.** Not looked up through a chain of joins — physically on every row. This makes the security rule in [§4](#4-security) one simple, uniform, auditable line instead of a different puzzle per table.

**Completions are separate from tasks, and can never be changed.** A completion is *evidence*. Evidence you can edit isn't evidence. When this system eventually gets quoted in an employment dispute — and it will — the record needs to be immutable. `tasks.status` is a convenience field; `task_completions` is the truth.

**Permission is a role; job is a tag.** `role` decides what the database will let you touch — three values, small enough to test exhaustively. `job_tags` decides which screens are worth showing you. A driver and a picker need identical permissions and different screens, so they differ by tag, not by role. Keeping these separate is what stops the permission matrix growing a new row every time GB names a new job. A tag gets promoted to a real role only when a phase needs it to carry different permissions — most likely `driver`, at the pre-trip checklist.

**The CEO is not a fourth role.** He is `management` with `scope = organization`, which is why he sees four warehouses where a manager sees one. A regional manager covering two sites is the same mechanism, and costs nothing to add.

**We record two timestamps on every completion.** An employee in a Wi-Fi dead zone finishes a task at 08:14 and their phone syncs at 11:02. If we stored only the server's time, it would look like they did three hours of work in one minute, and an honest employee would look like they were gaming the system. We store both, and we **show both** in the app.

### 3.3 What comes later

Phase 3 adds `products`, `inventory_snapshots`, `customers` — filled read-only from Dynamics, each row stamped with where it came from and when. Phase 5 adds `orders`, `order_lines`. Pick tickets (`pick_tickets`, `pick_lines`, `pick_scans`) arrive only if digital pick dispatch is scheduled — it is currently out of scope.

They're named here only so the MVP's design doesn't paint them into a corner. They aren't specified until their phase.

---

## 4. Security

### 4.1 Two layers, and the second one is the real one

1. **The app checks permissions** — does this role get to see this screen?
2. **Postgres Row-Level Security checks again** — *even if layer 1 is wrong,* can this specific login touch this specific row?

Layer 2 is the one that matters. It's enforced by the database, it cannot be talked around by a bug in Swift, and it's why [PRODUCT.md N4](PRODUCT.md#6-requirements-that-arent-features) says "in the database, not just in the app."

Because there's no API server in between, this isn't a nice-to-have — it *is* the security model. That's a feature, not a compromise: there's exactly one place where access is decided, so there's exactly one place to audit.

### 4.2 How company isolation works

Supabase issues a signed token when you log in. We attach your `organization_id` and `role` to it. Every table then gets the same shape of rule:

```sql
alter table tasks enable row level security;

create policy tenant_isolation on tasks
  using (organization_id = (auth.jwt() ->> 'organization_id')::uuid);
```

Role rules stack on top. An employee, for instance, sees only tasks assigned to them or open to their site, while management sees the whole site (or the whole company, if organization-scoped):

```sql
create policy tasks_readable on tasks for select
  using (
    organization_id = (auth.jwt() ->> 'organization_id')::uuid
    and (
      -- management: organization-scoped sees every site, site-scoped sees only its own
      (
        (auth.jwt() ->> 'role') = 'management'
        and (
          (auth.jwt() ->> 'scope') = 'organization'
          or site_id = current_user_site()
        )
      )
      -- employees: their own work, or work left open to their site
      or assigned_to_profile_id = auth.uid()
      or (assigned_to_profile_id is null and site_id = current_user_site())
    )
  );
```

**Note the `scope` check is load-bearing.** Without it, `role = 'management'` alone would let a single-warehouse manager read all four warehouses' tasks — the exact failure the scope field exists to prevent. Site scope is a boundary, not a display preference, so it belongs in the policy rather than in the app.

**The test we owe on this.** Success criterion 3 in [PRODUCT.md §4](PRODUCT.md#4-what-ships-when) is verified by an automated suite that runs on every single code change: it creates two fake companies **each with two sites**, then asserts that for every table, under every role and scope, looking across either boundary returns **zero rows**. It's written in Phase 2, *before* any feature that depends on it. A failure blocks the change from merging, no exceptions.

### 4.3 Sensitive data

- **Passwords** never touch our code. Supabase Auth owns hashing and reset flows.
- **Photos** live in a private bucket, handed out through short-lived signed links. Never a public URL.
- **Personal data in the MVP** is limited to a name and phone number. No SSN, no home address, no payment data anywhere until Phase 5 — and card data then goes to a payment processor, never into our database.
- **The audit log** cannot be updated or deleted, because the app's database role isn't granted permission to do either.

---

## 5. Working offline

This is the hardest technical requirement in the MVP, and the one that most decides whether the crew keeps opening the app.

**The model.** The phone keeps a local SQLite copy of **today's** tasks — never the full history, so it stays small by design. Reads always come from local storage, which means the list opens instantly with no spinner. Writes go into a durable queue and replay when the connection comes back.

```
Employee taps "Complete"
  → save to local SQLite                (instant — the UI updates immediately)
  → add to the outbox queue             (with the phone's timestamp + device id)
  → background sync drains the queue when signal returns
  → server stamps its own received-time and returns the official row
  → local copy reconciled
```

**What happens when two people do the same thing.** Completions are append-only, so they barely conflict: if two people complete the same group task, both are recorded and the manager sees both. We are **not** building conflict-resolution machinery.

The one real conflict is a manager deleting a task that someone already completed offline. That resolves as **completion wins** — the task comes back marked complete, and the manager gets told. Five lines of rule, not a subsystem.

**Deliberately not supported:** creating tasks offline. Managers build tasks on an iPad, where they have Wi-Fi. Offline writing is limited to completing work, which is the only thing that actually happens in a dead zone.

---

## 6. Connecting to Dynamics (Phase 3)

### 6.1 What we know and don't know

GB's data lives in a Microsoft Dynamics installation that the CEO describes as "really really old based on the interface." That most likely means **Dynamics GP, Dynamics NAV (pre-2013), or Dynamics SL** — all installed on a server somewhere, all backed by SQL Server, none with a modern way to ask them questions over the internet. It is emphatically *not* the modern Dynamics 365, which would have been the easy case.

**We're designing under uncertainty, so the design makes the uncertainty cheap.**

### 6.2 Three rules

**Rule 1 — Phases 1 and 2 need zero Dynamics data.** This is the single most important scheduling decision in the project. Connecting to Dynamics is the highest-variance work in the whole plan; it could take two weeks or four months and we genuinely cannot tell which yet. So nothing GB needs first is allowed to depend on it. If Dynamics access never happens at all, the MVP still ships and still delivers what the CEO asked for.

**Rule 2 — We only read, for now.** Dynamics stays the official record for stock and money. We're a copy with an operations layer on top. Writing into a 20-year-old ERP that runs a working business is how you take a working business offline.

> **The CEO has asked for write-back** — orders entered in the app, inserted into Dynamics. That is the correct long-term shape, and it is the direct fix for re-keying (P4 in [PRODUCT.md §2.1](PRODUCT.md#21-things-this-app-fixes)). It is **not** committed, and it is **not** part of Phase 3.
>
> **What would have to be true first.** A spike, scheduled only after the read path has run in production for a full quarter, answering: does this Dynamics version expose a supported write path at all, or would we be writing to SQL Server tables directly? Is there a sandbox copy to test against, or only production? Can a bad write be reversed without a database restore? Who at GB signs off on an automated process touching the financial record?
>
> Until every one of those has an answer, the direction of data is one-way. The failure mode we are avoiding is not an inconvenience — it is corrupting twenty years of inventory and receivables in a system nobody currently administers.

**Rule 3 — The plain-file option is the main plan, not the backup.**

We define one small interface, and three ways to satisfy it, in order of how much cooperation each needs from GB:

| Option | How it works | What GB has to do |
|---|---|---|
| **Nightly CSV drop** | GB exports files to a folder each night; we pick them up | Someone clicks Export nightly, or schedules a job |
| **Direct read-only SQL** | We query a read-only copy of their database | Provide a read-only login and network access |
| **Modern API** | Standard REST | Only if GB ever upgrades Dynamics |

**We build the CSV option first, unconditionally.** It works against every version of Dynamics ever shipped. It needs no database credentials, no VPN, no consultant, and no security review of an inbound connection to their production system. It turns Phase 3 from "unknown, possibly impossible" into "about two weeks." If direct access appears later, we swap it in behind the same interface and nothing above it changes.

### 6.3 How the nightly import works

```
GB Dynamics ──nightly export──▶ file drop ──▶ import job ──▶ staging tables
                                                                  │
                                                     check · compare · reconcile
                                                                  │
                                                                  ▼
                                                          OpsLink Postgres
                                                    (stamped: from Dynamics, at 03:00)
```

The rules: running the same file twice changes nothing. Rows update in place based on the ERP's own ID. One bad row gets quarantined and flagged rather than killing the whole batch. Every imported row is stamped with where it came from and when, so nobody ever has to guess whether a number is ours or theirs.

### 6.4 The long game

The CEO wants to eventually move off Dynamics. This setup builds the path without committing to it:

1. **Phase 3 — Shadow.** We read from Dynamics. Dynamics is still in charge of everything.
2. **Phases 4–5 — Own the operations.** Tasks and orders start in OpsLink. Dynamics still owns stock and money. Write-back, if the spike above clears it, belongs here.
3. **Someday — Selective handover.** One area at a time, OpsLink becomes the official record, feeding Dynamics a reconciliation copy until it can be switched off.

We're not scheduling step 3. We're just making sure we haven't made it impossible.

---

## 7. The decisions we made

*Each one: what we decided, why, and what it costs. This section is the one to read if you skipped the rest.*

### D1 — One app that adapts, not separate iPhone and iPad apps
**What:** a single SwiftUI codebase that lays itself out differently depending on screen size and who's logged in.
**Why:** SwiftUI does this natively. Two apps would mean two App Store listings, two review cycles, two things to install, and twice the code — for the same features.
**Cost:** the manager screens have to work acceptably on an iPhone too, even though iPad is where they belong. Accepted.

### D2 — Native Swift instead of React Native
**What:** build in Apple's own language and framework.
**Why:** the previous plan used React Native specifically so one codebase could serve a website *and* a phone app. With the website gone, that reason is gone with it. Native gives us better camera, offline storage, and performance, and a much simpler project with no JavaScript build system in the middle.
**Cost:** **the team has to be productive in Swift.** This is the biggest open risk in the technical plan.
**Mitigation:** Phase 0 includes a real test — build a working screen against the real database in the first two weeks. If it's painful, we switch back to React Native *before* any real code exists. See [TIMELINE.md](TIMELINE.md).
**Also cost:** we lose over-the-air updates. See D10.

### D3 — No API server; the app talks to the database directly
**What:** `supabase-swift` connects straight to Postgres. There's no Node/Next.js service in between.
**Why:** a middle tier's main job is enforcing rules, and Row-Level Security already does that — inside the database, where it can't be bypassed. Writing one anyway would be a service to build, test, host, monitor and pay for, adding no safety.
**Cost:** all business logic has to be expressible in SQL — policies, constraints, triggers, and the occasional Edge Function. That's a real constraint, and it's also exactly what makes D4 work.

### D4 — Keep the rules in the database so a website stays cheap *(new in v2)*
**What:** who-can-see-what, how a repeating task becomes tomorrow's task, and what counts as complete all live in Postgres. Swift holds screens and the offline cache. Nothing else.
**Why:** the owner wants to focus on the app now but keep a website open as an option. If the rules lived in Swift, a website would mean rewriting all of them in another language and then keeping two copies in agreement forever — the classic way products develop two different sets of behavior. With the rules in the database, a website is a new face on the same system.
**Cost:** essentially nothing. It's how we'd build it anyway. It just becomes a rule we hold ourselves to and check in code review.
**Payoff:** a read-only CEO dashboard on the web would be roughly 6–8 weeks instead of a rebuild.

### D5 — Multi-company from day one
**What:** every table carries `organization_id` from the very first migration, and isolation is enforced from Phase 2, the moment real logins exist.
**Why:** the CEO explicitly said to keep this general rather than GB-specific. Retrofitting this later is one of the most expensive changes in software — it touches every query, every rule, and every existing row. Adding a column and one uniform policy at the start costs about a day.
**Cost:** slightly more ceremony in every query. Worth it even at 10% odds of a second customer.

### D6 — The CSV file drop is the main Dynamics plan, not the fallback
**What:** build the nightly-file import first; treat direct database access as an optimization to add later.
**Why:** direct integration has enormous variance — unknown version, network access, credentials, and a consultant who may not exist. A nightly export file works against every version of Dynamics ever made and requires no inbound connection to GB's production system.
**Cost:** data is up to 24 hours old in Phase 3. Fine — stock lookup and a catalog don't need to be real-time for GB's use.

### D7 — Measure work finished, never bodily presence
**What:** the app measures task completion, on-time rates, pre-trip compliance, pick accuracy, cycle time. It does **not** measure break length, idle time, indoor location, or message content. AI drafts task lists and flags operational oddities; it never scores or ranks people.
**Why:** two reasons, either one sufficient on its own. **Legal:** California's duty-free break rules, protection of restroom frequency as a medical characteristic, two-party consent for recording communications, and new automated-decision-making rules landing inside our Phase 6 window together turn presence-monitoring into a liability GB doesn't currently have. **Product:** output metrics answer the CEO's real question better — *"14 of 16 tasks done, 2 overdue"* is more useful to a supervisor than *"44 minutes in the bathroom,"* and it's a conversation they can have without a lawyer in the room.
**Also declined:** the newer brief asks for **computer vision in the warehouse to alert management when workers are slacking off**, reasoning that the surveillance objection is the company's problem rather than ours. It isn't. Under California law the party that builds a monitoring system is named alongside the party that runs it, and "they chose to switch it on" is not a defence that has ever worked. The request is on the record; the answer is no.
**Cost:** we are declining to build three features as literally described.
**Status:** ⚠️ **The CEO should accept or reject this in writing before Phase 0 ends.** Full reasoning: [WORKFORCE-POLICY.md](WORKFORCE-POLICY.md).

### D8 — Buy logins rather than build them
**What:** Supabase Auth handles credentials, sessions, and password resets. Role and company travel in the login token.
**Why:** doing authentication correctly is about 4 weeks of our 25-hour weeks — over 15% of the entire MVP budget — spent on a solved, commoditized problem where a mistake is catastrophic.
**Cost:** Supabase becomes a hard dependency. Softened by the fact that it's ordinary Postgres underneath, so the data stays portable.

### D9 — Ship a two-tab app, not a five-tab shell
**What:** Phase 1 shows warehouse employees My Day and Profile. Other tabs appear as they become real.
**Why:** adoption is the top risk in the whole project. Three dead tabs on first launch teach the crew the app is unfinished and not worth opening, and first impressions with a warehouse workforce are hard to reverse.
**Cost:** the app looks less ambitious in early demos. Worth it.

### D10 — Accept App Store review; work around it with remote settings
**What:** without React Native, every bug fix goes through Apple's review queue (typically 24–48 hours).
**Why:** it's the price of D2.
**Mitigation:** (a) the entire trial runs on **TestFlight**, where builds land in hours, not days; (b) a small settings table in Postgres lets us turn features on and off, adjust thresholds, and change text **without a new build at all**. Most "urgent fixes" during a trial are really "turn that off for now," and that becomes a database update.
**Cost:** genuine, but bounded. Revisit only if it actually bites during the trial.

### D11 — Three permission roles, with job as a tag *(new in v3)*
**What:** `role` is one of `management`, `employee`, `customer`. The specific job — picker, stager, QC, driver, receiver, buyer, sales — is a **tag** on the person. Warehouse breadth is a separate `scope` field, which is how the CEO sees four sites and a manager sees one.
**Why:** the CEO's brief groups his own people this way, and it happens to be the right technical shape too. Permissions are the part that must be exhaustively tested, and three roles is a matrix small enough to test exhaustively. Six roles — the previous plan — meant six times the policy surface to cover a difference that was mostly about *which screen to open*, not *what the database should permit*.
**Cost:** a screen sometimes has to check a tag as well as a role, which is slightly more branching in the app. Cheap, and it happens where mistakes are visible rather than silent.
**Escape hatch:** a tag becomes a real role the moment a phase needs it to carry different permissions. `driver` is the likely first, at the pre-trip checklist. Adding one is a migration and a policy, not a redesign.

### D12 — Dynamics write-back is deferred, not refused *(new in v3)*
**What:** data flows one way — out of Dynamics, into OpsLink. Inserting orders back into Dynamics is a candidate later stage, gated on a spike that runs no earlier than a full quarter after the read path is live.
**Why:** the CEO asked for write-back and he is right about the destination — re-keying is a real cost. But nobody has yet established which Dynamics version this is, whether it exposes a supported write path or only raw SQL Server tables, whether a sandbox exists, or who administers the system. Committing to write into a financial record under those conditions is not ambition, it's a coin flip with twenty years of inventory and receivables.
**Cost:** orders still get keyed in by hand until that spike clears. That is the single largest piece of manual work we are choosing not to fix yet, and it should be named as such rather than buried.
**How it unblocks:** [§6.2, Rule 2](#62-three-rules) lists the four questions whose answers decide it.

---

## 8. Testing

At 25 hours a week we can't afford broad manual QA, so testing is concentrated where failure is expensive rather than spread evenly.

| What | Tool | Target | What it protects |
|---|---|---|---|
| **Company isolation** | Vitest against local Supabase | **100% of tables** | Success criterion 3 — the non-negotiable one |
| Database rules (permissions, recurrence, constraints) | Vitest + pgTAP | 80%+ | The rules everything else depends on |
| App logic (sync, outbox, view models) | Swift Testing | 80%+ | Offline correctness |
| Critical user journeys | XCUITest | 5 flows | Log in · build a task · complete offline · sync · view dashboard |
| Everything else | Manual, before each release | — | |

**Runs on every change:** build, lint, unit tests, the isolation suite, and a migration dry-run. **A failing isolation suite blocks the merge unconditionally.**

**Why the database tests are in TypeScript and not Swift.** They run on every single change, and a Linux CI machine is far cheaper and faster than a Mac one. The app tests need a Mac; the database tests don't, so they don't use one. It also means a future website would inherit the same test suite unchanged.

---

## 9. Environments and shipping

| Environment | Purpose | Database | App |
|---|---|---|---|
| **Local** | Day-to-day development | Supabase running in Docker | Xcode simulator |
| **Staging** | Rehearsal, migration testing | Its own Supabase project | TestFlight, internal only |
| **Production** | GB live | Its own Supabase project, with point-in-time recovery | App Store |

**How we ship.** Short-lived branches merged into `main`. **Both engineers review every change** — that's the two-person-team insurance policy, not a formality. Database changes deploy to staging automatically; production is a deliberate manual step. App builds go to TestFlight **weekly from November 2026**, so App Store review is never a nasty surprise at the end.

**Backups.** Point-in-time recovery on production, **with one restore actually rehearsed during the hardening window (Jan 25 – Feb 5, 2027).** A backup nobody has ever restored is not a backup.

---

## 10. Appendix — Dynamics questionnaire

*Hand this to whoever administers GB's Dynamics system. The answers decide whether Phase 3 takes two weeks or three months. **Needed by 2026-11-06.***

**Identifying the system**
1. Exact product and version — from Help → About in the Dynamics window ("Dynamics GP 2013", "Dynamics NAV 2009 R2", "Dynamics SL 2011"…).
2. Where does it run — a server in a GB office, a hosting company, or a consultant's data center?
3. Which version of SQL Server is behind it?
4. Roughly how many products, customers, and orders per month does it hold?

**Access**
5. Who administers it — internal staff, an outside consultant, or nobody right now?
6. Is there already a reporting login or a read-only copy of the database?
7. **Can a scheduled export be added, and can it write files to a folder?** ← *the important one*
8. Is anything already exporting from it — accounting, EDI, a website?

**The data itself** — for each of Products, Stock, Customers, Pick Tickets:
9. Which screen or report shows it today?
10. Can someone export one sample as CSV or Excel? *(Feel free to black out prices and customer names — we need the column names and formats, not real values.)*
11. What's the unique ID — item number, customer ID, ticket number?
12. What date format does the export use?

**Constraints**
13. Any maintenance window when the database mustn't be touched?
14. Any contract or vendor restriction on third-party access?
15. Is there a current support contract with the Dynamics vendor?

> **The minimum useful answer:** if the only question GB can answer is **#7** — *yes, we can produce a nightly export file* — then Phase 3 goes ahead on schedule. Everything else on this list is optimization.

---

## Related documents

- **[FOR-THE-CEO.md](FOR-THE-CEO.md)** — the plain-English overview
- **[PRODUCT.md](PRODUCT.md)** — what we're building and for whom
- **[TIMELINE.md](TIMELINE.md)** — the schedule
- **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)** — what we measure and what we won't
