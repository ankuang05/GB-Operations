# What We're Building

**Product:** OpsLink — an operations app for distribution warehouses
**First customer:** Goldberry Distributors (GB)
**Owner:** Andy Kuang
**Version:** 3.0 — 2026-09-14
**Status:** Draft for review

> **What changed in 3.0.** A fuller brief from the CEO arrived. Four things moved:
> 1. **The roadmap is now the CEO's six phases**, not our R1–R6 lettering. Dates were re-derived, not relabelled.
> 2. **Six roles collapse to three login buckets** — Management, Employee, Customer — with the specific job (picker, QC, driver, receiver, buyer, sales) carried as a tag on the person.
> 3. **Dynamics stays read-only for now.** Writing orders back into it is a candidate later stage, gated on proving Dynamics 2009 can accept input safely — not a commitment.
> 4. **How GB actually operates today is written down** for the first time, in [OPERATIONS-TODAY.md](OPERATIONS-TODAY.md). Read that first; this document assumes it.
>
> Version 2.0 narrowed scope from a website-plus-app to iOS only. That still holds — see [§9](#9-the-website-question).

---

## 1. The short version

GB is a 20-year-old food distribution business — meat, seafood, vegetables, dry goods — selling to meat markets, supermarkets, restaurants, and other distributors. Four warehouses, Bay Area and Sacramento. Orders come in by phone, text, WeChat and email. Work gets assigned by walking up to someone and telling them. The system of record is a Microsoft Dynamics installation old enough to vote.

The CEO's priority is **efficiency, not growth**. He wants to know whether work is actually getting done, and he wants the paper-and-phone-calls part of the day to stop eating hours.

**OpsLink is one iOS app that changes shape depending on who opens it.**

- A warehouse employee opens it and sees today's work. They check things off as they go.
- A manager opens it on an iPad and builds tomorrow's list, then watches today's get finished.
- The CEO opens it and sees all four warehouses at once.

Same app. Same download. The screens are different because the person is different.

**What we're committing to first:** Phases 1 and 2 — people can log in as themselves, managers can assign work, employees can complete it on a phone even with no signal, and managers can see what got done. One warehouse, supervised, as a trial. Everything else in this document comes after that.

---

## 2. The problems we're solving

The CEO listed what's going wrong. We've sorted it into what software actually fixes and what it doesn't. The full current-state workflow these come from is in **[OPERATIONS-TODAY.md](OPERATIONS-TODAY.md)**.

### 2.1 Things this app fixes

| # | The problem | What it costs today | What we do about it |
|---|---|---|---|
| **P1** | Drivers skip pre-trip checks — tire pressure, fuel | Breakdowns, DOT violations, late deliveries | A checklist on the phone with photos, that must be finished before the shift starts |
| **P2** | Pick tickets are typed out by hand | Hours of retyping, typos that become wrong deliveries | Tickets generated automatically and sent to the picker's phone |
| **P3** | A person is paid to double-check every pick | One full salary spent on verification | The picker scans each item; the check happens as the work happens |
| **P4** | Orders get re-keyed into the system by hand | Delays and transcription errors | Orders arrive in the system already typed |
| **P5** | Order-takers get several calls at once, across four channels | **Lost orders**, customers on hold, burned-out staff | Regular customers reorder themselves, which takes the routine calls off the phone |
| **P6** | Nobody can see whether the day's work is progressing | Managers and the CEO are guessing | Every assigned task is tracked — who, what, when, done or not |
| **P7** | Proof of delivery is a paper signature carried back in a truck | Nothing is knowable until the driver returns | Delivery confirmation recorded on the phone at the door |

### 2.2 Things this app deliberately does *not* fix

The CEO raised **30+ minute bathroom breaks**, **chatting on the clock**, and — in the newer brief — **computer vision that flags workers for slacking off**, with the note that the surveillance objection is not our responsibility to worry about.

These are real frustrations, and the request is on the record. Our answer is still no, and the reasoning is not squeamishness: in California, the party that builds a monitoring system gets named alongside the party that runs it. "The company can decide whether to turn it on" is not the shield it sounds like.

We address the actual worry — *is the work getting done?* — by measuring **work finished**, not time spent in a room.

This is important enough to have its own document: **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)**. The CEO should read it. So should GB's employment lawyer.

