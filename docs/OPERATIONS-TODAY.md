# How Goldberry Runs Today

**Version:** 1.0 — 2026-09-14
**Purpose:** the factual substrate everything else is built on. What GB actually does, before we change any of it.
**Companion to:** [PRODUCT.md](PRODUCT.md) · [ENGINEERING.md](ENGINEERING.md)

> **Read the confidence labels.** Parts of this document are observed fact from the CEO. One major part — purchasing and receiving — is explicitly an **assumption**, flagged as such in the CEO's own brief. Anything we build on an assumption is a guess wearing a requirement's clothes, so the labels stay until someone walks the floor and confirms.

---

## 1. The business

Goldberry Distributors is a food distributor, 20+ years old, four warehouses across the California Bay Area and the Sacramento area.

| | |
|---|---|
| **What they sell** | Meat · Seafood · Vegetables · Dry goods |
| **Who they sell to** | Meat markets · Supermarkets · Other food distributors · Restaurants |
| **How they get business** | Returning customers and word of mouth. No sales engine to speak of. |
| **System of record** | Microsoft Dynamics — **"2009, or an older version — not sure"** (the brief's own words) |

That last row matters more than it looks, and the uncertainty in it is the point. A 2009-era Dynamics install has no modern way to be asked a question over the internet, GB's twenty years of history all live inside it, and **nobody has yet told us which version it actually is or who administers it.** Every estimate for Phase 3 carries that unknown — see [ENGINEERING.md §10](ENGINEERING.md#10-appendix--dynamics-questionnaire) for the questionnaire that closes it.

**The CEO's priority is efficiency, not growth.** Nothing in the plan should read as a growth feature dressed up as an operations one.

---

## 2. The sales order, end to end

**Confidence: observed.** This is the CEO's description of the real path an order takes.

```
Customer calls ──▶ employee writes it down ──▶ typed into Dynamics 2009
                                                        │
                                          pick ticket typed up + printed
                                                        │
                                    warehouse worker picks to staging area
                                                        │
                                       QC person re-checks against ticket
                                                        │
                                       loaded onto the assigned driver's truck
                                                        │
                              driver runs the route, customer signs the invoice
                                                        │
                            signed invoices return to the office, day is closed out
```

**Where the order comes in:** phone call primarily, but also **text, WeChat, and email**. Four uncoordinated channels with no shared queue between them.

### What this path costs

| # | Friction point | What it costs |
|---|---|---|
| 1 | Order is spoken, then handwritten, then typed | Two transcription opportunities before it reaches a system |
| 2 | Several calls land at once and staff cannot answer both | **Orders are lost.** The CEO raised this directly. |
| 3 | The pick ticket is typed by hand and printed | Hours of re-keying data that is already in Dynamics |
| 4 | A QC person re-checks every pick | A full salary spent verifying work that was never recorded as it happened |
| 5 | Proof of delivery is a signature on paper, carried back in a truck | Nothing is knowable until the driver returns |
| 6 | Nothing between intake and close-out leaves a queryable trace | The CEO cannot answer *"is it getting done?"* without walking the floor |

### Routing

**The CEO personally determines truck routes.** In practice routes are stable — the same trucks run the same route every N days and the drivers have the workflow down — so he intervenes mainly when something breaks, most often **a driver out sick**.

This is worth naming as its own fact rather than burying it in the flow: **route planning is a person, not a system, and that person is the CEO.** We are not replacing it, and route optimization stays out of scope. But it is a single point of failure in daily operations and it belongs on the record.

---

## 3. The purchase order and receiving

> ⚠️ **Confidence: ASSUMED, NOT VERIFIED.**
> The CEO's brief states outright that how GB actually runs this process has not been determined. What follows is the working assumption. **No requirement may be committed on top of this section until a walkthrough happens.**

```
Buyer decides what to order (based on what's selling and what's low)
                     │
        PO entered and sent to vendor ──▶ vendor ships
                     │
    truck arrives ──▶ receiver checks product against the printed PO
                      and the driver's Bill of Lading
                     │
         checks: count · weight · temperature · condition · dates
                     │
     cases labeled, given a warehouse location, put away
                     │
           received quantities entered into Dynamics
                     │
    signed BOL and receiving paperwork to the office ──▶ vendor paid
```

**What we would need to confirm before building anything here:**

- Who the buyer is, and whether purchasing decisions are one person's or several
- Whether "what's been selling and what's running low" comes from a Dynamics report or from memory
- Whether receiving discrepancies (short counts, temperature failures, expired dates) have a defined path today, or get handled by walking to the office
- Whether the warehouse location assigned at put-away is recorded anywhere, or held in the receiver's head

Each of those changes what a receiving feature would even be. Until they are answered, receiving is a topic, not a requirement.

---

## 4. Who does what

The CEO's brief groups people into **three login buckets**, with specific jobs sitting inside them. We follow that grouping — see [PRODUCT.md §3](PRODUCT.md#3-who-uses-it).

| Bucket | Jobs inside it | Notes |
|---|---|---|
| **Management** | CEO, managers | Also take customer orders and place orders with vendors. The CEO's view spans all four warehouses; a manager's is one. |
| **Employee** | Picker/stager · QC · Driver · Receiver · Buyer | The brief calls them **"task followers."** That phrase is the whole design brief for the employee experience. |
| **Customer** | — | Meat markets, supermarkets, restaurants, distributors |

---

## 5. What the CEO wants fixed

His own list, sorted by whether software is the right instrument.

### Software is the right instrument

| Problem | Where it shows up above |
|---|---|
| Pick tickets typed up by hand | §2, friction #3 |
| Orders manually inserted into the system | §2, friction #1 |
| Remove the QC person from `pick → QC → load` | §2, friction #4 |
| Several simultaneous calls while taking orders | §2, friction #2 — partial fix is digital ordering |
| Drivers skipping routine checks (tire pressure, fuel) | Not in the flow above; a pre-trip gap |
| No daily task list for warehouse staff | The CEO wants to assign the day's work and have staff see it next morning |

### Software is *not* the right instrument

| Problem | Why it is handled differently |
|---|---|
| Staff in the bathroom 30+ minutes | Measuring bodily presence is legally exposed in California and answers the wrong question |
| Chatting on the clock, clocking out late | Same |

The real question behind both is *"is the work getting done?"* — which is answered by measuring **work finished**, not time in a room. Full reasoning and the specific line we draw: **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)**.

