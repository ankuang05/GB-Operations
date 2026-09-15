# OpsLink — The Plain English Version

**For:** the CEO of Goldberry Distributors
**From:** Andy Kuang
**Date:** 2026-09-14
**Reading time:** about 9 minutes

*No jargon in this document. If you only read one thing about this project, read this one. Everything here has a longer, more technical version in the other files, and you never need to open them.*

---

## The whole thing in one paragraph

We're building **one iPhone and iPad app** for Goldberry. A warehouse employee opens it in the morning and sees exactly what they're responsible for that day. They check things off as they go — and it works even in the parts of the warehouse with no signal. A manager opens the same app on an iPad, builds tomorrow's list in about five minutes, and watches today's work get finished in real time. You open it and see all four warehouses at once. **First warehouse goes live 2027-02-08. All four by 2027-03-01.**

---

## 1. What changed since the last version

Your fuller brief landed, and it changed four things. Three are improvements. One moved a date, and we want you to see exactly why.

**We're following your six phases.** You laid out the order you want things built — interface, then logins, then your Dynamics data, then your dashboard, then customers, then the extras. We've rebuilt the schedule on that order instead of ours.

**That order costs one week, and here's where it goes.** Building screens before logins exist means those screens get revisited once logins arrive. It's about twenty hours of rework. We think it's worth paying, because it means you see something real in October instead of January. But it's why launch is now **March 1** rather than February 22. We'd rather show you the arithmetic than quietly move a date.

**Six job types became three kinds of login.** You group your people as management, employees, and customers — so we do too. Picker, QC, driver, receiver and buyer become *labels on a person* rather than separate logins. Simpler for you to administer, and considerably safer for us to get right.

**We wrote down how Goldberry actually runs today.** The order path from phone call to signed invoice, and the purchasing path from buyer to put-away. It's in [OPERATIONS-TODAY.md](OPERATIONS-TODAY.md). **Please check it** — one half of it we're confident about, and the other half is our best guess (see §7).

---

## 2. What you get, and when

Six phases. Each one is real, working software you can hold — not a demo.

### 🎯 Phase 1 — The app itself · **Sep – Dec 2026**

- Managers build the day's list on an iPad, in under five minutes
- Employees see today's work on their phone and tap to complete it, with a photo as proof
- **It works with no signal** and catches up when they walk back into coverage
- A live board: who's doing what, what's late, what's finished
- Everything in English and Spanish

### 🔑 Phase 2 — Everyone gets their own login · **Dec 2026 – Jan 2027**
Real credentials per person. Management, employees, customers. You see all four warehouses; a manager sees theirs. And a hard guarantee, checked automatically every time we change any code, that one company's data can never be seen by another.

**Phases 1 and 2 together are what goes into your trial warehouse on February 8, and to all four on March 1.** They need *nothing* from Dynamics — which is deliberate, and explained in §9.

### 🔌 Phase 3 — Your Dynamics data · **as soon as you can unblock it, earliest March 2027**
A nightly, read-only copy of your Dynamics data. Stock lookup from the app. The beginnings of a product list.

**About writing orders back into Dynamics.** You asked for this, and you're right that it's the real fix for re-keying orders by hand. We're not saying no — we're saying not yet. Nobody has yet told us which version of Dynamics this is, whether it can accept new records safely, whether there's a test copy to try it on, or who administers it. Until those have answers, data flows one way: out of Dynamics, into the app. **Writing into a twenty-year-old system that runs a working business is how you take a working business offline**, and we'd want a proper trial run before going near your inventory and receivables.

⚠️ **This phase depends on you more than any other.** See §6.

### 📊 Phase 4 — Your dashboard · **around mid-2027**
The employee breakdown you asked for. Completion trends, per-person history, overdue analysis, all four warehouses side by side, plus inventory and purchasing analysis once Phase 3 has landed.