---

## 3. Who uses it

**Three login buckets.** The CEO's brief groups people this way, and we follow it. Within a bucket, a person carries **job tags** describing what they actually do.

| Bucket | Who | Device | What they do |
|---|---|---|---|
| **Management** | CEO, managers | iPad to plan, iPhone on the floor | Create and assign the day's work, see what got finished, handle problems, take customer orders, place vendor orders, manage accounts |
| **Employee** | Warehouse floor — the brief calls them **"task followers"** | iPhone | See today's tasks, complete them, clock in and out |
| **Customer** | Meat markets, supermarkets, restaurants, distributors | iPhone or iPad | Browse the catalog, place orders, see order history and bills |

**Job tags inside Employee:** `picker` · `stager` · `qc` · `driver` · `receiver` · `buyer`
**Job tags inside Management:** `sales` · `office`

**Scope, not a fourth role.** The CEO is not a separate login type — he is Management whose view spans **all four warehouses**, where a manager's spans **one**. That is a scope setting on the person, which means adding a regional manager later costs nothing.

**Why tags instead of roles.** A tag changes *which screens are useful to you*. A role changes *what the database will let you touch*. Driver and picker want different screens but need the same permissions, so they are tags. A tag gets promoted to a real role only when a phase genuinely needs different permissions — most likely `driver`, at the pre-trip checklist. Starting with three keeps the permission surface small enough to test exhaustively, which is the part that has to be right.

**The rule:** your role changes *what you see*, not *which app you install*. There is one app in the App Store. It builds itself around your permissions when you log in.

**Why iPad for managers.** A manager building tomorrow's list for 30 people is doing dense, table-shaped work — the kind of thing that's miserable on a 6-inch screen. The same app on an iPad gets a wider layout with side-by-side panels. It's not a separate app; it's the same app using the space it's given.

---

## 4. What ships when

The CEO's brief defines six phases. They are the spine. Dates come from the capacity arithmetic in **[TIMELINE.md](TIMELINE.md)**, not from ambition.

### Phase 1 — The interface
**Sep 28 – Dec 4, 2026**

The shell, and the task loop working end to end. Built against a single seeded company with a development role switch, because real logins are Phase 2.

| # | Feature | Plain English |
|---|---|---|
| 1.1 | The shell | The tab bar, navigation, and visual language the whole app is built in |
| 1.2 | Building the day's work (iPad) | One-off tasks and repeating ones, assigned to a person or a whole warehouse, with a time due and a priority |
| 1.3 | Doing the day's work (iPhone) | A "My Day" list. Tap to complete. Attach a photo or note as proof. **Works with no signal** and catches up when you're back online |
| 1.4 | Saved checklists | Save a routine daily list so tomorrow's plan takes five minutes, not thirty |
| 1.5 | The live board (iPad) | What's done, what's in progress, what's overdue, right now, for one warehouse |
| 1.6 | Spanish | The whole app in English and Spanish |

> **Tasks are a to-do list, deliberately.** The brief is explicit: tasks are manually created and **not tied to order or pick-ticket records**. That link is a later phase. Building it now would mean inventing the order model before Dynamics has told us what one looks like.

### Phase 2 — Logins and roles
**Dec 7, 2026 – Jan 22, 2027** *(winter break Dec 21 – Jan 1)*

