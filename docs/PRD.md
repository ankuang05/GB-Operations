# Product Requirements Document

**Product:** Workforce & Operations Platform — working title **OpsLink**
**First customer:** Goldberry Distributors (GB)
**Document owner:** Andy Kuang
**Version:** 1.0 — 2026-09-09
**Status:** Draft for stakeholder review

---

## 1. Executive summary

GB is a 20-year-old food distribution business running four Bay Area and Sacramento warehouses on word-of-mouth demand and a legacy Microsoft Dynamics ERP. The CEO's stated priority is **efficiency, not expansion**: he wants visibility into whether work is actually getting done, and he wants the paper-and-phone-call parts of the operation to stop costing hours a day.

OpsLink is a role-based operations platform delivered as **a web application and an iOS app sharing one backend**. The web app is the full-capability surface — task authoring, dashboards, reporting, administration. The iOS app is the floor surface — a warehouse employee opens it in the morning, sees what they owe today, and closes it out as they go.

We are building it as a **multi-tenant product**, not a GB-specific tool, per the CEO's explicit instruction. GB is tenant #1.

**What we commit to first:** authenticated role-based access, daily task management, and manager visibility into completion — shipped to one warehouse as a supervised pilot. Everything else in this document is sequenced behind that.

---

## 2. Problem statement

The CEO identified the failures below. We have grouped them by whether software actually fixes them.

### 2.1 Problems software solves directly

| # | Problem | Cost today | How OpsLink addresses it |
|---|---|---|---|
| P1 | Drivers skip routine pre-trip checks (tire pressure, fuel) | Breakdown risk, DOT exposure, delayed deliveries | Mandatory digital pre-trip checklist with photo capture, gating shift start |
| P2 | Pick tickets are typed manually | Hours of duplicate data entry, transcription errors | Pick tickets generated from order data, dispatched to the picker's device |
| P3 | A dedicated QC person sits between pick and load | One salary spent on verification | Scan-verified picking makes the pick self-verifying; QC becomes exception handling |
| P4 | Orders are manually re-keyed into the system | Latency, transcription errors | Customer-placed orders land in the system directly |
| P5 | Phone order-takers get multiple simultaneous calls | Lost orders, held customers, burnout | Self-service ordering diverts routine reorders off the phone |
| P6 | No visibility into whether daily work is progressing | CEO and managers are flying blind | Task assignment + completion telemetry per person, per warehouse, per day |

### 2.2 Problems software should *not* try to solve directly

The CEO also raised **30+ minute bathroom breaks** and **chatting on the clock**. These are real management frustrations. They are also the two items where the obvious software response — monitor the person — is the wrong answer. We address the underlying concern (is the work getting done?) by measuring **work output**, not bodily presence. See **§7**, formalized as ADR-004 in the Engineering Design Document.

---

## 3. Users and roles

Six roles. Every user belongs to exactly one organization and holds one primary role.

| Role | Primary surface | What they do |
|---|---|---|
| **CEO** | Web | Cross-warehouse dashboards, trend reporting, org administration |
| **Manager** | Web (authoring) + iOS (floor) | Create and assign daily tasks, review completion, handle exceptions |
| **Salesperson** | Web + iOS | Take and enter orders, manage customer accounts, check inventory |
| **Warehouse employee** | iOS | See today's tasks, complete them, clock in/out |
| **Driver** | iOS | Pre-trip vehicle checks, delivery manifest, proof of delivery |
| **Customer** | Web + iOS | Browse catalog, place orders, view order history and billing |

**Design principle:** roles change *what you see*, not *which app you install*. There is one iOS app and one web app; the interface composes itself from the permissions attached to your role.

---

## 4. Product scope by release

### Release 1 — MVP: Accountability Core
**Pilot 2027-02-15 · GA 2027-03-08**

The narrowest release GB will use daily, and the one that answers the CEO's first question.

**In scope**
- **R1.1 Authentication** — email/password login, password reset, session management, forced logout
- **R1.2 Role-based access control** — six roles, permissions enforced at the database layer
- **R1.3 Organization & user administration** — invite users, assign roles and warehouse, deactivate
- **R1.4 Task authoring (web)** — one-off and recurring tasks, assigned to a person or warehouse group, with due time and priority
- **R1.5 Task execution (iOS)** — "My Day" list, mark complete, attach photo or note as proof, works offline and syncs on reconnect
- **R1.6 Task templates** — save a recurring daily checklist so a manager builds tomorrow's list in under five minutes
- **R1.7 Manager visibility (web)** — live completion board per warehouse, per-person history, overdue queue
- **R1.8 Spanish localization** — full UI in English and Spanish

**Explicitly out of scope for R1:** chat, ordering, inventory, Dynamics data, AI features, barcode scanning, Android.

**R1 acceptance criteria**
1. A manager builds a full next-day task list for one warehouse in under 5 minutes.
2. A warehouse employee completes a task with no network connection, and it appears on the manager dashboard within 60 seconds of reconnecting.
3. A user in one organization cannot read or write any data belonging to another organization, verified by automated test.
4. The pilot warehouse records ≥80% daily task completion for two consecutive weeks.