*(The live "what's happening right now" board doesn't wait for this — it's in Phase 1, because your trial can't prove anything without it. Phase 4 is the history and the trends on top.)*

### 🛒 Phase 5 — Customers ordering themselves · **late 2027 / early 2028**
Customers browse, add to cart, and place their own orders. Order history and billing.

*This is the one that gets routine reorders off the phone lines.* ⚠️ **It has a blocker that isn't software** — see §7.

### ✨ Phase 6 — The rest · **2028**
In-app messaging, notifications, the driver's pre-trip vehicle check, clock in and out, AI that drafts task lists from what usually happens, and an Android version.

---

## 3. What it fixes, from your own list

| What you told us | Which phase fixes it |
|---|---|
| **No idea whether the day's work is getting done** | **Phase 1** — this is the whole point of going first |
| Wanting a daily task list for warehouse staff | **Phase 1** — you build it, they see it next morning |
| Drivers skipping tire pressure and fuel checks | **Phase 1** as a task with a photo; **Phase 6** as a proper checklist that blocks the shift |
| Orders re-keyed by hand into Dynamics | **Phase 3 at the earliest**, and only once we know Dynamics can take it safely |
| Order-takers juggling several calls at once | **Phase 5** — routine reorders move off the phone |
| Pick tickets typed out by hand | **Not scheduled yet** — it needs Phase 3 first |
| Paying someone just to double-check picks | **Not scheduled yet** — same reason |

**We want to be straight about the bottom two rows.** Pick tickets and the QC position were on the previous plan for late 2027. They depend entirely on having your Dynamics data, and we've stopped putting dates on things that depend on a connection nobody has confirmed is possible. They're still the goal. They're just not a promise until Phase 3 is real.

---

## 4. The things you asked for that we're doing differently

You raised long bathroom breaks, chatting on the clock, AI to keep people on track, and — in the new brief — **cameras with computer vision to flag workers who are slacking off.** Those are fair frustrations and we're not brushing them off.

**But we're not going to build software that watches people. We're going to build software that watches the work.**

Here's why, in one comparison. Two things a supervisor could be told about the same person on the same day:

> *"Marcus spent 44 minutes in the bathroom today."*

> *"Marcus finished 14 of his 16 tasks. Two are overdue — the cold-storage log and the dock sweep."*

The first one you can't really act on. You can't discipline someone over it, and bringing it up is awkward at best. The second is a conversation a supervisor can have this afternoon, and it points at exactly what to fix.

**And the second one catches something the first misses completely:** the person who never leaves their station and still doesn't get their work done.

### On the cameras specifically

Your brief says we're aware of the surveillance objection but aren't responsible for it, so we could build it and let Goldberry decide whether to turn it on.

**We don't think that works, and we'd rather tell you now than leave it quietly off the list.** When a monitoring system ends up as evidence in a wrongful-termination or disability claim — and in California, at 150 employees, that's a matter of time — the people who *built* it get named next to the people who *ran* it. "They chose to switch it on" isn't a defence that has held up.

There's a plainer problem too. Your crew won't distinguish between the tab that shows their tasks and the camera watching them work. It's one app, and it's the app management installed. The day that feature exists is the day everything else we're doing to get people to actually open it stops working.

There's also the legal detail on the other two: in California, timing bathroom breaks runs into rules about breaks being genuinely free of employer control, and into the fact that frequent restroom use is a symptom of medical conditions that are legally protected. Scanning employees' messages runs into consent laws. **We're not lawyers**, and this isn't legal advice — but it's the kind of thing that turns into an expensive problem, and we'd rather flag it now.

**There is a short document about exactly this: [WORKFORCE-POLICY.md](WORKFORCE-POLICY.md). Please read it, and please show it to whoever handles GB's employment law.**

**We need a yes or a no on it in writing by September 25.** If you want presence-monitoring built anyway, that's your business and your call — we just need to know before we start, not in February.

---

## 5. What it costs

**To run it:** about **$25 to $50 a month**, plus **$99 a year** for the Apple developer account.

**To build it:** two part-time engineers. Please read this next part carefully, because every date in this plan depends on it.

> The two of us have about **25 hours a week between us** — this is a part-time project alongside school. A normal full-time pair of engineers has about 80. **We run at roughly a quarter speed.** Something a full-time team would ship in a week takes us about a month.
>
> That's arithmetic, not pessimism. Planning against the real number is the difference between software your warehouse is actually using in February and a demo that never lands.

There's a two-week gap over the holidays with nothing planned. That's deliberate — it's what keeps every date after January honest.

---

## 6. What we need from you

This is the actual to-do list. **Item 1 is the one that matters most, and it's due in under two weeks.**

| # | What we need | By when | What happens if it's late |
|---|---|---|---|
| **1** | **Your written yes/no on this plan and on the workforce policy** | **Sep 25, 2026** | Everything stops |
| 2 | Apple developer account paid ($99/yr) | Sep 18, 2026 | We can't put the app on anyone's phone |
| 3 | Approve about $25–50/month for hosting | Sep 25, 2026 | No place to build |
| 4 | Pick the trial warehouse and a manager who'll champion it | Oct 9, 2026 | No trial |
| 5 | Decide: personal phones or company phones? | Oct 16, 2026 | Changes how we design the app |
| 6 | **Answers about your Dynamics system** — we'll send a short questionnaire | Nov 6, 2026 | **Phase 3 slips, and everything that needs it** |
| 7 | Two hours of a real manager's time, to prove the five-minute claim | Nov 6, 2026 | We can't verify our own promise |
| 8 | **New: walk us through how purchasing and receiving actually work** | Dec 4, 2026 | We'd be building on a guess — see §7 |
| 9 | Your employment lawyer reads the workforce policy | Jan 15, 2027 | Clock-in can't ship |
| 10 | Two iPads for the trial warehouse | Feb 1, 2027 | No trial |
| 11 | **New: name someone to build the product list, and a date** | Mar 1, 2027 | **Phase 5 has no start date** — see §7 |

**On #5 — phones.** If employees use their own phones for work, California law generally requires reimbursing them for it. Worth deciding deliberately rather than by default.

**On #6 — the Dynamics questionnaire.** Most of it is nice-to-have. **One question matters:** *can somebody set up a nightly export from Dynamics to a folder?* If yes, Phase 3 takes about two weeks. If nobody can answer anything about that system, Phase 3 could take months. **This is the single thing on this list you most directly control.**

---

## 7. Three things we found that you should know about

### There is no product list

Not a spreadsheet, not a printout, nothing. Goldberry has been around long enough that your customers simply know what you sell and order it by name.

That's a genuine strength of a twenty-year business. It's also a wall in front of Phase 5: **a customer cannot browse a catalog that has never been written down.** Someone at Goldberry has to write down what you sell — with units and pack sizes and what's actually orderable — and that's a real project with a real owner, not something we can do from outside. Until it has a name and a date against it, Phase 5 doesn't have a start date either. That's item 11 above.

### We don't actually know how your purchasing works

We wrote down the order path — phone call, handwritten note, Dynamics, printed pick ticket, pick, QC, truck, signature, back to the office — and we're confident about it because you walked us through it.

**Purchasing and receiving, we guessed.** Buyer decides what to order, PO to the vendor, truck arrives, receiver checks against the paperwork, cases get labelled and put away, quantities go into Dynamics. That's a reasonable guess and it might be exactly right. It might also be wrong in ways that would make anything we built on it useless.

So we've labelled it a guess everywhere it appears, and we're not committing to any purchasing feature until someone walks us through the real thing. **That's an hour of somebody's time** and it's item 8 above.

### You are the routing system

Trucks run the same routes every few days, and you re-plan by hand when a driver is out sick. That works — the drivers have it down and it isn't costing you anything today.

We're not touching it, and route planning isn't on the roadmap. But it's worth saying out loud: **the knowledge of how routes work lives with one person, and that person is you.** Not something to fix this year. Something to be aware of.

---

## 8. About the website

**We're not building one. We're also not making it impossible.**

All the actual rules of the system — who can see what, how a repeating task becomes tomorrow's task, what counts as finished — live in the **database**, not inside the app. The app is a face on top of them. If we add a website later, it talks to the same database and inherits every rule automatically instead of having them rewritten and slowly drifting out of agreement.

**This costs us nothing now.** It's how we'd build it anyway.

**If you decide you want one later, roughly:**

- A read-only dashboard so you can check the numbers in a browser: **6–8 weeks**
- Full manager capability in a browser: **12–14 weeks**
- Customer ordering on the web: **8–10 weeks**

**Three things would make us come back and recommend one:**

1. **When Phase 5 gets close.** Asking a restaurant owner to download an app just to reorder produce is a much bigger ask than sending them a link. This is where "no website" costs the most, and it's worth deciding by mid-2027.
2. **If you stop opening the iPad app.** If checking your numbers means picking up a specific device, it may just not happen — and then the visibility problem isn't actually solved.
3. **If a second company wants to try it** without an App Store install.

None of these are decisions for today.

---

## 9. Why the first phase looks small

You'll notice Phases 1 and 2 have no ordering, no chat, no stock, no AI. That's intentional, and there are two reasons.

**Reason one: the app has to actually get used.** The single biggest risk to this whole project isn't technical — it's that the crew decides the app is a hassle, or worse, that it's surveillance, and stops opening it. An accountability system nobody uses is *worse* than no system, because it hands you numbers you trust that aren't real.

So the first version does one thing well: it tells you what you're responsible for today, and lets you check it off. If people use it for that, we add to it. If they don't, we fix that before adding anything.

Related: you described a five-tab app like OMELINK. We're getting there — but at first, a warehouse employee sees **two tabs**, not five. Three tabs that don't do anything teach people the app is unfinished, and with a warehouse crew you get one first impression.

*(One note on OMELINK: it actually runs **separate** apps for customers and for employees. You asked how a single app can serve four very different jobs without turning into four apps in a trenchcoat — that's a good question, and the competitor you pointed us at answered it by not trying. We still think one app is right for you. But it's a bet, and you should know it's a bet.)*

**Reason two: nothing important should depend on Dynamics.** We don't know yet how hard your Dynamics system is to connect to. It could be two weeks. It could be four months. So Phases 1 and 2 need **nothing** from it. Even if that connection never happens at all, you still get the thing you said you most wanted, on schedule.

---

## 10. What could go wrong

We'd rather tell you now than in February.

**The crew rejects it as surveillance.** *Most likely and most damaging.* We handle it with the workforce policy, by letting every employee see their own numbers, and by having the manager introduce it as *"here's what you're responsible for today"* rather than *"here's how we'll check on you."*

**The product list turns out to be a bigger job than the app.** Writing down everything Goldberry sells, accurately, may take longer than anyone expects. We've flagged it now rather than discovering it in 2028.

**We're not fast enough in Apple's programming language.** **So we test that in the first two weeks**, before writing anything real. If it's painful, we switch approach immediately and it costs about three weeks instead of three months.

**We fall behind.** Two part-time students. Exams happen. **Our rule is: dates hold, features get cut.** We've already written down what gets dropped first and in what order, so we're not making that decision in a panic in January.

**Dynamics turns out to be unreachable.** Phases 1 and 2 need nothing from it, and our plan for connecting is the simplest possible one — a nightly export file, which works with every version of Dynamics ever made.

**Apple rejects the app.** We enroll in week one and start submitting test builds in November, not the week before launch.

---

## 11. What we need from you right now

Three things, by **September 25**:

1. **Read [WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)** — it's short — and give us a written yes or no.
2. **Say yes or no to this plan overall**, including the **March 1** launch date.
3. **Approve the two small costs**: $99/year for Apple, ~$25–50/month for hosting.

And when you have a moment: tell us **which warehouse** should go first, and **which manager** would be a good champion for it. That person matters more to whether this works than almost anything we build.

---

## If you want more detail

You don't need any of these. They're here if you want them.

| Document | What's in it |
|---|---|
| **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)** | ⚠️ **Please do read this one.** What we measure about employees and what we refuse to. Show it to your employment lawyer. |
| **[OPERATIONS-TODAY.md](OPERATIONS-TODAY.md)** | How Goldberry runs today, as we understand it. **Worth ten minutes to check we got it right.** |
| [PRODUCT.md](PRODUCT.md) | The full feature list, every role, every phase, how we'll measure success |
| [TIMELINE.md](TIMELINE.md) | Week-by-week schedule, and exactly what gets cut first if we fall behind |
| [ENGINEERING.md](ENGINEERING.md) | The technical detail. Section 7 explains every major decision in plain language if you're curious. |
