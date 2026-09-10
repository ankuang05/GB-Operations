# Roadmap & Delivery Plan

**Product:** OpsLink — Workforce & Operations Platform
**Version:** 1.0 — 2026-09-09
**Companion to:** [`PRD.md`](PRD.md) · [`ENGINEERING-DESIGN.md`](ENGINEERING-DESIGN.md)

---

## 1. The capacity math, stated openly

This plan rests on one number, and it should be visible to every stakeholder who reads it:

| | |
|---|---|
| Team | 2 engineers, part-time (students / side project) |
| Nominal capacity | ~10–15 hrs/week each → **~25 hrs/week combined** |
| Effective capacity after context-switching, meetings, and life | **~20–23 hrs/week** |
| A conventional 2-person full-time team | ~80 hrs/week |
| **This team runs at roughly one-quarter of that** | |

**In plain terms for the CEO:** a feature a full-time team ships in one week takes us about a month. This is not pessimism — it is arithmetic. Planning against it is the difference between a system GB actually uses in March and a demo that never lands.

**R1 total budget:** 25 calendar weeks × ~23 hrs = **~575 engineer-hours**, against ~460 hours of estimated work plus buffer.

---

## 2. Milestone calendar — Release 1

| Sprint | Duration | Dates | Focus | Hours |
|---|---|---|---|---|
| **S0** | 2 wks | **Sep 14 – Sep 25, 2026** | Discovery & Foundations | 50 |
| **S1** | 4 wks | **Sep 28 – Oct 23, 2026** | Identity, RBAC & Design System | 100 |
| **S2** | 4 wks | **Oct 26 – Nov 20, 2026** | Task Core — API + Web Authoring | 100 |
| **S3** | 4 wks | **Nov 23 – Dec 18, 2026** | iOS Task App | 100 |
| — | 2 wks | **Dec 21 – Jan 1, 2027** | ❄️ Winter break — **no committed work** | 0 |
| **S4** | 3 wks | **Jan 4 – Jan 22, 2027** | Manager Visibility Dashboard | 75 |
| **S5** | 3 wks | **Jan 25 – Feb 12, 2027** | Hardening, QA & TestFlight | 75 |
| **PILOT** | 3 wks | **Feb 15 – Mar 5, 2027** | Live pilot, Warehouse 1 | 75 |
| **🚩 GA** | — | **Mar 8, 2027** | Rollout to all four warehouses | — |

**The winter break is deliberate and non-negotiable.** Students are not productive over the holidays. Planning for zero output in those two weeks is what keeps every date after January honest, instead of a lie we discover in February.

### Hard deadlines the CEO can hold us to

| Date | Commitment |
|---|---|
| **2026-09-25** | Architecture confirmed, repo running, Apple Developer enrollment submitted |
| **2026-10-23** | Any GB employee can log in and see a role-appropriate screen |
| **2026-11-20** | A manager can build tomorrow's task list on the web |
| **2026-12-18** | A warehouse employee can complete tasks on an iPhone, offline |
| **2027-01-22** | A manager can see who did what — yesterday and last week |
| **2027-02-12** | Release candidate on TestFlight, zero P1 bugs |
| **2027-03-05** | Four weeks of real warehouse usage complete |
| **2027-03-08** | **General availability across all four warehouses** |

---

## 3. Sprint detail

### S0 — Discovery & Foundations · Sep 14 – Sep 25 · 50 hrs

**Goal:** remove everything that blocks writing feature code.

| Deliverable | Owner | Hrs |
|---|---|---|
| Monorepo scaffold (pnpm + Turborepo, `apps/web`, `apps/mobile`, `packages/*`) | A | 10 |
| Supabase projects (local, staging, production) + Drizzle wiring | B | 8 |
| GitHub Actions CI: typecheck, lint, test, migration check | A | 8 |
| Sentry + PostHog instrumentation | B | 4 |
| **Apple Developer Program enrollment submitted** *(takes up to 2 weeks — do it day one)* | A | 2 |
| Deliver the Dynamics discovery checklist (EDD Appendix A) to GB | B | 3 |
| Confirm pilot warehouse + manager sponsor with the CEO | Both | 3 |
| Stakeholder review of PRD + EDD; **CEO sign-off on ADR-004** | Both | 6 |
| Buffer | — | 6 |

