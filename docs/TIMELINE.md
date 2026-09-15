# The Schedule

**Product:** OpsLink — iOS operations app
**Version:** 3.0 — 2026-09-14
**Companion to:** [PRODUCT.md](PRODUCT.md) · [ENGINEERING.md](ENGINEERING.md) · [OPERATIONS-TODAY.md](OPERATIONS-TODAY.md)

> **What changed in 3.0.** The roadmap is now built on the **CEO's six phases** instead of our own R1–R6 lettering. The dates were **re-derived from the capacity arithmetic**, not relabelled — and re-deriving them moved general availability from **2027-02-22 to 2027-03-01**. [§3](#3-what-the-phase-order-costs) shows exactly where that week went, because a date that moves without a reason is a date nobody should trust.

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

That isn't pessimism, it's arithmetic. Planning against the real number is the difference between software GB is actually using in March and a demo that never lands.

**MVP budget:**

| | Hours |
|---|---|
| 22 working weeks × ~22 hrs | 484 |
| Less the Thanksgiving week, realistically half a week | −11 |
| **Actually available** | **473** |
| **Planned work** (44 + 209 + 110 + 44 + 66) | **473** |
| **Buffer** | **0** |

**There is no buffer. Read that again, because it is the most important number on this page.**

Every hour between now and the trial is already spoken for. That does not mean the plan is wrong — it means the plan has exactly one shock absorber, and it is **scope**, not time. The first thing that goes wrong comes straight out of [§6, the cut list](#6-what-gets-cut-first), immediately, without a meeting.

If that is not acceptable, the honest fixes are to move the launch date or to add a third pair of hands. Pretending a buffer exists is not on the list.

---

## 2. What dropping the website saved

Carried forward from version 2.0, still true.

**Removed with the website:** shared-code project setup (15) · the API layer (26) · a second design system (14) · separate browser login flows (8) · browser end-to-end tests (8) · web hosting and deployment (5) — **76 hours.**

**But some work got harder:** dense dashboards and tables are genuinely harder in SwiftUI than in HTML (16) · making every screen work on both iPhone and iPad (10) — **26 hours.**

**Net saving: about 50 hours — roughly two weeks.**

**Why it isn't more.** The manager features didn't go away. Building tomorrow's task list, the completion board, the per-person history — all of it still has to be built. It just gets built on an iPad instead of in a browser. Dropping the website removed a *second place to build things*, not the things themselves.

---

## 3. What the phase order costs

The CEO's brief runs **interface first, logins second**. Version 2.0 ran the other way. That reordering is a real choice with a real price, and here it is.

| | Hours |
|---|---|
| Building Phase 1 screens against a development role switch instead of real identity | 0 — this part is free |
| **Revisiting those screens when real identity and database-level permissions land in Phase 2** | **+20** |
| Net effect on the schedule | **≈ 1 week** |

**Why it's worth paying.** Interface-first means there is something to demo in October rather than January. With a two-person part-time team and a CEO who has never seen the thing he's funding, that matters more than twenty hours does.

**What we're not pretending.** Screens built before permissions exist *will* need changing when permissions arrive. Anyone who tells you otherwise has not done it. Twenty hours is the honest estimate, it is in the budget above, and it is why general availability is **2027-03-01** rather than **2027-02-22**.

**Where the order genuinely worries us: Phase 3.** The brief places Dynamics integration third, ahead of the management dashboard. Dynamics is the single least controllable piece of work in the project — the version is uncertain, nobody has yet said who administers it, and the entire integration depends on GB producing an export file. Putting it on the critical path would mean the CEO's own top priority waits behind the one thing he would have to unblock himself.

**So we don't.** Phase 3 is planned as a **dependency-gated parallel track**: it starts the day GB's export exists, and if that's late it runs beside Phase 4 rather than in front of it. The phase *number* is the CEO's. The *sequencing* protects him from his own ordering.

---

## 4. The calendar

| Phase | Length | Dates | What we're building | Hours |
|---|---|---|---|---|
| **P0** | 2 wks | **Sep 14 – Sep 25, 2026** | Foundations & the Swift go/no-go | 44 |
| **P1** | 10 wks | **Sep 28 – Dec 4, 2026** | The interface — shell and the task loop | 209 |
| **P2** | 2 wks | **Dec 7 – Dec 18, 2026** | Logins and roles *(part 1)* | 44 |
| — | 2 wks | **Dec 21 – Jan 1, 2027** | ❄️ **Winter break — nothing committed** | 0 |
| **P2** | 3 wks | **Jan 4 – Jan 22, 2027** | Logins and roles *(part 2)* | 66 |
| **Hardening** | 2 wks | **Jan 25 – Feb 5, 2027** | Bug fixing, polish, App Store | 44 |
| **TRIAL** | 3 wks | **Feb 8 – Feb 26, 2027** | Live in one warehouse | 66 |
| **🚩 LAUNCH** | — | **2027-03-01** | All four warehouses | — |
| **P3** | — | *gated — earliest 2027-03* | Dynamics data, read-only | TBD |
| **P4** | — | *~2027-Q2* | Management dashboard | TBD |
| **P5** | — | *~2027-Q4 / 2028-Q1* | Customer experience | TBD |
| **P6** | — | *2028* | Quality of life | TBD |

**The winter break is deliberate and not negotiable.** Students are not productive over the holidays. Planning for zero output in those two weeks is exactly what keeps every date after January honest, instead of a fiction we discover in February.

**Why Phase 2 is split across the break.** The first half is authentication and the permission model — self-contained database work that doesn't need to be held in anyone's head over Christmas. The screen work resumes cleanly in January.

### Dates the CEO can hold us to

| Date | What will be true |
|---|---|
| **2026-09-25** | The technical approach is confirmed and tested, the project runs, Apple enrollment is submitted |
| **2026-10-16** | The app shell exists and a task can be created and seen |
| **2026-11-13** | A manager can build tomorrow's task list on an iPad |
| **2026-12-04** | A warehouse employee can complete tasks on an iPhone, with no signal, and a manager sees the live board |
| **2027-01-22** | Any GB employee logs in as themselves and sees a screen appropriate to their job; company isolation is proven in CI |
| **2027-02-05** | The release candidate is on TestFlight with zero serious bugs |
| **2027-02-26** | Three weeks of real warehouse use are complete |
| **2027-03-01** | **Live across all four warehouses** |

---

## 5. Phase by phase

### P0 — Foundations & the Swift go/no-go · Sep 14–25 · 44 hrs

**Goal:** clear everything blocking real feature work, and answer one question honestly before it's expensive.

The question is whether this team is actually productive in Swift. We answer it by building a real screen against the real database — not by discussing it. If it's painful, we switch approach now, while switching is free.

Also: Apple Developer enrollment submitted, the staging environment up, the repository and review process running, and the first database migration with `organization_id` on every table.

**Exit:** a go/no-go on Swift, in writing.

---

### P1 — The interface · Sep 28 – Dec 4 · 209 hrs

**Goal:** the task loop works end to end, for one seeded company, with a development role switch standing in for real logins.

| Work | Hours |
|---|---|
| The shell — navigation, design language, iPhone and iPad layouts | 36 |
| Task model, creation and assignment on iPad | 45 |
| My Day and completion with photo or note on iPhone | 40 |
| Offline queue, sync, and conflict handling | 38 |
| Saved checklists and recurrence | 20 |
| The live board | 18 |
| Spanish | 12 |
| **Total** | **209** |

**Note the Thanksgiving week (Nov 23–27) is budgeted at half.** It always is in practice; the only choice is whether the plan admits it.

**Exit:** a manager creates tomorrow's list on an iPad, an employee completes it on an iPhone in airplane mode, and the live board shows it once signal returns.

---

### P2 — Logins and roles · Dec 7–18 and Jan 4–22 · 110 hrs

**Goal:** every person is themselves, and the database — not the app — decides what they can touch.

| Work | Hours |
|---|---|
| Authentication: login, password reset, session persistence | 22 |
| Three buckets, job tags, warehouse scope, and the database policies behind them | 30 |
| **Retrofitting Phase 1 screens onto real identity** *(the cost from [§3](#3-what-the-phase-order-costs))* | 20 |
| Account admin: invite, assign bucket and tags, deactivate | 24 |
| Company isolation test suite, running on every code change | 14 |
| **Total** | **110** |

**Exit:** two companies exist in the database, and an automated test proves neither can see the other. That test never gets deleted.

---

### Hardening · Jan 25 – Feb 5 · 44 hrs

Bug fixing, performance on real cell data, App Store submission, TestFlight build to the trial warehouse, and the trial-day runbook. No new features. None.

---

### TRIAL — one warehouse · Feb 8–26 · 66 hrs

Three weeks of real use by a real crew. Most of these hours are support and fixes, not features.

**Week 1** — on-site for the first two mornings. Watch people use it. Do not explain it to them; watching someone be confused is the data.
**Week 2** — the 70% daily-open threshold. Below it, feature work stops and adoption work starts.
**Week 3** — the 80% completion bar over two consecutive weeks, and a go/no-go on launching to all four warehouses.

---

## 6. What gets cut first

We will fall behind at some point. Deciding **now** what gets dropped prevents deciding badly under pressure in January.

**Dates hold. Scope moves.** In this order:

| Cut # | What goes | Moves to | What it costs us |
|---|---|---|---|
| 1 | Drag-to-reassign on the board | P6 | Low — a dropdown menu works fine |
| 2 | Saved checklist library (keep one-off tasks) | P6 | Medium — managers do more typing |
| 3 | Photo proof on completion (note only) | P6 | Medium — weaker evidence trail |
| 4 | Recurrence rules (keep one-off tasks only) | P6 | Medium — daily routines get rebuilt by hand |
| 5 | Spanish | P6 | **High — this hurts adoption badly. Resist hard.** |
| 6 | Per-person history (keep the live board only) | P4 | High — this is the CEO's core ask |
| 7 | Trial narrows to one shift instead of one warehouse | — | Low — still proves the point |

**Never cut, under any circumstances:**
- The database security rules and the cross-company test
- Offline sync
- The audit log
- The workforce policy commitments

Each of those is either a security guarantee, the reason the app gets used at all, or a legal position — and none of them can be added back cheaply later.

---

## 7. After launch

Planning estimates, not commitments. Each gets re-scoped once the one before it ships.

| Phase | Target | What's in it | What it depends on |
|---|---|---|---|
| **P3 — Dynamics data** | **gated, earliest 2027-03** | Read-only nightly import, stock lookup, product and customer data | **GB committing to a nightly export file**, and an answer to *who administers Dynamics* |
| **P4 — Management dashboard** | **~2027-Q2** | Per-warehouse trends, per-person history, cross-warehouse comparison, daily reports | P2 shipped and enough task history to have trends at all. Inventory and purchase analysis additionally need P3 |
| **P5 — Customer experience** | **~2027-Q4 / 2028-Q1** | Customer catalog, cart, ordering, order history, billing view | **A product catalog that does not currently exist** + the website decision ([PRODUCT.md §9](PRODUCT.md#9-the-website-question)) |
| **P6 — Quality of life** | **2028** | Chat, notifications, driver pre-trip hardening, clock-in, AI task drafting, Android | Employment-counsel review for clock-in; California ADMT compliance for anything AI-assisted |

**P3 is the one most likely to move,** and it's the one the CEO most directly controls. Its entire schedule rests on GB producing a nightly export file. It is planned as a parallel track precisely so that its slipping doesn't drag P4 with it.

**P5 has a blocker that isn't software.** There is no product catalog at GB — see [OPERATIONS-TODAY.md §6.1](OPERATIONS-TODAY.md#61-there-is-no-product-catalog). Someone has to write down what GB sells, with units and pack sizes, before a customer can browse it. If that work has no owner by **2027-03-01**, P5 has no start date either.

**P5 is also where the website question comes due.** If we're going to build a web ordering surface, the decision needs making by roughly mid-2027 to land in the P5 window.

---

## 8. How we work

**Roles.** No permanent split between us. Both engineers touch every part of the system, and ownership rotates each phase. This costs some speed. It buys the only real protection a two-person team has against one person becoming unavailable.

**Review.** Every change is reviewed by the other person — no exceptions, including trivial ones. The review *is* how knowledge transfers.

**Rhythm.**
- **Monday, 30 min** — plan the week, name what's blocked
- **Friday, 30 min** — demo whatever actually runs, update the burn-down
- **Monthly, 1 hr** — demo to the CEO. **Working software only. No slides, no mockups.**

**Escalation.** Anything blocked for more than 3 working days goes to the CEO immediately. At 25 hours a week, one lost week is 4% of the entire MVP budget.

**"Done" means.** Merged · all automated checks green, including the cross-company test · reviewed by the other engineer · deployed to staging · demonstrable.

---

## 9. What we need from GB, with dates

| # | What | By | What breaks if it's late |
|---|---|---|---|
| 1 | Written sign-off on this plan and the workforce policy | **2026-09-25** | Everything |
| 2 | Apple Developer Program paid ($99/yr) | **2026-09-18** | TestFlight, App Store |
| 3 | Hosting budget approved (~$25–50/mo) | **2026-09-25** | The staging environment |
| 4 | Named trial warehouse + manager sponsor | **2026-10-09** | The trial |
| 5 | Decision: personal phones or company phones | **2026-10-16** | The phone app design |
| 6 | Answers to the Dynamics questionnaire | **2026-11-06** | All of P3 |
| 7 | 2 hours of a real manager's time to time-test task building | **2026-11-06** | Proving the "under 5 minutes" claim |
| 8 | **New:** a walkthrough of the real purchasing and receiving process | **2026-12-04** | Every purchasing feature — the current description is an admitted guess |
| 9 | Employment lawyer review of the workforce policy | **2027-01-15** | P6 clock-in |
| 10 | Two iPads for the trial warehouse | **2027-02-01** | The trial |
| 11 | **New:** a named owner and date for building the product catalog | **2027-03-01** | All of P5 |

---

## Related documents

- **[FOR-THE-CEO.md](FOR-THE-CEO.md)** — the plain-English overview
- **[PRODUCT.md](PRODUCT.md)** — what we're building
- **[OPERATIONS-TODAY.md](OPERATIONS-TODAY.md)** — how GB runs today
- **[ENGINEERING.md](ENGINEERING.md)** — how it's built
- **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)** — what we measure and what we won't
