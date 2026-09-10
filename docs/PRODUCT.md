# What We're Building

**Product:** OpsLink — an operations app for distribution warehouses
**First customer:** Goldberry Distributors (GB)
**Owner:** Andy Kuang
**Version:** 2.0 — 2026-09-09
**Status:** Draft for review

> **Version 2.0 changed one big thing.** Version 1.0 described two products: a website for managers and a phone app for the warehouse floor. The owner has asked us to build **the iOS app only**. Everything in this document assumes one app and no website. See [§9](#9-the-website-question) for how we keep the website possible later.

---

## 1. The short version

GB is a 20-year-old food distribution business. Four warehouses, Bay Area and Sacramento. Orders come in by phone. Work gets assigned by walking up to someone and telling them. The system of record is a Microsoft Dynamics installation old enough to vote.

The CEO's priority is **efficiency, not growth**. He wants to know whether work is actually getting done, and he wants the paper-and-phone-calls part of the day to stop eating hours.

**OpsLink is one iOS app that changes shape depending on who opens it.**

- A warehouse employee opens it and sees today's work. They check things off as they go.
- A manager opens it on an iPad and builds tomorrow's list, then watches today's get finished.
- The CEO opens it and sees all four warehouses at once.

Same app. Same download. The screens are different because the person is different.

**What we're committing to first:** people can log in, managers can assign work, employees can complete it on a phone even with no signal, and managers can see what got done. One warehouse, supervised, as a trial. Everything else in this document comes after that.

---

## 2. The problems we're solving

The CEO listed what's going wrong. We've sorted it into what software actually fixes and what it doesn't.

### 2.1 Things this app fixes

| # | The problem | What it costs today | What we do about it |
|---|---|---|---|
| **P1** | Drivers skip pre-trip checks — tire pressure, fuel | Breakdowns, DOT violations, late deliveries | A checklist on the phone with photos, that must be finished before the shift starts |
| **P2** | Pick tickets are typed out by hand | Hours of retyping, typos that become wrong deliveries | Tickets generated automatically and sent to the picker's phone |
| **P3** | A person is paid to double-check every pick | One full salary spent on verification | The picker scans each item; the check happens as the work happens |
| **P4** | Orders get re-keyed into the system by hand | Delays and transcription errors | Orders arrive in the system already typed |
| **P5** | Order-takers get several phone calls at once | Lost orders, customers on hold, burned-out staff | Regular customers reorder themselves, which takes the routine calls off the phone |
| **P6** | Nobody can see whether the day's work is progressing | Managers and the CEO are guessing | Every assigned task is tracked — who, what, when, done or not |

### 2.2 Things this app deliberately does *not* fix

The CEO also raised **30+ minute bathroom breaks** and **chatting on the clock**.

These are real frustrations. They're also the two where the obvious software answer — watch the person — is the wrong answer, legally and practically. We address the actual worry ("is the work getting done?") by measuring **work finished**, not time spent in a room.

This is important enough to have its own document: **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)**. The CEO should read it. So should GB's employment lawyer.

---

## 3. Who uses it

Six kinds of people. Everyone belongs to one company and has one job title in the app.

| Role | Device | What they do |
|---|---|---|
| **CEO** | iPad | Looks across all four warehouses, reads trends, manages accounts |
| **Manager** | iPad to plan, iPhone on the floor | Creates and assigns the day's work, checks what got finished, handles problems |
| **Salesperson** | iPhone or iPad | Takes orders, manages customer accounts, checks stock |
| **Warehouse employee** | iPhone | Sees today's tasks, completes them, clocks in and out |
| **Driver** | iPhone | Pre-trip vehicle check, delivery list, proof of delivery |
| **Customer** | iPhone or iPad | Browses the catalog, places orders, sees order history and bills |

**The rule:** your role changes *what you see*, not *which app you install*. There is one app in the App Store. It builds itself around your permissions when you log in.

**Why iPad for managers.** A manager building tomorrow's list for 30 people is doing dense, table-shaped work — the kind of thing that's miserable on a 6-inch screen. The same app on an iPad gets a wider layout with side-by-side panels. It's not a separate app; it's the same app using the space it's given.

---

## 4. What ships when

### Release 1 — The Accountability Core
**Trial starts 2027-02-01 · Everyone on it 2027-02-22**

The smallest version GB would actually use every day, and the one that answers the CEO's first question.

**What's in it**

