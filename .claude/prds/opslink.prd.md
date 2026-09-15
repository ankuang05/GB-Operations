# OpsLink — Operations App for Goldberry Distributors

**Owner:** Andy Kuang
**Version:** 1.0 — 2026-09-14
**Status:** DRAFT — requirements only. Implementation planning pending via `/plan`.
**Source of record:** the CEO's project brief (Sep 2026) + the planning suite in `docs/`

---

## Problem

Goldberry Distributors runs a four-warehouse food distribution business on phone calls, handwritten order notes, a Microsoft Dynamics 2009 install, and printed paper at every handoff. Nothing between "a customer calls" and "the signed invoice comes back to the office" leaves a queryable trace, so the CEO cannot answer *is the work getting done* without walking the floor or calling someone.

The cost is paid three ways: hours of re-keying and re-typing that produce transcription errors, a full salary spent on a QC person whose job is to re-check work that was never recorded in the first place, and a CEO who is personally the routing system, the escalation path, and the only person with a whole-company view.

## Evidence

**Verified — documented current-state workflow** (see `docs/OPERATIONS-TODAY.md`):

- The sales-order path has **six** manual handoffs — phone/text/WeChat/email intake, handwritten note, Dynamics entry, typed-and-printed pick ticket, QC re-check, signed paper invoice returned to the office.
- Orders arrive on at least four uncoordinated channels. The CEO reports **simultaneous inbound calls that staff cannot answer at once**, and orders are lost as a result.
- **The CEO personally determines truck routes.** Routes are stable every N days; he re-plans manually whenever a driver is out sick. This is a named single point of failure in daily operations.
- **There is no product catalog.** Not a database, not a spreadsheet — customers know what GB sells because they have bought it for years.
- Dynamics is on a version from roughly 2009, predating any modern integration surface.

**CEO-reported pain, not yet independently measured:**

- Drivers skipping pre-trip checks (tire pressure, fuel).
- Warehouse staff idle on the clock — extended bathroom absences, chatting before clocking out.

**Explicitly unverified — flagged as an assumption in the CEO's own brief:**

- The entire purchase-order and receiving flow. The brief states outright that how GB actually runs purchasing has not been determined. Every requirement touching purchasing, receiving, or put-away is therefore `TBD — needs validation via a purchasing/receiving process walkthrough on site`.

**Competitive reference:** OMELINK, a mobile app in the same market, which the CEO cites as inspiration. Notably it ships **separate** customer-facing and employee-facing experiences — a direct tension with this product's single-shell goal, and an open design question rather than a settled one.

## Users

**Primary — three login buckets**, per the CEO's brief:

| Bucket | Who | What triggers the need |
|---|---|---|
| **Management** | CEO and managers. Also take customer orders and place vendor orders. | Start of day: needs to assign work and, later, see whether yesterday's got done. CEO's scope is all four warehouses; a manager's is one. |
| **Employee** | Warehouse floor — picker/stager, QC, driver, receiver. "Task followers" in the CEO's words. | Start of shift: needs to know what to do today, and to record that each item is finished. |
| **Customer** | Meat markets, supermarkets, restaurants, other distributors. | Reorder time — currently means calling and possibly being put on hold. |

Within **Employee**, the specific job (picker, stager, QC, driver, receiver, buyer) is a **tag on the person**, not a separate login role. This follows the brief's flattening of six roles to three and keeps the permission surface small; a tag is promoted to a full role only when a phase genuinely needs different permissions.

**Not for:** GB's vendors (no vendor portal), GB's accounting function (Dynamics stays the money system of record), and any user who needs a browser rather than an iPhone or iPad.

## Hypothesis

We believe **one adaptive iOS app that assigns work, records its completion, and reports it by warehouse** will **give GB a queryable record of daily operations in place of paper and recollection** for **management and warehouse employees**.

We'll know we're right when **a trial warehouse records 80%+ of assigned daily tasks as completed in the app for two consecutive weeks, and a manager builds the next day's list in under 5 minutes.**