### Release 2 — Field Operations · Target 2027-05
- Driver pre-trip vehicle checklist with photo evidence and blocking logic
- Clock in/out with coarse warehouse geofence (perimeter only — see §7)
- Push notifications for assignment and overdue tasks
- Android build

### Release 3 — Data Bridge · Target 2027-08 *(dependency-gated, see §8)*
- Read-only nightly ingestion of GB's Dynamics data (products, inventory, customers, pick tickets)
- Inventory lookup on web and iOS
- Product catalog backing the ordering UI

### Release 4 — Digital Pick Tickets · Target 2027-11
- Pick tickets generated and dispatched digitally (**solves P2**)
- Barcode scanning via phone camera for scan-verified picking (**solves P3**)
- Pick accuracy and cycle-time metrics

### Release 5 — Customer Ordering · Target 2028-Q1
- Customer catalog browsing, cart, order submission (**solves P4, P5**)
- Order status tracking, order history, billing view
- Salesperson order entry on behalf of a customer

### Release 6 — Intelligence & Communication · Target 2028-Q2
- In-app messaging (the CEO's "first page" chat concept)
- CEO analytics: purchase analysis, daily reports, cross-warehouse benchmarks
- AI assistance: draft task lists from historical patterns, flag *operational* anomalies (boundary defined in §7)

### Backlog — not scheduled
Interactive warehouse map · route optimization · demand forecasting · vendor portal

---

## 5. The five-page mobile structure

The CEO described a five-tab iOS app modeled on OMELINK. We keep that as the long-term target and fill it in progressively rather than shipping five empty tabs.

| Tab | CEO's description | R1 state | Filled by |
|---|---|---|---|
| 1. Chat | iMessage-style, add by phone number | **Hidden** | R6 |
| 2. Orders | Customer product ordering | **Hidden** | R5 |
| 3. "+" Create | Create tasks | **Ships in R1** (managers) | R1 |
| 4. Tools | Inventory, purchase analysis, daily reports | **Partial in R1** — task stats only | R3, R6 |
| 5. Profile | Account, orders, billing, contacts | **Partial in R1** — account and settings | R5 |

**Recommendation to the CEO:** in R1 the warehouse employee sees a two-tab app — My Day and Profile. Shipping a five-tab shell with three dead tabs teaches the crew the app is unfinished, and adoption is the single biggest risk to this project. Tabs appear as they become real.

---

## 6. Non-functional requirements

| ID | Requirement | Rationale |
|---|---|---|
| NFR-1 | iOS task list fully functional offline; writes queue and sync on reconnect | Warehouse interiors and cold storage have unreliable Wi-Fi |
| NFR-2 | Minimum 44×44pt tap targets; primary actions reachable one-handed | Users wear gloves in cold storage |
| NFR-3 | Full English/Spanish localization from R1 | Predominantly bilingual warehouse workforce in CA food distribution |
| NFR-4 | Tenant isolation enforced at the database row level, not only in application code | A bug in app code must not leak data across customers |
| NFR-5 | Web dashboard loads in <2s on 4G | Managers check it on the floor |
| NFR-6 | Personal data encrypted at rest and in transit | CPRA obligations extend to employee data |
| NFR-7 | Complete audit log of who changed what and when, retained 24 months | Any accountability system will eventually be cited in an employment dispute |
| NFR-8 | Supports GB's 4 warehouses / ~150 users, and 50 tenants without re-architecture | Product, not a one-off |
| NFR-9 | iOS 17+ | Covers the practical device base; avoids back-compat tax |

---

## 7. Workforce measurement policy

*This is a product requirement, not commentary. It constrains what we build.*

**The concern, stated plainly.** The CEO asked the app to address employees spending 30+ minutes in the bathroom, chatting during clocked-in hours, and for "AI to keep employees on track." Software that measures bathroom duration, monitors conversations, or algorithmically scores individuals would expose GB to meaningful legal risk in California:

- **Rest and meal breaks** must be duty-free and free of employer control under the IWC Wage Orders and Labor Code §226.7. Systems that time or flag bathroom usage invite claims that breaks were not genuinely relinquished.
- **Disability and pregnancy accommodation.** Restroom frequency is a protected medical characteristic under FEHA and the ADA. Flagging it — even neutrally — creates discrimination exposure.
- **Communications.** California is a two-party-consent state (Penal Code §632, CIPA). Automated scanning of in-app messages for productivity signals is legally hazardous.
- **Automated decision-making.** California's ADMT regulations under the CPRA impose notice, opt-out, and risk-assessment obligations on automated technology used in employment decisions, with compliance obligations landing in the same window as our R6. Building an individual scoring engine now creates a compliance project later.

We are not lawyers, and **GB should have employment counsel review any monitoring feature before it ships.** But the design response needs no legal advice, because it is also the better product:

**We measure work output, not bodies.**

| We build | We do not build |
|---|---|
| Task assignment and completion rates | Bathroom or break duration tracking |
| On-time completion %, per person and per team | Idle-time or "inactivity" detection |
| Pre-trip check compliance | Continuous or indoor location tracking |
| Pick accuracy and order cycle time | Chat content analysis or keyword scanning |
| Clock in/out against a **coarse site perimeter** | Movement tracking inside the warehouse |
| AI that drafts task lists and flags **operational** anomalies ("Warehouse 2's pick accuracy fell 12% this week") | AI that scores or ranks individual employees |

**Hard constraints on every metric we ship:**
- **RM-1** Individual-level metrics are advisory. The system never auto-generates discipline, warnings, or rankings.
- **RM-2** Every metric shown to a manager links to its evidence — which tasks, when, completed or not.
- **RM-3** Employees see their own metrics exactly as their manager sees them. No secret scores.
- **RM-4** A written Notice at Collection appears at first login, describing what is collected and why.
- **RM-5** No feature infers a reason for absence from a workstation.

This gets the CEO what he actually wants. "Is this person doing their work?" is answered better by *14 of 16 assigned tasks completed, 2 overdue* than by *44 minutes in the bathroom* — and the first is a conversation a supervisor can have without a lawyer in the room.

---

## 8. Dependencies, assumptions, and open questions

### Assumptions
- **A1** Team capacity is ~25 engineer-hours/week combined (two people, ~10–15 hrs each). **Every date in this document derives from this number.**
- **A2** GB provides one warehouse and a willing manager for the R1 pilot.
- **A3** Warehouse employees have iPhones on iOS 17+, or GB provides shared devices. **See Q1.**
- **A4** GB funds an Apple Developer Program account ($99/yr) and hosting (~$50–100/mo at pilot scale).

### Dependencies
- **D1** *(blocks R3)* Read access to GB's Dynamics database, or a nightly export GB commits to producing.
- **D2** *(blocks R3)* Identification of the Dynamics version and schema. Discovery checklist: Engineering Design Document, Appendix A.
- **D3** *(blocks R5)* A cleaned product catalog with pricing. Does not exist in usable form today.
- **D4** *(blocks R1 pilot)* Apple Developer Program enrollment — start in Sprint 0; it can take two weeks.

### Open questions for the CEO

| # | Question | Blocks | Needed by |
|---|---|---|---|
| Q1 | Personal phones (BYOD) or company devices? Affects device management, expense reimbursement under Labor Code §2802, and whether we need shared-device login. | R1.5 design | Sprint 1 |
| Q2 | Who owns the Dynamics system — internal IT, an outside consultant, or nobody? | R3 entirely | Sprint 2 |
| Q3 | Which warehouse runs the pilot, and who is the manager sponsor? | R1 pilot | Sprint 2 |
| Q4 | Is there an existing product catalog file, even a messy spreadsheet? | R5 | Sprint 4 |
| Q5 | Does GB have employment counsel to review §7? | R2 clock-in | Sprint 4 |
| Q6 | Is the intent to sell OpsLink to other distributors, or is multi-tenancy future-proofing? Changes investment in tenant onboarding. | Roadmap past R2 | Sprint 3 |

---

## 9. Success metrics

**R1 succeeds if, at the end of the pilot:**

| Metric | Baseline | Target |
|---|---|---|
| Daily assigned tasks completed and recorded in-app | 0% (no system) | ≥80% |
| Time for a manager to build a next-day task list | ~30 min (verbal/paper) | <5 min |
| Warehouse employees logging in on a given workday | 0% | ≥90% |
| Manager reports better visibility than before (survey) | — | ≥4/5 |
| P1 bugs open at pilot exit | — | 0 |

**Leading indicator, watched weekly:** daily active users. If crew adoption stalls below 70% in week 2, we stop adding features and fix adoption. An unused accountability system is worse than none — it produces false confidence.

---

## 10. Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| **Crew rejects the app as surveillance** | High | Critical | §7 policy; employees see their own data; the manager sponsor introduces it as task clarity, not monitoring; pilot with a receptive crew |
| Team capacity slips (exams, jobs, life) | High | High | 25-week plan with a built-in winter break; **scope is cut, never dates**; R1 is already the minimum cut |
| Dynamics data proves inaccessible | Medium | High | R1 and R2 require **zero** Dynamics data; CSV-drop fallback designed in from the start (ADR-003) |
| App Store review rejection | Medium | Medium | Enroll in Sprint 0; TestFlight from Sprint 3, not at the end |
| Scope pressure to add ordering/AI early | High | High | This document; the release sequence *is* the commitment |
| Two-person bus factor | Medium | High | No solo ownership of any subsystem; both engineers touch every layer; decisions recorded in `docs/` |

---

## 11. Related documents

- **Engineering Design Document** — [`docs/ENGINEERING-DESIGN.md`](ENGINEERING-DESIGN.md) — architecture, data model, integration strategy, ADRs
- **Roadmap & Delivery Plan** — [`docs/ROADMAP.md`](ROADMAP.md) — sprint-by-sprint schedule with dates
- **Build Orchestration** — [`docs/ORCHESTRATION.md`](ORCHESTRATION.md) — per-step agent execution chains