**Exit criteria:** `pnpm dev` runs web and mobile locally against Supabase · CI green on a trivial PR · Apple enrollment in flight · ADR-004 accepted or rejected **in writing**.

---

### S1 — Identity, RBAC & Design System · Sep 28 – Oct 23 · 100 hrs

**Goal:** the right person sees the right thing, and the database enforces it.

| Deliverable | Owner | Hrs |
|---|---|---|
| Schema migration 001: `organizations`, `sites`, `profiles`, `audit_log` | A | 10 |
| Supabase Auth: login, logout, password reset, session refresh | B | 14 |
| JWT custom claims (`organization_id`, `role`) | B | 8 |
| **RLS policies on every table + the cross-tenant test suite** | A | 18 |
| tRPC setup with role-guarded procedure middleware | B | 10 |
| Shared design tokens; web (Tailwind) + mobile (NativeWind) primitives | A | 14 |
| User administration UI: invite, assign role and site, deactivate | B | 12 |
| i18n scaffolding, EN + ES string extraction | A | 8 |
| Buffer | — | 6 |

**Exit criteria:** all six roles log in on web and iOS · **cross-tenant suite passes at 100% table coverage and gates CI** · a CEO can invite a warehouse employee who lands on a role-correct empty state · the UI renders in Spanish.

> **Why RBAC gets a full month:** PRD acceptance criterion 3 (no cross-tenant leakage) is the one requirement that cannot be fixed later without re-auditing every query written after it. Building the isolation test *before* the features it protects is the highest-leverage 18 hours in this plan.

---

### S2 — Task Core: API + Web Authoring · Oct 26 – Nov 20 · 100 hrs

**Goal:** a manager can create tomorrow's work on the web.

| Deliverable | Owner | Hrs |
|---|---|---|
| Schema migration 002: `task_templates`, `tasks`, `task_completions` | A | 8 |
| Recurrence engine (RFC 5545 RRULE) + nightly materialization job | A | 16 |
| tRPC task procedures: CRUD, assign, bulk create, complete | B | 16 |
| Web: task authoring form (one-off + recurring, person or group) | B | 18 |
| Web: task template library — save, reuse, edit | A | 14 |
| Web: today/tomorrow board with drag-to-reassign | B | 16 |
| Unit tests: recurrence edge cases, permission resolution | Both | 8 |
| Buffer | — | 4 |

**Exit criteria:** **a manager builds a full next-day task list for one warehouse in under 5 minutes** (PRD acceptance criterion 1 — timed with a real GB manager, not with us) · recurring templates materialize correctly across a DST boundary · every task mutation writes to `audit_log`.

---

### S3 — iOS Task App · Nov 23 – Dec 18 · 100 hrs

**Goal:** a warehouse employee closes out their day on a phone, in a dead zone.

| Deliverable | Owner | Hrs |
|---|---|---|
| Expo app shell, navigation, auth flow, **two tabs only** (ADR-006) | A | 12 |
| "My Day" list: today's tasks, priority ordering, overdue emphasis | B | 14 |
| **Local SQLite mirror + durable outbox sync engine** (EDD §6) | A | 24 |
| Complete-task flow: photo capture, note, optimistic UI | B | 16 |
| Offline conflict handling + both-timestamp display | A | 10 |
| Profile tab: account, language toggle, own metrics (RM-3) | B | 10 |
| **EAS build pipeline → TestFlight internal, first build shipped** | A | 8 |
| Maestro E2E: login, complete offline, sync | Both | 8 |