| # | Feature | Plain English |
|---|---|---|
| R1.1 | Login | Email and password, password reset, staying logged in |
| R1.2 | Roles and permissions | Six roles, enforced by the database itself — not just by the app |
| R1.3 | Account admin | Invite people, give them a role and a warehouse, turn them off when they leave |
| R1.4 | Building the day's work (iPad) | One-off tasks and repeating ones, assigned to a person or a whole warehouse, with a time due and a priority |
| R1.5 | Doing the day's work (iPhone) | A "My Day" list. Tap to complete. Attach a photo or note as proof. **Works with no signal** and catches up when you're back online |
| R1.6 | Saved checklists | Save a routine daily list so tomorrow's plan takes five minutes, not thirty |
| R1.7 | Seeing what got done (iPad) | A live board per warehouse, each person's history, and a list of what's overdue |
| R1.8 | Spanish | The whole app in English and Spanish |

**Not in Release 1:** chat, ordering, inventory, anything from Dynamics, AI features, barcode scanning, Android, a website.

**How we'll know Release 1 worked**

1. A manager builds a full next-day list for one warehouse in **under 5 minutes**.
2. An employee completes a task with **no internet**, and it shows up on the manager's screen within a minute of getting signal back.
3. One company's data is **provably invisible** to another company — checked automatically on every code change.
4. The trial warehouse hits **80%+ of assigned tasks completed** for two weeks running.

### Release 2 — Out in the Field · around 2027-04
- Driver pre-trip vehicle checklist, with photos, that blocks the shift until it's done
- Clock in and out, using a rough boundary around the warehouse (the property line — not tracking inside the building)
- Notifications when you're assigned something or something's overdue
- Android version

### Release 3 — Connecting to Dynamics · around 2027-07 *(depends on GB — see [§8](#8-what-we-need-from-gb))*
- A nightly, **read-only** copy of GB's Dynamics data: products, stock levels, customers, pick tickets
- Look up stock from the app
- The product catalog the ordering screens will need

We never write anything back into Dynamics. It stays the official record. We read from it.

### Release 4 — Digital Pick Tickets · around 2027-10
- Pick tickets created and sent to phones automatically *(fixes P2)*
- Barcode scanning with the phone camera, so picks check themselves *(fixes P3)*
- Numbers on pick accuracy and how long picks take

### Release 5 — Customer Ordering · around 2028-Q1
- Customers browse, add to cart, submit orders *(fixes P4 and P5)*
- Order tracking, order history, billing view
- Salespeople placing orders on a customer's behalf

> ⚠️ **Read this one twice.** With no website, a customer has to download an app from the App Store to place an order. For a restaurant owner reordering produce, that's a much bigger ask than clicking a link. This is the release where the no-website decision costs the most. See [§9](#9-the-website-question).

