# OpsLink — The Plain English Version

**For:** the CEO of Goldberry Distributors
**From:** Andy Kuang
**Date:** 2026-09-09
**Reading time:** about 8 minutes

*No jargon in this document. If you only read one thing about this project, read this one. Everything here has a longer, more technical version in the other files, and you never need to open them.*

---

## The whole thing in one paragraph

We're building **one iPhone and iPad app** for Goldberry. A warehouse employee opens it in the morning and sees exactly what they're responsible for that day. They check things off as they go — and it works even in the parts of the warehouse with no signal. A manager opens the same app on an iPad, builds tomorrow's list in about five minutes, and watches today's work get finished in real time. You open it and see all four warehouses at once. **First warehouse goes live 2027-02-01. All four by 2027-02-22.**

---

## 1. What changed since the last version

Last time we proposed two things: a website for managers and a phone app for the floor.

**You asked us to build only the app. So that's what this plan is.** One app, no website.

Two honest consequences:

**It saves about two weeks, not two months.** The manager features — building the day's list, the dashboards, the history — still have to be built. They just get built on an iPad instead of in a web browser. We removed a second *place* to build things, not the things themselves. Launch moves from March 8 to **February 22**.

**We're keeping the website possible.** You said you might want to merge one in later, so we're building it so that's cheap rather than a rewrite. More on that in §7 — it costs us nothing now.

---

## 2. What you get, and when

Think of it as five deliveries. Each one is real, working software you can hold — not a demo.

### 🎯 Delivery 1 — Knowing what's getting done · **February 2027**

The one that answers your main question.

- Every employee logs in and sees today's work on their phone
- They tap to complete, and can attach a photo as proof
- **It works with no signal** and catches up when they walk back into coverage
- Managers build tomorrow's list on an iPad in under five minutes
- You get a live board: who's doing what, what's late, what's finished
- Everything in English and Spanish

**Nothing else is in this first delivery.** No ordering, no chat, no stock, no AI, no connection to Dynamics. That's on purpose — see §8.

### 🚚 Delivery 2 — The drivers · **around April 2027**
Pre-trip vehicle checks with photos, that a driver has to finish before the shift starts. Clock in and out. Notifications. An Android version for anyone not on iPhone.

*This is the one that fixes drivers skipping tire and fuel checks.*

### 🔌 Delivery 3 — Connecting to Dynamics · **around July 2027**
A nightly, read-only copy of your Dynamics data. Stock lookup from the app. The product list.

**We never write anything into Dynamics.** It stays your official system. We just read from it. Writing into a 20-year-old system that runs a working business is how you take a working business offline.

⚠️ **This delivery depends on you more than any other.** See §6.

### 📦 Delivery 4 — Pick tickets · **around October 2027**
Pick tickets generated automatically and sent to the picker's phone. Barcode scanning with the phone camera, so the pick checks itself as it happens.

*This is the one that removes hours of retyping — and makes the dedicated quality-check position unnecessary, because verification becomes part of the picking instead of a step after it.*

### 🛒 Delivery 5 — Customers ordering themselves · **early 2028**
Customers browse, add to cart, and place their own orders. Order history and billing.

*This is the one that gets routine reorders off the phone lines.*

**Then, mid-2028:** in-app messaging, your purchasing analysis and daily reports, and AI that drafts task lists from what usually happens.

---

## 3. What it fixes, from your own list

| What you told us | Which delivery fixes it |
|---|---|
| Drivers skipping tire pressure and fuel checks | **Delivery 2** — a checklist that blocks the shift until it's done |
| Pick tickets typed out by hand | **Delivery 4** — generated automatically |
| Paying someone just to double-check picks | **Delivery 4** — the scan does the checking |
| Orders re-keyed by hand | **Delivery 5** — they arrive already typed |
| Order-takers juggling several calls at once | **Delivery 5** — routine reorders move off the phone |
| **No idea whether the day's work is getting done** | **Delivery 1** — this is the whole point of the first release |

---

## 4. The two things you asked for that we're doing differently

You raised long bathroom breaks and chatting on the clock, and asked for AI to keep people on track. Those are fair frustrations and we're not brushing them off.