## Success Metrics

| Metric | Target | How measured |
|---|---|---|
| **Assigned daily tasks completed and recorded** (primary) | ≥ 80% for 2 consecutive weeks | In-app task completion records, trial warehouse |
| Time for a manager to build the next day's list | < 5 min (from ~30 min verbal/paper) | Timed observation with a real manager |
| Warehouse employees opening the app on a workday | ≥ 90% | Daily active logins ÷ scheduled headcount |
| Manager-reported visibility vs. before | ≥ 4 / 5 | Post-trial interview |
| Serious defects open at end of trial | 0 | Defect tracker |
| Cross-company data isolation | Provable on every code change | Automated test in CI |

**The weekly leading indicator:** daily app opens. If that sits below 70% in trial week 2, feature work stops and adoption work starts. An accountability system nobody opens is worse than none, because it produces confidence in numbers that aren't real.

## Scope

The brief defines six phases. They are the spine of this product, in the CEO's order.

**MVP — Phases 1 and 2.** An employee opens the app on their own credentials and sees today's work; a manager builds and assigns that work; both hold up on a warehouse floor with no signal. That is the minimum that tests the hypothesis, and it needs nothing from Dynamics.

| Phase | Outcome |
|---|---|
| **P1 — The interface** | The shell and the task loop exist and are usable: create work, see today's work, complete it with photo or note, offline. |
| **P2 — Logins and roles** | Real credentials per person. Management / Employee / Customer buckets with job tags. One company's data provably invisible to another. |
| **P3 — Dynamics data** | Products, stock, and customers readable in the app from a nightly **read-only** copy. |
| **P4 — Management dashboard** | Per-warehouse view of what got done, by whom, and what's overdue. |
| **P5 — Customer experience** | Customers browse and order; order history and billing view. |
| **P6 — Quality of life** | Chat, notifications, driver pre-trip checklist hardening, clock-in, reporting depth. |

**Out of scope**

| Item | Why deferred |
|---|---|
| **Writing back into Dynamics** | Decided: read-only for now. Writing into a 20-year-old ERP that runs a live business is how the live business goes down. Revisited as a candidate later stage, gated on a capability spike proving Dynamics 2009 accepts input safely. |
| **A website** | Owner's direction is the app. Kept cheap to add later by keeping business rules in the database rather than in the app. |
| **Android** | iOS first; the decision point is device-mix data from the trial. |
| **Computer vision to flag idle workers** | Requested in the brief and **declined**. See `docs/WORKFORCE-POLICY.md`. The brief's position — that the surveillance objection is not our responsibility — does not hold under California employment law, where the builder is named too. Recorded, refused, and re-openable only on written employment-counsel advice. |
| **Break-length, idle-time, and indoor-location monitoring** | Same policy. We measure work finished, never bodily presence. |
| **Barcode scanning, warehouse map, route optimization** | Brief lists as future. Each depends on P3 data landing first. |
| **AI auto-ordering** (parse a customer text → match inventory → place order → assign truck → emit pick ticket) | Brief lists as future. Depends on P3, P5, and a product catalog that does not yet exist. |
| **Multi-seller marketplace** (GB + other distributors in one app) | Brief lists as far-future. |
| **Vendor portal, accounting, payroll** | Never in scope. |

**Generalization is in scope, quietly.** The brief wants the app usable by other distributors eventually. That is not a phase — it is a constraint applied from the first day: every record carries which company it belongs to, and nothing is hardcoded to GB. Retrofitting that later is one of the most expensive changes possible; applying it now costs about a day.

## Delivery Milestones

<!-- Business outcomes, not engineering tasks. /plan turns each into a plan. -->
<!-- Status: pending | in-progress | complete -->

