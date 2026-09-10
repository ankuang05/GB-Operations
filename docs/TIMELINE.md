# The Schedule

**Product:** OpsLink — iOS operations app
**Version:** 2.0 — 2026-09-09
**Companion to:** [PRODUCT.md](PRODUCT.md) · [ENGINEERING.md](ENGINEERING.md)

---

## 1. The number this whole plan rests on

Every date below comes from one figure, and it should be visible to anyone reading this:

| | |
|---|---|
| The team | **2 people, part-time** (students, side project) |
| Hours each, per week | ~10–15 |
| **Combined** | **~25 hours/week** |
| Realistically, after context-switching and life | **~22 hours/week** |
| A normal full-time pair of engineers | ~80 hours/week |
| **So this team runs at about a quarter speed** | |

**In plain terms:** something a full-time team ships in one week takes us about a month.

That isn't pessimism, it's arithmetic. Planning against the real number is the difference between software GB is actually using in February and a demo that never lands.

**Release 1 budget:** 21 working weeks × ~22 hrs = **~460 hours available**, against ~450 hours of planned work. The buffer is thin on purpose — see [§5, the cut list](#5-what-gets-cut-first).

---

## 2. What dropping the website actually saved

I want to show this rather than assert it, because the honest answer is smaller than you'd expect.

**Removed with the website:**

| Work that disappeared | Hours |
|---|---|
| Shared-code project setup (monorepo, build tooling) | 15 |
| The API layer between the apps and the database | 26 |
| A second design system (web styling *and* mobile styling) | 14 |
| Separate login flows for the browser | 8 |
| Browser end-to-end tests | 8 |
| Web hosting and deployment pipeline | 5 |
| **Total removed** | **76** |

**But some work got *harder*:**

| New cost | Hours |
|---|---|
| Dense dashboards and tables are genuinely harder in SwiftUI than in HTML | 16 |
| Making every screen work on both iPhone and iPad | 10 |
| **Total added** | **26** |

**Net saving: about 50 hours — roughly two weeks.**

**Why it isn't more.** The manager features didn't go away. Building tomorrow's task list, the completion board, the per-person history — all of it still has to be built. It just gets built on an iPad instead of in a browser. Dropping the website removed a *second place to build things*, not the things themselves.

**What that means for dates:** general availability moves from **2027-03-08 → 2027-02-22.**

---

## 3. The calendar

| Sprint | Length | Dates | What we're building | Hours |
|---|---|---|---|---|
| **S0** | 2 wks | **Sep 14 – Sep 25, 2026** | Foundations & the Swift go/no-go | 50 |
| **S1** | 3 wks | **Sep 28 – Oct 16, 2026** | Logins, roles & the look of the app | 75 |
| **S2** | 4 wks | **Oct 19 – Nov 13, 2026** | Tasks & the manager's iPad | 100 |
| **S3** | 4 wks | **Nov 16 – Dec 11, 2026** | The crew's iPhone app & offline | 100 |
| **S4a** | 1 wk | **Dec 14 – Dec 18, 2026** | The numbers behind the dashboard | 25 |
| — | 2 wks | **Dec 21 – Jan 1, 2027** | ❄️ **Winter break — nothing committed** | 0 |
| **S4b** | 2 wks | **Jan 4 – Jan 15, 2027** | The dashboard itself | 50 |
| **S5** | 2 wks | **Jan 18 – Jan 29, 2027** | Bug fixing, polish, App Store | 50 |
| **TRIAL** | 3 wks | **Feb 1 – Feb 19, 2027** | Live in one warehouse | 75 |
| **🚩 LAUNCH** | — | **Feb 22, 2027** | All four warehouses | — |

**The winter break is deliberate and not negotiable.** Students are not productive over the holidays. Planning for zero output in those two weeks is exactly what keeps every date after January honest, instead of a fiction we discover in February.

**Why S4 is split across the break.** The week before the break is pure database work — the queries and rollups behind the dashboard. It's self-contained, it doesn't need to be held in anyone's head over Christmas, and the screen work resumes cleanly in January.

### Dates the CEO can hold us to

| Date | What will be true |
|---|---|
| **2026-09-25** | The technical approach is confirmed and tested, the project runs, Apple enrollment is submitted |
| **2026-10-16** | Any GB employee can log in and see a screen appropriate to their job |
| **2026-11-13** | A manager can build tomorrow's task list on an iPad |
| **2026-12-11** | A warehouse employee can complete tasks on an iPhone, with no signal |
| **2027-01-15** | A manager can see who did what — yesterday and last week |
| **2027-01-29** | The release candidate is on TestFlight with zero serious bugs |
| **2027-02-19** | Three weeks of real warehouse use are complete |
| **2027-02-22** | **Live across all four warehouses** |

---

## 4. Sprint by sprint

### S0 — Foundations & the Swift go/no-go · Sep 14–25 · 50 hrs

**Goal:** clear everything blocking real feature work, and answer one question honestly before it's expensive.

| What | Who | Hrs |
|---|---|---|
| **The Swift spike** — build one real screen, logging into the real database, running on a real iPhone | Both | 12 |
| Xcode project set up, dependencies, code style, folder structure | A | 8 |
| Supabase projects created (local, staging, production) + first migration | B | 8 |
| Automated checks running on every change | A | 8 |
| Crash reporting and usage analytics wired in | B | 3 |
| **Apple Developer enrollment submitted** *(can take 2 weeks — do it day one)* | A | 2 |
| Send GB the Dynamics questionnaire ([ENGINEERING.md §10](ENGINEERING.md#10-appendix--dynamics-questionnaire)) | B | 2 |
| Confirm the trial warehouse and manager sponsor | Both | 3 |
| Review these documents with the CEO; **get the workforce policy signed off** | Both | 4 |

**⚠️ The gate at the end of Sprint 0.** The Swift spike either goes well or it doesn't.

- **Goes well** → we proceed exactly as written.
- **Goes badly** → we switch to React Native in Sprint 1, add roughly **3 weeks**, and launch moves to mid-March.

Either way, we decide **before writing real code**, when switching costs days instead of months. This is the single most valuable thing in Sprint 0.

**Done when:** the app builds and runs against Supabase · checks pass on a trivial change · Apple enrollment is in flight · the workforce policy is accepted or rejected **in writing** · the Swift decision is made.

---

### S1 — Logins, roles & the look of the app · Sep 28 – Oct 16 · 75 hrs

**Goal:** the right person sees the right thing, and the database is what enforces it.

| What | Who | Hrs |
|---|---|---|
| First migration: companies, sites, people, audit log | A | 8 |
| Login, logout, password reset, staying signed in | B | 12 |
| Role and company attached to the login token | B | 6 |
| **Security rules on every table + the cross-company test suite** | A | 18 |
| Shared visual components; iPhone and iPad layout scaffolding | A | 12 |
| Account admin: invite someone, set their role and warehouse, deactivate | B | 11 |
| English/Spanish framework and text extraction | A | 8 |

**Done when:** all six roles can log in · **the cross-company test passes on 100% of tables and blocks any change that breaks it** · a CEO can invite a warehouse employee who lands on a correct (empty) screen · the app renders in Spanish.

> **Why security gets 18 hours before any feature exists.** "One company can't see another's data" is the one requirement that can't be fixed later without re-auditing every single query written after it. Building the test *before* the features it protects is the highest-value 18 hours in this entire plan.

---

### S2 — Tasks & the manager's iPad · Oct 19 – Nov 13 · 100 hrs

**Goal:** a manager can create tomorrow's work.

| What | Who | Hrs |
|---|---|---|
| Second migration: templates, tasks, completions | A | 8 |
| Repeating-task engine + the nightly job that creates tomorrow's list | A | 16 |
| Database rules for tasks: who can create, assign, complete | B | 14 |
| iPad: the task-building screen (one-off and repeating, person or group) | B | 20 |
| iPad: saved checklist library — save, reuse, edit | A | 16 |
| iPad: today/tomorrow board with drag-to-reassign | B | 16 |
| Tests: repeat rules, edge cases, permissions | Both | 10 |

**Done when:** **a real GB manager builds a full next-day list for one warehouse in under 5 minutes** (timed with them, not with us) · repeating checklists work correctly across a daylight-saving change · every task change is recorded in the audit log.

---

### S3 — The crew's iPhone app & offline · Nov 16 – Dec 11 · 100 hrs

**Goal:** a warehouse employee finishes their day on a phone, in a dead zone.

| What | Who | Hrs |
|---|---|---|
| App shell, navigation, login, **two tabs only** ([D9](ENGINEERING.md#7-the-decisions-we-made)) | A | 10 |
| "My Day": today's tasks, priority order, overdue emphasis | B | 14 |
| **Local database + the offline queue** ([ENGINEERING.md §5](ENGINEERING.md#5-working-offline)) | A | 26 |
| Completing a task: photo, note, instant response | B | 16 |
| Offline conflict handling + showing both timestamps | A | 10 |
| Profile tab: account, language switch, your own numbers | B | 10 |
| **Build pipeline → first TestFlight build shipped** | A | 8 |
| End-to-end tests: log in, complete offline, sync | Both | 6 |

**Done when:** **a task completed in airplane mode appears on the manager's iPad within 60 seconds of reconnecting** · the app is on TestFlight and installable by GB staff · **44-point tap targets confirmed with gloves on, in an actual warehouse**.

> **Why offline sync gets 26 hours.** It's the highest-risk item in Release 1. If Wi-Fi dead zones make the app feel broken, the crew stops opening it — and getting people to use it is the top risk in the entire project.

---

### S4a — The numbers behind the dashboard · Dec 14–18 · 25 hrs

**Goal:** finish the self-contained database work before the break.

| What | Who | Hrs |
|---|---|---|
| Completion aggregation queries and nightly rollups | A | 14 |
| Performance work: indexes, query plans, cached daily summaries | B | 8 |
| Write down where things stand, for our January selves | Both | 3 |

---

### ❄️ Winter break · Dec 21 – Jan 1 · 0 hrs

No commitments. No "light work." The plan assumes zero output.

---

### S4b — The dashboard itself · Jan 4–15, 2027 · 50 hrs

**Goal:** answer the CEO's original question — *is the work getting done?*

| What | Who | Hrs |
|---|---|---|
| iPad: live completion board per warehouse | B | 14 |
| iPad: each person's history, **with drill-down to the evidence** | A | 12 |
| iPad: overdue list with a reassign action | B | 8 |
| Employees see their own numbers, identical to what the manager sees | A | 8 |
| **The privacy notice shown at first login** | B | 4 |
| End-to-end tests for the manager journeys | Both | 4 |

**Done when:** the dashboard loads in under 2 seconds on cell data · every number on screen can be tapped through to the tasks behind it · **no number anywhere measures presence rather than output** (a deliberate self-audit against [WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)) · the employee view and the manager view are provably the same.

---

### S5 — Bug fixing, polish, App Store · Jan 18–29 · 50 hrs

**Goal:** something GB can put in front of real employees.

| What | Who | Hrs |
|---|---|---|
| Fixing what internal testing found | Both | 16 |
| Spanish reviewed by a native speaker | B | 5 |
| Security pass: re-audit the rules, check photo link expiry, scan dependencies | A | 10 |
| Speed: dashboard queries, app cold start | B | 6 |
| **Practice restoring the database from backup** | A | 4 |
| App Store submission: description, privacy labels, screenshots | B | 5 |
| Trial materials: a one-page crew guide in English and Spanish, manager walkthrough | Both | 4 |

**Done when:** **zero serious bugs** · App Store submission accepted, or in review with nothing blocking · a backup restored successfully at least once · the crew guide printed and physically in the warehouse.

---

### TRIAL — one warehouse · Feb 1–19 · 75 hrs

**Goal:** find out what we got wrong while it's still cheap to fix.

| Week | What happens |
|---|---|
| **Week 1** (Feb 1) | Launch on site. **Both engineers physically present on day one.** The manager sponsor introduces it as *knowing what to do today*, not as monitoring. Daily 10-minute check-in with the sponsor. |
| **Week 2** (Feb 8) | Rapid fixes. **Adoption checkpoint: if fewer than 70% of the crew are opening it daily, all feature work stops and we fix adoption.** |
| **Week 3** (Feb 15) | Measure against the targets. Sit down with the crew and the manager. Go/no-go for full launch. |

**All four must be true to launch everywhere:**

1. 80%+ of assigned daily tasks completed and recorded, two weeks running
2. 90%+ of warehouse employees logging in on a workday
3. Manager rates their visibility 4 out of 5 or better
4. Zero serious bugs open

**If it doesn't pass:** we extend the trial by two weeks rather than roll out a system the first warehouse rejected. A failed launch across four sites costs far more to recover from than a delayed one at a single site.

---

## 5. What gets cut first

We will fall behind at some point. Deciding **now** what gets dropped prevents deciding badly under pressure in January.

**Dates hold. Scope moves.** In this order:

| Cut # | What goes | Moves to | What it costs us |
|---|---|---|---|
| 1 | Drag-to-reassign on the board | R2 | Low — a dropdown menu works fine |
| 2 | Saved checklist library (keep one-off tasks) | R2 | Medium — managers do more typing |
| 3 | Photo proof on completion (note only) | R2 | Medium — weaker evidence trail |
| 4 | Spanish | R2 | **High — this hurts adoption badly. Resist hard.** |
| 5 | Per-person history (keep the live board only) | R2 | High — this is the CEO's core ask |
| 6 | Trial narrows to one shift instead of one warehouse | — | Low — still proves the point |

**Never cut, under any circumstances:**
- The database security rules and the cross-company test
- Offline sync
- The audit log
- The workforce policy commitments

Each of those is either a security guarantee, the reason the app gets used at all, or a legal position — and none of them can be added back cheaply later.

---

## 6. After Release 1

Planning estimates, not commitments. Each gets re-scoped once the one before it ships.

| Release | Target | What's in it | What it depends on |
|---|---|---|---|
| **R2 — Out in the Field** | **2027-04** | Driver pre-trip checks, clock in/out, notifications, Android | Employment lawyer review of the workforce policy |
| **R3 — Dynamics Connection** | **2027-07** | Read-only nightly import, stock lookup, catalog | **GB committing to a nightly export file** |
| **R4 — Digital Pick Tickets** | **2027-10** | Digital pick dispatch, barcode scan verification | R3 shipped |
| **R5 — Customer Ordering** | **2028-Q1** | Customer catalog, cart, ordering, billing view | A clean product catalog **+ the website decision** ([PRODUCT.md §9](PRODUCT.md#9-the-website-question)) |
| **R6 — Reports & Messaging** | **2028-Q2** | Chat, CEO analytics, AI task drafting | Workforce policy boundaries; California ADMT compliance |

**R3 is the one most likely to move,** and it's the one the CEO most directly controls. Its entire schedule rests on GB producing a nightly export file. If that commitment lands in October as planned, R3 holds. If it slips to mid-2027, R3 and everything behind it slips with it.

**R5 is where the website question comes due.** If we're going to build a web ordering surface, the decision needs making by roughly mid-2027 to land in the R5 window.

---

## 7. How we work

**Roles.** No permanent split between us. Both engineers touch every part of the system, and ownership rotates each sprint. This costs some speed. It buys the only real protection a two-person team has against one person becoming unavailable.

**Review.** Every change is reviewed by the other person — no exceptions, including trivial ones. The review *is* how knowledge transfers.

**Rhythm.**
- **Monday, 30 min** — plan the week, name what's blocked
- **Friday, 30 min** — demo whatever actually runs, update the burn-down
- **End of each sprint, 1 hr** — demo to the CEO. **Working software only. No slides, no mockups.**

**Escalation.** Anything blocked for more than 3 working days goes to the CEO immediately. At 25 hours a week, one lost week is 4% of the entire Release 1 budget.

**"Done" means.** Merged · all automated checks green, including the cross-company test · reviewed by the other engineer · deployed to staging · demonstrable.

---

## 8. What we need from GB, with dates

| # | What | By | What breaks if it's late |
|---|---|---|---|
| 1 | Written sign-off on this plan and the workforce policy | **2026-09-25** | Everything |
| 2 | Apple Developer Program paid ($99/yr) | **2026-09-18** | TestFlight, App Store |
| 3 | Hosting budget approved (~$25–50/mo) | **2026-09-25** | The staging environment |
| 4 | Named trial warehouse + manager sponsor | **2026-10-09** | The trial |
| 5 | Decision: personal phones or company phones | **2026-10-16** | The phone app design |
| 6 | Answers to the Dynamics questionnaire | **2026-11-06** | All of Release 3 |
| 7 | 2 hours of a real manager's time to time-test task building | **2026-11-06** | Proving the "under 5 minutes" claim |
| 8 | Employment lawyer review of the workforce policy | **2027-01-15** | R2 clock-in |
| 9 | Two iPads for the trial warehouse | **2027-01-22** | The trial |

---

## Related documents

- **[FOR-THE-CEO.md](FOR-THE-CEO.md)** — the plain-English overview
- **[PRODUCT.md](PRODUCT.md)** — what we're building
- **[ENGINEERING.md](ENGINEERING.md)** — how it's built
- **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)** — what we measure and what we won't