| # | Feature | Plain English |
|---|---|---|
| 2.1 | Login | Email and password, password reset, staying logged in |
| 2.2 | Three buckets + job tags | Management / Employee / Customer, with tags, enforced by the database itself — not just by the app |
| 2.3 | Warehouse scope | CEO sees four warehouses; a manager sees one |
| 2.4 | Account admin | Invite people, give them a bucket, tags and a warehouse, turn them off when they leave |
| 2.5 | Company isolation | One company's data provably invisible to another, checked automatically on every code change |

**Then:** hardening and App Store submission, **Jan 25 – Feb 5** · **Trial in one warehouse, Feb 8 – 26** · **Live across all four warehouses, 2027-03-01**.

**Phases 1 and 2 together are the MVP.** They need **nothing** from Dynamics. If Dynamics access never happens, this still ships and still answers the CEO's first question.

**How we'll know it worked**

1. A manager builds a full next-day list for one warehouse in **under 5 minutes**.
2. An employee completes a task with **no internet**, and it shows up on the manager's screen within a minute of getting signal back.
3. One company's data is **provably invisible** to another — checked automatically on every code change.
4. The trial warehouse hits **80%+ of assigned tasks completed** for two weeks running.

### Phase 3 — Dynamics data · *dependency-gated, earliest 2027-03*
- A nightly, **read-only** copy of GB's Dynamics data: products, stock levels, customers
- Look up stock from the app
- The product catalog the ordering screens will need

> **This phase does not control its own start date. GB does.** It begins the day a nightly export file exists and not before. The brief puts it third; we plan it as a **parallel track** so that if GB's export is late, Phase 3 runs *beside* Phase 4 rather than in front of it. Nothing the CEO asked for first is allowed to wait on the least controllable work in the plan.