**Exit criteria:** **PRD acceptance criterion 2 verified** — an airplane-mode completion appears on the manager dashboard within 60s of reconnecting · the app is on TestFlight and installable by GB staff · 44pt tap targets confirmed **with gloves on, in the actual warehouse**.

> **Why offline sync gets 24 hours:** it is the highest-risk item in R1. If Wi-Fi dead zones make the app feel broken, the crew stops opening it — and adoption is the top risk in the PRD register.

---

### ❄️ Winter break · Dec 21 – Jan 1 · 0 hrs

No commitments. No "light work." The plan assumes zero output.

---

### S4 — Manager Visibility Dashboard · Jan 4 – Jan 22 · 75 hrs

**Goal:** answer the CEO's original question — *is the work getting done?*

| Deliverable | Owner | Hrs |
|---|---|---|
| Completion aggregation queries + materialized daily rollups | A | 14 |
| Web: live completion board per warehouse | B | 16 |
| Web: per-person history with **evidence drill-down** (RM-2) | A | 14 |
| Web: overdue queue with reassign action | B | 10 |
| Employee-facing view of own metrics, identical to the manager's (RM-3) | A | 8 |
| **Notice at Collection** screen at first login (RM-4) | B | 5 |
| Playwright E2E: 5 critical web flows | Both | 8 |

**Exit criteria:** dashboard loads in <2s on 4G (NFR-5) · every displayed number drills through to the underlying tasks · **no metric in the UI measures presence rather than output** (ADR-004 self-audit) · employee and manager views are provably identical.

---

### S5 — Hardening, QA & TestFlight · Jan 25 – Feb 12 · 75 hrs

**Goal:** a release candidate GB can put in front of real employees.

| Deliverable | Owner | Hrs |
|---|---|---|
| Bug fixing from internal testing | Both | 24 |
| Spanish translation review by a native speaker | B | 6 |
| Security pass: RLS re-audit, signed-URL expiry, dependency scan | A | 12 |
| Performance: dashboard queries, cold app start | B | 10 |
| **Backup restore rehearsal on staging** (EDD §9) | A | 5 |
| App Store submission: metadata, privacy nutrition label, screenshots | B | 8 |
| Pilot materials: one-page EN/ES crew guide, manager walkthrough | Both | 10 |

**Exit criteria:** **zero P1 bugs** · App Store submission accepted, or in review with no blocking feedback · a restore from backup performed successfully at least once · crew guide printed and in the warehouse.

---

### PILOT — Warehouse 1 · Feb 15 – Mar 5 · 75 hrs

**Goal:** find out what we got wrong while it is still cheap.

| Week | Focus |
|---|---|
| **Wk 1** (Feb 15) | On-site launch. **Both engineers physically present on day 1.** The manager sponsor introduces the app as *task clarity*, not monitoring. Daily standup with the sponsor. |
| **Wk 2** (Feb 22) | Rapid fixes via OTA. **Adoption checkpoint: if DAU < 70%, all feature work stops and we fix adoption.** |
| **Wk 3** (Mar 1) | Measure against PRD §9 targets. Retro with crew and manager. GA go/no-go. |

**GA gate — all four must hold:**
1. ≥80% daily task completion recorded, two consecutive weeks
2. ≥90% of warehouse employees logging in on a workday
3. Manager visibility survey ≥4/5
4. Zero open P1 bugs

**If the gate fails:** extend the pilot two weeks rather than roll out a system the first warehouse rejected. A failed rollout across four sites costs far more to recover from than a delayed one at a single site.

---

## 4. Beyond R1 — indicative dates

Planning estimates, not commitments. Each is re-scoped once the preceding release ships.