**But we're not going to build software that watches people. We're going to build software that watches the work.**

Here's why, in one comparison. Two things a supervisor could be told about the same person on the same day:

> *"Marcus spent 44 minutes in the bathroom today."*

> *"Marcus finished 14 of his 16 tasks. Two are overdue — the cold-storage log and the dock sweep."*

The first one you can't really act on. You can't discipline someone over it, and bringing it up is awkward at best. The second is a conversation a supervisor can have this afternoon, and it points at exactly what to fix.

**And the second one catches something the first misses completely:** the person who never leaves their station and still doesn't get their work done.

There's also a legal reason, and it's not a small one. In California, timing bathroom breaks runs into rules about breaks having to be genuinely free of employer control, and into the fact that frequent restroom use is a symptom of medical conditions that are legally protected. Scanning employees' messages runs into consent laws. **We're not lawyers**, and we're not giving you legal advice — but this is the kind of thing that turns into an expensive problem at a company with 150 employees, and we'd rather flag it now than have you find out later.

**There is a short document about exactly this: [WORKFORCE-POLICY.md](WORKFORCE-POLICY.md). Please read it, and please show it to whoever handles GB's employment law.**

**We need a yes or a no on it in writing by September 25.** If you want presence-monitoring built anyway, that's your business and your call to make — we just need to know before we start, not in February.

---

## 5. What it costs

**To run it:** about **$25 to $50 a month**, plus **$99 a year** for the Apple developer account.

That's not a typo. Dropping the website removed the web hosting bill, and everything else is one managed service.

**To build it:** two part-time engineers. Please read this next part carefully, because every date in this plan depends on it.

> The two of us have about **25 hours a week between us** — this is a part-time project alongside school. A normal full-time pair of engineers has about 80. **We run at roughly a quarter speed.** Something a full-time team would ship in a week takes us about a month.
>
> That's arithmetic, not pessimism. Planning against the real number is the difference between software your warehouse is actually using in February and a demo that never lands.

There's a two-week gap over the holidays with nothing planned. That's deliberate — it's what keeps every date after January honest.

---

## 6. What we need from you

This is the actual to-do list. **Item 1 is the one that matters most, and it's due in two weeks.**

| # | What we need | By when | What happens if it's late |
|---|---|---|---|
| **1** | **Your written yes/no on this plan and on the workforce policy** | **Sep 25, 2026** | Everything stops |
| 2 | Apple developer account paid ($99/yr) | Sep 18, 2026 | We can't put the app on anyone's phone |
| 3 | Approve about $25–50/month for hosting | Sep 25, 2026 | No place to build |
| 4 | Pick the trial warehouse and a manager who'll champion it | Oct 9, 2026 | No trial |
| 5 | Decide: personal phones or company phones? | Oct 16, 2026 | Changes how we design the app |
| 6 | **Answers about your Dynamics system** — we'll send a short questionnaire | Nov 6, 2026 | **Delivery 3 slips, and everything after it** |
| 7 | Two hours of a real manager's time, to prove the five-minute claim | Nov 6, 2026 | We can't verify our own promise |
| 8 | Your employment lawyer reads the workforce policy | Jan 15, 2027 | Clock-in can't ship |
| 9 | Two iPads for the trial warehouse | Jan 22, 2027 | No trial |

**On #5 — phones.** If employees use their own phones for work, California law generally requires reimbursing them for it. Worth deciding deliberately rather than by default.

**On #6 — the Dynamics questionnaire.** Most of it is nice-to-have. **One question matters:** *can somebody set up a nightly export from Dynamics to a folder?* If the answer is yes, Delivery 3 takes about two weeks. If nobody can answer anything about that system, Delivery 3 could take months, and Deliveries 4 and 5 sit behind it. **This is the single thing on this list you most directly control.**

---

## 7. About the website

**We're not building one. We're also not making it impossible.**

You said you might want to merge a website in later, so here's how we're handling that.

All the actual rules of the system — who can see what, how a repeating task becomes tomorrow's task, what counts as finished — live in the **database**, not inside the app. The app is a face on top of them. If we add a website later, it talks to the same database and inherits every rule automatically instead of having them rewritten and slowly drifting out of agreement.