| # | Milestone | Outcome | Status | Plan |
|---|---|---|---|---|
| 0 | Foundations & approach go/no-go | The project builds and runs; the technical approach is proven on a real screen, not argued about | in-progress | — |
| 1 | P1 — The interface | A manager can create work and an employee can complete it on a phone with no signal | pending | — |
| 2 | P2 — Logins and roles | Every person signs in as themselves; role and warehouse decide what they see; company data is provably isolated | pending | — |
| 3 | Trial — one warehouse | Three weeks of real daily use by a real crew, hitting the 80% completion bar | pending | — |
| 4 | Launch — four warehouses | Every GB warehouse on the app | pending | — |
| 5 | P3 — Dynamics data | Stock, products, and customers visible in the app, read-only, refreshed nightly | pending | — |
| 6 | P4 — Management dashboard | The CEO sees all four warehouses; a manager sees theirs | pending | — |
| 7 | P5 — Customer experience | A customer places an order without phoning it in | pending | — |
| 8 | P6 — Quality of life | Chat, notifications, and the accumulated smaller asks | pending | — |

**Two sequencing hazards worth stating plainly, since the phase order is the CEO's:**

1. **P1 before P2 means rework.** Screens built before identity exists get revisited when identity lands. The cost is real and bounded — roughly a week — and it is the price of having something to show early. Accepted knowingly, not overlooked.
2. **P3 sits in front of P4 but is the least controllable work in the plan.** Dynamics access depends entirely on GB producing an export, and the brief cannot yet say who administers that system. P3 is therefore planned as a **dependency-gated parallel track**: it starts the day GB's export file exists, and if that is late it runs *beside* P4 rather than in front of it. Nothing the CEO asked for first is allowed to wait on it.

## Open Questions

- [ ] **How does GB actually run purchasing and receiving?** The brief flags its own description as an assumption. Blocks every purchasing requirement. *Needs a process walkthrough on site.*
- [ ] **Who administers the Dynamics install** — internal IT, an outside consultant, or nobody? Blocks all of P3.
- [ ] **Can a product catalog be assembled at all**, and by whom? There is no catalog today. Blocks P5 entirely — customers cannot browse what has never been written down.
- [ ] **Personal phones or company phones?** California requires reimbursing employees for work use of a personal phone. Affects P1/P2 design and whether shared-device login is needed.
- [ ] **How does one shared shell serve three very different jobs without becoming three apps in a trenchcoat?** The brief raises this itself, and the cited competitor (OMELINK) solved it by *not* sharing — separate customer and employee experiences. Unresolved design question, not a settled decision.
- [ ] **What are the middle three tabs for an Employee?** The brief defines the customer and management uses of tabs 2–4 and leaves the employee column blank.
- [ ] **Which warehouse runs the trial, and which manager sponsors it?**
- [ ] **Does GB have employment counsel** who can review the workforce policy before any clock-in or monitoring feature?
- [ ] **Is selling this to other distributors a real intent or future-proofing?** Changes how much is invested in onboarding a second company.

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| The crew reads it as surveillance and rejects it | High | Critical | Workforce policy holds; employees see their own numbers; introduced as *clarity about what to do*, not monitoring; trial with a receptive crew |
| No product catalog exists, and building one is a bigger project than the app | High | High | Surfaced now, not at P5; catalog assembly is a GB deliverable with its own owner and date, or P5 does not start |
| Dynamics turns out to be unreachable | Medium | High | P1, P2 and the trial need **zero** Dynamics data; nightly plain-file export is the primary integration plan, not the fallback |
| The purchasing flow is materially different from what the brief assumes | High | Medium | No purchasing work is committed until a walkthrough happens; the assumption is labelled in every document |
| The CEO is the routing system | High | Medium | Named as a dependency, not designed around; route planning stays manual and out of scope until the core lands |
| Two part-time engineers, ~25 hrs/week combined | High | High | Scope moves, dates hold; winter break planned as zero output; nobody solo-owns any area |
| Pressure to pull AI, ordering, or monitoring forward | High | High | This document. The phase order is the commitment |
| Asking customers to install an app to place an order suppresses ordering | Medium | High | Flagged now; database keeps a web client cheap to add; decision revisited before P5 |

---
*Status: DRAFT — requirements only. Implementation planning pending via /plan.*