### Release 6 — Reports and Messaging · around 2028-Q2
- Messaging inside the app (the CEO's "first page" chat idea)
- CEO reporting: purchasing analysis, daily summaries, warehouse-vs-warehouse comparison
- AI help: drafting task lists from what usually happens, flagging **operational** oddities — never scoring people. See [WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)

### Someday, not scheduled
Interactive warehouse map · delivery route optimization · demand forecasting · vendor portal

---

## 5. The five-tab idea

The CEO described a five-tab app modeled on OMELINK. We're keeping that as the destination, but filling the tabs in as they become real rather than shipping five tabs where three do nothing.

| Tab | What the CEO described | Release 1 | Arrives in |
|---|---|---|---|
| 1. Chat | iMessage-style, add people by phone number | **Hidden** | R6 |
| 2. Orders | Customers ordering products | **Hidden** | R5 |
| 3. "+" Create | Making tasks | **Ships in R1** (managers only) | R1 |
| 4. Tools | Stock, purchasing analysis, daily reports | **Partly** — task stats only | R3, R6 |
| 5. Profile | Account, orders, billing, contacts | **Partly** — account and settings | R5 |

**Our recommendation:** in Release 1, a warehouse employee should see a **two-tab app** — My Day and Profile.

Here's why that matters. Three tabs that do nothing teach the crew that the app is half-finished and not worth opening. And getting people to actually use it is the single biggest risk in this whole project. The five-tab target hasn't changed. We're just not showing empty rooms.

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
| **N8** | Handles GB's 4 warehouses and ~150 people, and 50 other companies without a rebuild | We're building a product, not a one-off |
| **N9** | iOS 17 and up, iPhone and iPad | Covers the realistic device base without paying a backwards-compatibility tax |

---

## 7. Assumptions we're working from

If any of these turn out to be wrong, tell us early — several dates move.

- **A1** — The team is two people, part-time, about **25 hours a week between them**. *Every date in these documents comes from this number.*
- **A2** — GB gives us one warehouse and a willing manager for the trial.
- **A3** — The engineers are comfortable enough in Swift (Apple's language) to be productive. **We test this in the first two weeks before committing.**
- **A4** — Warehouse employees have iPhones on iOS 17+, or GB provides shared ones. *See Q1 below.*
- **A5** — GB pays for an Apple Developer account ($99/year) and hosting (about $25–50/month at trial size).

### Open questions for the CEO

| # | Question | Holds up | Needed by |
|---|---|---|---|
| **Q1** | Personal phones or company phones? This affects device management, expense reimbursement (California law requires reimbursing employees for work use of a personal phone), and whether we need shared-device login. | R1.5 design | Late Sept 2026 |
| **Q2** | Who looks after the Dynamics system — internal IT, an outside consultant, or nobody? | All of R3 | Oct 2026 |
| **Q3** | Which warehouse runs the trial, and which manager is sponsoring it? | The trial | Oct 2026 |
| **Q4** | Is there a product catalog file anywhere, even a messy spreadsheet? | R5 | Jan 2027 |
| **Q5** | Does GB have an employment lawyer who can review [WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)? | R2 clock-in | Jan 2027 |
| **Q6** | Is the plan to eventually sell this to other distributors, or is that just future-proofing? Changes how much we invest in onboarding new companies. | Roadmap past R2 | Nov 2026 |
| **Q7** | **New:** at Release 5, are we comfortable asking GB's customers to install an app to place orders? | R5 | Mid 2027 |

---

## 8. What we need from GB

| # | What | By when | What breaks if it's late |
|---|---|---|---|
| 1 | Written sign-off on this plan and on the workforce policy | **2026-09-25** | Everything |
| 2 | Apple Developer account paid ($99/yr) | **2026-09-18** | Testing on real phones, App Store |
| 3 | Hosting budget approved (~$25–50/mo) | **2026-09-25** | The test environment |
| 4 | Named trial warehouse and manager sponsor | **2026-10-09** | The trial |
| 5 | Answers to the Dynamics questionnaire | **2026-11-06** | All of R3 |
| 6 | Decision on personal vs. company phones (Q1) | **2026-10-16** | The phone app design |
| 7 | Two hours of a real manager's time to time-test task building | **2026-11-06** | Proving the "under 5 minutes" claim |
| 8 | Employment lawyer review of the workforce policy | **2027-01-15** | R2 clock-in |

---

## 9. The website question

**We are not building a website. We are also not making one impossible.**

The owner's direction is to focus on the app. That's the plan. But "no website now" and "never a website" are different decisions, and we're only making the first one.

**How we keep the door open.** All the real rules of the system — who can see what, how a repeating task becomes tomorrow's task, what counts as complete — live in the **database**, not in the iOS app. The app is a face on top of them. That means a website added later would talk to the same database and inherit every rule for free, instead of re-implementing them and drifting out of sync.

This costs us essentially nothing now. It's how we'd build it anyway.

**What adding a website later would take:**

| Scope | Rough effort at 25 hrs/week |
|---|---|
| Read-only dashboard so the CEO can check numbers in a browser | **6–8 weeks** |
| Full manager parity — build tasks, admin, reporting, all in a browser | **12–14 weeks** |
| Customer ordering on the web (the R5 problem above) | **8–10 weeks** |

**When we'd revisit it.** Three moments would make a website the obvious next move:

1. **Release 5 approaches** and asking customers to install an app looks like it'll block orders.
2. **The CEO stops opening the iPad app.** If checking the numbers requires picking up a specific device, it may just not happen — and then the visibility problem isn't solved.
3. **A second customer appears** who wants to try it without an App Store install.

None of those are decisions for today. All three are worth watching.

---

## 10. How we'll know it worked

**Release 1 succeeds if, at the end of the trial:**

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
| **The engineers aren't fast enough in Swift** | Medium | High | A two-week test in Sprint 0 — build a real screen against the real database. If it's painful, we switch approach before any real code exists |
| **Team capacity slips** — exams, jobs, life | High | High | A plan with a built-in winter break; **scope gets cut, dates don't move**; R1 is already the minimum version |
| **Dynamics turns out to be unreachable** | Medium | High | R1 and R2 need **zero** Dynamics data; a plain-CSV-file fallback is designed in from the start |
| **App Store rejects the app** | Medium | Medium | Enroll in week one; test builds go out from December, not the week before launch |
| **Pressure to add ordering or AI early** | High | High | This document. The release order *is* the commitment |
| **Two people, no backup** | Medium | High | Nobody solo-owns any part; both engineers touch everything; every decision written down in `docs/` |
| **No website becomes a problem at R5** | Medium | High | Flagged now, database designed to make a website cheap to add, decision revisited in mid-2027 |

---

## Related documents

- **[FOR-THE-CEO.md](FOR-THE-CEO.md)** — the short, plain-English version of everything here
- **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)** — what we measure about employees, and what we refuse to
- **[ENGINEERING.md](ENGINEERING.md)** — how it's actually built
- **[TIMELINE.md](TIMELINE.md)** — dates, sprint by sprint