**This costs us nothing now.** It's how we'd build it anyway. It just means we don't accidentally paint ourselves into a corner.

**If you decide you want one later, roughly:**

- A read-only dashboard so you can check the numbers in a browser: **6–8 weeks**
- Full manager capability in a browser: **12–14 weeks**
- Customer ordering on the web: **8–10 weeks**

**Three things would make us come back and recommend one:**

1. **When Delivery 5 gets close.** Asking a restaurant owner to download an app from the App Store just to reorder produce is a much bigger ask than sending them a link. This is where "no website" costs the most, and it's worth deciding by mid-2027.
2. **If you stop opening the iPad app.** If checking your numbers means picking up a specific device, it may just not happen — and then the visibility problem isn't actually solved. We'd rather notice that and fix it than watch it quietly fail.
3. **If a second company wants to try it** without an App Store install.

None of these are decisions for today.

---

## 8. Why the first delivery looks small

You'll notice Delivery 1 has no ordering, no chat, no stock, no AI. That's intentional, and there are two reasons.

**Reason one: the app has to actually get used.** The single biggest risk to this whole project isn't technical — it's that the crew decides the app is a hassle, or worse, that it's surveillance, and stops opening it. An accountability system nobody uses is *worse* than no system, because it hands you numbers you trust that aren't real.

So the first version does one thing well: it tells you what you're responsible for today, and lets you check it off. If people use it for that, we add to it. If they don't, we fix that before adding anything.

Related: you described a five-tab app like OMELINK. We're getting there — but in the first version, a warehouse employee sees **two tabs**, not five. Three tabs that don't do anything teach people the app is unfinished and not worth opening, and with a warehouse crew you get one first impression.

**Reason two: nothing important should depend on Dynamics.** We don't know yet how hard your Dynamics system is to connect to. It could be two weeks. It could be four months. So we deliberately built Deliveries 1 and 2 to need **nothing** from Dynamics. Even if that connection never happens at all, you still get the thing you said you most wanted, on schedule.

---

## 9. What could go wrong

We'd rather tell you now than in February.

**The crew rejects it as surveillance.** *Most likely and most damaging.* We handle it with the workforce policy, by letting every employee see their own numbers, and by having the manager introduce it as *"here's what you're responsible for today"* rather than *"here's how we'll check on you."*

**We're not fast enough in Apple's programming language.** We're switching to Apple's native tools, which is the right choice for an app-only product — but we have to be good at it. **So we test that in the first two weeks**, before writing anything real. If it's painful, we switch approach immediately and it costs about three weeks instead of three months.

**We fall behind.** Two part-time students. Exams happen. **Our rule is: dates hold, features get cut.** We've already written down what gets dropped first and in what order, so we're not making that decision in a panic in January.

**Dynamics turns out to be unreachable.** Deliveries 1 and 2 need nothing from it, and our plan for connecting is the simplest possible one — a nightly export file, which works with every version of Dynamics ever made.

**Apple rejects the app.** We enroll in week one and start submitting test builds in December, not the week before launch.

---

## 10. What we need from you right now

Three things, by **September 25**:

1. **Read [WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)** — it's short — and give us a written yes or no.
2. **Say yes or no to this plan overall**, including the February 22 launch date.
3. **Approve the two small costs**: $99/year for Apple, ~$25–50/month for hosting.

And when you have a moment: tell us **which warehouse** should go first, and **which manager** would be a good champion for it. That person matters more to whether this works than almost anything we build.

---

## If you want more detail

You don't need any of these. They're here if you want them.

| Document | What's in it |
|---|---|
| **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)** | ⚠️ **Please do read this one.** What we measure about employees and what we refuse to. Show it to your employment lawyer. |
| [PRODUCT.md](PRODUCT.md) | The full feature list, every role, every release, how we'll measure success |
| [TIMELINE.md](TIMELINE.md) | Week-by-week schedule, and exactly what gets cut first if we fall behind |
| [ENGINEERING.md](ENGINEERING.md) | The technical detail. Section 7 explains every major decision in plain language if you're curious. |