> **Read-only, for now.** The brief asks for orders to be inserted back into Dynamics. That is not a no forever — it is a no until a spike proves Dynamics 2009 accepts input safely and reversibly. Writing into a 20-year-old ERP that runs a live business is how the live business goes down. See [ENGINEERING.md §6.2](ENGINEERING.md#62-three-rules).

### Phase 4 — Management dashboard · *around 2027-Q2*
- Per-warehouse dashboards: completion trends, per-person history, overdue analysis — the CEO's "employee breakdown"
- Cross-warehouse comparison, all four at once
- Current inventory and purchase analysis *(needs Phase 3)*
- Daily reports

> Not to be confused with Phase 1.5. The **live board** — what's happening right now — ships in Phase 1, because the trial cannot prove anything without it. Phase 4 is the **analytical** layer on top: trends, history, and comparison across warehouses.

### Phase 5 — Customer experience · *around 2027-Q4 / 2028-Q1*
- Customers browse, add to cart, submit orders *(fixes P4 and P5)*
- Order tracking, order history, billing view
- Management placing orders on a customer's behalf

> 🚩 **Two blockers, both bigger than the software.**
> **There is no product catalog** — see [OPERATIONS-TODAY.md §6.1](OPERATIONS-TODAY.md#61-there-is-no-product-catalog). Customers cannot browse what has never been written down. Someone at GB has to write it down, and that is a project with its own owner and date.
> **With no website, a customer must install an app to order.** For a restaurant owner reordering produce, that is a far bigger ask than clicking a link. This is the phase where the no-website decision costs the most — see [§9](#9-the-website-question).

### Phase 6 — Quality of life · *2028*
- Chat between customers and GB *(the brief's first tab)*
- Notifications for assignment and overdue work
- Driver pre-trip vehicle checklist, hardened past a generic task
- Clock in and out, using a rough boundary around the property line — **gated on employment-counsel review**
- AI help: drafting task lists from what usually happens, flagging **operational** oddities — never scoring people. See [WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)
- Android

### Out of scope, on the record
Barcode scanning · warehouse map · route optimization · AI auto-ordering (parse a text order → match inventory → place it → assign a truck → emit a pick ticket) · multi-seller marketplace · vendor portal · **computer-vision monitoring of workers** ([declined](WORKFORCE-POLICY.md))

---

## 5. The five-tab idea

The brief describes a five-tab bottom bar shared across all roles, modelled on OMELINK. We're keeping it as the destination, but filling tabs in as they become real rather than shipping five tabs where three do nothing.

| Tab | Management | Employee | Customer | Arrives |
|---|---|---|---|---|
| **1. Chat** | Message customers | *undefined* | Message GB | P6 |
| **2. Role-specific** | *undefined* | *undefined* | **Ordering** — browse catalog, place order | P5 |
| **3. "+" Create** | **Create tasks; place purchase and sales orders** | *undefined* | *undefined* | **P1** (tasks) · P3+ (orders) |
| **4. Role-specific** | **Dashboards, inventory, purchase analysis, daily reports — split by warehouse** | *undefined* | Purchase history and metrics *(or in Profile — undecided)* | P1 (live board) · P4 |
| **5. Profile** | Account | Account | Account, orders, billing | P2 |

**The gap in that table is the real finding.** Four of the fifteen cells are defined. The Employee column — the users we ship to *first* — is blank in the brief for four of five tabs.

**Our recommendation stands:** in Phase 1, a warehouse employee sees a **two-tab app** — My Day and Profile. Three dead tabs on first launch teach the crew the app is half-finished and not worth opening, and getting people to actually use it is the single biggest risk in this project. The five-tab target hasn't changed. We're just not showing empty rooms.

**The open design question, in the brief's own words:** how does one shared interface flex to fit roles with very different jobs without becoming several different apps wearing the same shell? Worth noting that **OMELINK answered it by not sharing** — separate customer and employee experiences. We're still betting on one adaptive app, but it is a bet, and the nearest comparable product bet the other way. See [OPERATIONS-TODAY.md §6.2](OPERATIONS-TODAY.md#62-the-competitor-solved-the-hard-design-problem-by-avoiding-it).

---

## 6. Requirements that aren't features

Things the app must do that don't show up as a button.

| # | Requirement | Why |
|---|---|---|
| **N1** | The task list works completely offline; anything you do queues up and syncs when signal returns | Warehouse interiors and cold storage have terrible Wi-Fi |
| **N2** | Buttons at least 44×44 points; the main actions reachable with one thumb | People are wearing gloves in cold storage |
| **N3** | Full English and Spanish from day one | Most of the workforce in California food distribution is bilingual |
| **N4** | One company's data is walled off **in the database**, not just in the app | An app bug must never be able to leak another customer's data |
| **N5** | The manager dashboard loads in under 2 seconds on cell data | Managers check it standing on the floor |
| **N6** | Personal data encrypted, stored and in transit | California privacy law (CPRA) covers employee data too |
| **N7** | A complete record of who changed what and when, kept 24 months | Any accountability system eventually gets quoted in an employment dispute |
| **N8** | Handles GB's 4 warehouses and ~150 people, and 50 other companies without a rebuild | The brief wants this usable by other distributors eventually — see below |
| **N9** | iOS 17 and up, iPhone and iPad | Covers the realistic device base without paying a backwards-compatibility tax |

**On N8 — generalization is a constraint, not a phase.** The brief lists "make the app relatively generalized, not hardcoded to GB" as a far-future goal. Treating it as future work is the expensive way to do it: retrofitting multi-company support touches every query, every rule and every existing row. Applying it from the first migration costs about a day. So it is in from day one and never appears on the roadmap.

---

## 7. Assumptions we're working from

If any of these turn out to be wrong, tell us early — several dates move.

- **A1** — The team is two people, part-time, about **25 hours a week between them**. *Every date in these documents comes from this number.*
- **A2** — GB gives us one warehouse and a willing manager for the trial.
- **A3** — The engineers are comfortable enough in Swift to be productive. **Tested in the first two weeks before committing.**
- **A4** — Warehouse employees have iPhones on iOS 17+, or GB provides shared ones. *See Q1.*
- **A5** — GB pays for an Apple Developer account ($99/year) and hosting (about $25–50/month at trial size).
- **A6** — **The purchasing and receiving flow in [OPERATIONS-TODAY.md §3](OPERATIONS-TODAY.md#3-the-purchase-order-and-receiving) is a guess.** The brief says so itself. No purchasing requirement is committed until someone walks it.

### Open questions

| # | Question | Holds up | Needed by |
|---|---|---|---|
| **Q1** | Personal phones or company phones? Affects device management, expense reimbursement (California requires reimbursing work use of a personal phone), and whether shared-device login is needed. | P1 design | Oct 2026 |
| **Q2** | Who looks after the Dynamics system — internal IT, an outside consultant, or nobody? | All of P3 | Nov 2026 |
| **Q3** | Which warehouse runs the trial, and which manager is sponsoring it? | The trial | Oct 2026 |
| **Q4** | ~~Is there a product catalog file anywhere?~~ **Answered: there is none.** New question — **who at GB can build one, and by when?** | All of P5 | Mid 2027 |
| **Q5** | Does GB have an employment lawyer who can review [WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)? | P6 clock-in | Jan 2027 |
| **Q6** | Is selling this to other distributors a real intent, or future-proofing? Changes how much we invest in onboarding a second company. | Roadmap past P2 | Nov 2026 |
| **Q7** | At Phase 5, are we comfortable asking GB's customers to install an app to place orders? | P5 | Mid 2027 |
| **Q8** | **New:** how does GB actually run purchasing and receiving? The brief flags its own description as unverified. | Every purchasing feature | Before any P3 purchasing work |
| **Q9** | **New:** what do tabs 1, 2 and 4 do for an **Employee**? The brief leaves the column blank. | P6 tab build-out | Early 2027 |
| **Q10** | **New:** the CEO personally plans truck routes. Is that a constraint we design around, or something he wants to stop doing? | Any routing feature | 2027 |

---

## 8. What we need from GB

| # | What | By when | What breaks if it's late |
|---|---|---|---|
| 1 | Written sign-off on this plan and on the workforce policy | **2026-09-25** | Everything |
| 2 | Apple Developer account paid ($99/yr) | **2026-09-18** | Testing on real phones, App Store |
| 3 | Hosting budget approved (~$25–50/mo) | **2026-09-25** | The test environment |
| 4 | Named trial warehouse and manager sponsor | **2026-10-09** | The trial |
| 5 | Decision on personal vs. company phones (Q1) | **2026-10-16** | The phone app design |
| 6 | Answers to the Dynamics questionnaire | **2026-11-06** | All of Phase 3 |
| 7 | Two hours of a real manager's time to time-test task building | **2026-11-06** | Proving the "under 5 minutes" claim |
| 8 | **New:** a walkthrough of the real purchasing and receiving process | **2026-12-04** | Every purchasing feature (Q8) |
| 9 | Employment lawyer review of the workforce policy | **2027-01-15** | P6 clock-in |
| 10 | Two iPads for the trial warehouse | **2027-02-01** | The trial |
| 11 | **New:** a named owner and date for building the product catalog | **2027-03-01** | All of Phase 5 |

---

## 9. The website question

**We are not building a website. We are also not making one impossible.**

The owner's direction is to focus on the app. That's the plan. But "no website now" and "never a website" are different decisions, and we're only making the first one. The brief itself lists a website as a far-future goal — "similar to the Instagram app and instagram.com" — which is exactly the shape this design keeps cheap.

**How we keep the door open.** All the real rules of the system — who can see what, how a repeating task becomes tomorrow's task, what counts as complete — live in the **database**, not in the iOS app. The app is a face on top of them. A website added later would talk to the same database and inherit every rule for free, instead of re-implementing them and drifting out of sync.

This costs us essentially nothing now. It's how we'd build it anyway.

**What adding a website later would take:**

| Scope | Rough effort at 25 hrs/week |
|---|---|
| Read-only dashboard so the CEO can check numbers in a browser | **6–8 weeks** |
| Full manager parity — build tasks, admin, reporting, all in a browser | **12–14 weeks** |
| Customer ordering on the web (the Phase 5 problem above) | **8–10 weeks** |

**When we'd revisit it.** Three moments would make a website the obvious next move:

1. **Phase 5 approaches** and asking customers to install an app looks like it'll block orders.
2. **The CEO stops opening the iPad app.** If checking the numbers requires picking up a specific device, it may just not happen — and then the visibility problem isn't solved.
3. **A second customer appears** who wants to try it without an App Store install.

None of those are decisions for today. All three are worth watching.

---

## 10. How we'll know it worked

**The MVP succeeds if, at the end of the trial:**

| What we measure | Where it is today | Where it needs to be |
|---|---|---|
| Assigned daily tasks completed and recorded in the app | 0% — there's no system | **80% or better** |
| Time for a manager to build tomorrow's list | ~30 min, verbally and on paper | **Under 5 minutes** |
| Warehouse employees logging in on a workday | 0% | **90% or better** |
| Manager says they have better visibility than before | — | **4 out of 5 or better** |
| Serious bugs still open at the end of the trial | — | **Zero** |

**The number we watch weekly:** how many people open the app each day. If that stalls below 70% in week 2, **we stop adding features and fix adoption instead.** An accountability system nobody uses is worse than no system, because it produces confidence in numbers that aren't real.

---

## 11. What could go wrong

| Risk | How likely | How bad | What we do about it |
|---|---|---|---|
| **The crew sees it as surveillance and rejects it** | High | Critical | The workforce policy; employees see their own numbers; the manager introduces it as *clarity about what to do*, not monitoring; trial with a receptive crew first |
| **No product catalog exists, and building one is its own project** | High | High | Surfaced now rather than at Phase 5; catalog assembly is a GB deliverable with an owner and a date, or Phase 5 doesn't start |
| **Phase 1 before Phase 2 means rework** | Certain | Medium | Known and budgeted — about a week of revisiting screens when real identity lands. The price of having something to show early, accepted knowingly |
| **The engineers aren't fast enough in Swift** | Medium | High | A two-week test in Phase 0 — build a real screen against the real database. If it's painful, we switch approach before any real code exists |
| **Team capacity slips** — exams, jobs, life | High | High | A plan with a built-in winter break; **scope gets cut, dates don't move**; the MVP is already the minimum version |
| **Dynamics turns out to be unreachable** | Medium | High | Phases 1 and 2 need **zero** Dynamics data; a plain-CSV-file fallback is designed in from the start; Phase 3 is a parallel track, not a blocker |
| **The purchasing flow differs from what the brief assumes** | High | Medium | Nothing purchasing-related is committed until the walkthrough (Q8); the assumption is labelled everywhere it appears |
| **App Store rejects the app** | Medium | Medium | Enroll in week one; test builds go out from December, not the week before launch |
| **Pressure to add ordering, AI, or monitoring early** | High | High | This document. The phase order *is* the commitment |
| **Two people, no backup** | Medium | High | Nobody solo-owns any part; both engineers touch everything; every decision written down in `docs/` |
| **No website becomes a problem at Phase 5** | Medium | High | Flagged now, database designed to make a website cheap to add, decision revisited mid-2027 |

---

## Related documents

- **[OPERATIONS-TODAY.md](OPERATIONS-TODAY.md)** — how GB actually runs today, and which parts of that we've verified
- **[FOR-THE-CEO.md](FOR-THE-CEO.md)** — the short, plain-English version of everything here
- **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)** — what we measure about employees, and what we refuse to
- **[ENGINEERING.md](ENGINEERING.md)** — how it's actually built
- **[TIMELINE.md](TIMELINE.md)** — dates, phase by phase