| Release | Target | Scope | Gating dependency |
|---|---|---|---|
| **R2 — Field Operations** | **2027-05** | Driver pre-trip checks, clock in/out, push notifications, Android | Employment counsel review of PRD §7 |
| **R3 — Data Bridge** | **2027-08** | Read-only Dynamics ingest, inventory lookup, catalog | **D1/D2 — GB nightly export commitment** |
| **R4 — Digital Pick Tickets** | **2027-11** | Digital pick dispatch, barcode scan verification | R3 shipped |
| **R5 — Customer Ordering** | **2028-Q1** | Customer catalog, cart, order submission, billing view | **D3 — clean product catalog** |
| **R6 — Intelligence & Comms** | **2028-Q2** | Chat, CEO analytics, AI task drafting | ADR-004 boundaries; CPRA ADMT compliance |

**R3 is the one that can move.** Its entire schedule rests on GB producing a nightly export (EDD §5.2). If that commitment lands in Sprint 2 as planned, R3 holds. If it slips to mid-2027, R3 and everything behind it slips with it. **This is the single dependency the CEO most directly controls.**

---

## 5. The scope-cut ladder

We will fall behind at some point. Deciding *now* what gets cut prevents deciding badly under pressure later.

**Dates hold. Scope moves.** In this order:

| Cut # | What drops | Deferred to | Cost of cutting |
|---|---|---|---|
| 1 | Drag-to-reassign on the web board | R2 | Low — a dropdown works |
| 2 | Task template library (keep one-off tasks) | R2 | Medium — managers do more typing |
| 3 | Spanish localization | R2 | **High — hurts adoption. Resist.** |
| 4 | Photo proof on completion (note only) | R2 | Medium — weaker evidence trail |
| 5 | Per-person history view (keep the live board only) | R2 | High — this is the CEO's core ask |
| 6 | Pilot narrows to one shift instead of one warehouse | — | Low — still validates |

**Never cut:** RLS and the cross-tenant test suite · offline sync · the audit log · ADR-004 compliance. Each is either a security guarantee, the reason the app gets used at all, or a legal position — and none can be retrofitted cheaply.

---

## 6. Working agreements

**Roles.** No permanent frontend/backend split. Both engineers touch every layer, and ownership rotates each sprint. This costs some speed and buys the only real defense a two-person team has against one person becoming unavailable.

**Review.** Every PR is reviewed by the other engineer — no exceptions, including trivial changes. The review *is* the knowledge transfer.

**Cadence.**
- Monday, 30 min — plan the week, name blockers
- Friday, 30 min — demo whatever runs, update the burn-down
- End of sprint, 1 hr — stakeholder demo with the CEO. **Live software only: no slides, no mockups.**

**Escalation.** Anything blocked more than 3 working days goes to the CEO immediately. At 25 hrs/week, one lost week is 4% of the entire R1 budget.

**Definition of done.** Merged to `main` · CI green including the cross-tenant suite · reviewed by the other engineer · deployed to staging · demoable.

---

## 7. What we need from GB

| # | Need | By | Blocks if late |
|---|---|---|---|
| 1 | CEO written sign-off on this plan and on ADR-004 | **2026-09-25** | Everything |
| 2 | Named pilot warehouse + manager sponsor | **2026-10-09** | Pilot |
| 3 | Apple Developer Program payment ($99/yr) | **2026-09-18** | TestFlight, App Store |
| 4 | Hosting budget approval (~$50–100/mo) | **2026-09-25** | Staging environment |
| 5 | Answers to the Dynamics discovery checklist | **2026-11-06** | R3 |
| 6 | Decision: BYOD or company-provided devices (PRD Q1) | **2026-10-23** | iOS design |
| 7 | 2 hrs of a real manager's time to time-test task authoring | **2026-11-13** | Acceptance criterion 1 |
| 8 | Employment counsel review of PRD §7 | **2027-01-22** | R2 clock-in |

---

## 8. Related documents

- **Product Requirements** — [`docs/PRD.md`](PRD.md)
- **Engineering Design** — [`docs/ENGINEERING-DESIGN.md`](ENGINEERING-DESIGN.md)
- **Build Orchestration** — [`docs/ORCHESTRATION.md`](ORCHESTRATION.md)