---

## 6. Two facts that reshape the roadmap

These emerged from the CEO's brief and each one changes a downstream plan.

### 6.1 There is no product catalog

Not a database table, not a spreadsheet, not a printed list. GB has been in business long enough that **customers simply know what GB sells**, and orders are placed by name over the phone.

**Consequence:** the customer ordering experience (Phase 5) cannot be built on top of nothing. Someone has to write down what GB sells — with units, pack sizes, and what is actually orderable — before a customer can browse it. That is a GB project with its own owner and date, and it is on the critical path for Phase 5 whether or not anyone schedules it.

### 6.2 The competitor solved the hard design problem by avoiding it

**OMELINK** is cited in the brief as inspiration: a mobile app in this market with **separate customer-facing and employee-facing experiences**.

That is worth sitting with. The brief's own open question is *how one shared interface flexes to fit very different jobs without becoming several different apps wearing the same shell* — and the reference product's answer was to not share the shell at all. We are still pursuing one adaptive app, for good reasons ([ENGINEERING.md D1](ENGINEERING.md#d1--one-app-that-adapts-not-separate-iphone-and-ipad-apps)), but we should stop treating the single shell as obviously correct. It is a bet, and the nearest comparable product bet the other way.

---

## Related documents

- **[PRODUCT.md](PRODUCT.md)** — what we're building on top of all this
- **[TIMELINE.md](TIMELINE.md)** — when
- **[ENGINEERING.md](ENGINEERING.md)** — how
- **[WORKFORCE-POLICY.md](WORKFORCE-POLICY.md)** — what we measure about people, and what we refuse to
- **[FOR-THE-CEO.md](FOR-THE-CEO.md)** — the plain-English overview
