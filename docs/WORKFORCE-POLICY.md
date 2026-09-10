# What We Measure About People — And What We Won't

**Product:** OpsLink
**Version:** 2.0 — 2026-09-09
**Who should read this:** the CEO, and GB's employment lawyer
**Status:** ⚠️ **Needs the CEO's written yes or no before 2026-09-25**

---

## Read this first

This is a short document about a genuinely important decision, so it's written for a person, not a lawyer or an engineer.

**We are not lawyers.** Nothing here is legal advice. What's below is our reasoning for a product decision, and the specific reason we think GB's employment counsel should look at it before anything ships. If GB's lawyer disagrees with any of it, they're right and we're wrong.

---

## 1. What the CEO asked for

Three things came up, and they're all completely reasonable frustrations for someone running four warehouses:

1. **Employees taking 30+ minute bathroom breaks.**
2. **People chatting instead of working while on the clock.**
3. **"AI to keep employees on track."**

Nobody reading this thinks those frustrations are unfair. If you're paying for eight hours and getting six, that's a real problem and it costs real money.

## 2. What we're going to do about it

**We're going to solve the problem you actually have, using a different tool than the one you asked for.**

The question underneath all three complaints is the same one: **"Is this person doing their work?"**

There are two ways to answer that question with software.

| Approach | What it looks like | What it tells you |
|---|---|---|
| **Watch the person** | Time their breaks, detect idle time, track them around the building, scan their messages | How long they were away from a spot |
| **Watch the work** | Track what was assigned, what got done, when, and whether it was on time | Whether the job got done |

**We're building the second one.** Not as a compromise — because it's the better answer to your question.

Here's the difference in practice. Two things a supervisor could be told about the same employee on the same day:

> *"Marcus spent 44 minutes in the bathroom today."*

> *"Marcus completed 14 of his 16 assigned tasks. Two are overdue: the cold-storage temperature log and the dock sweep."*

The first one is a problem you can't act on. You can't discipline someone for it, you can't coach on it, and raising it is uncomfortable at best. The second one is a conversation a supervisor can have this afternoon, without a lawyer in the room, and it points at exactly what needs to change.

**And note what the second one catches that the first doesn't:** an employee who never leaves their station and still doesn't finish their work. Presence tracking misses that person entirely.

---

## 3. Why not just build what was asked for

Two independent reasons. Either one alone would be enough.

### Reason 1 — It's the better product

Covered above. Output tells you more than presence does, and it's actionable.

### Reason 2 — In California specifically, presence-tracking creates legal exposure GB doesn't currently have

We're not lawyers. But these four points are well-established enough that we'd be doing GB a disservice not to raise them.

**Breaks have to be genuinely free of employer control.** California law requires that rest and meal breaks be duty-free — the employee is fully relieved. A system that times or flags bathroom trips invites the argument that breaks weren't really relinquished, because the employer was still measuring them. That's the kind of claim that becomes a class action in a company with 150 employees.

**How often someone uses the restroom is protected medical information.** Frequent restroom use is a symptom of pregnancy, diabetes, Crohn's disease, IBS, and the side effects of many common medications. Disability and pregnancy discrimination law protects all of it. A system that flags "excessive" restroom time is, in practice, flagging medical conditions — and it does that even if nobody intended it to. The intent doesn't matter much; the effect does.

**Recording or scanning conversations needs everyone's consent in California.** California requires all parties to consent to the recording of a confidential communication. Automatically scanning employees' in-app messages for productivity signals sits in genuinely hazardous territory.

**New rules on automated decision-making are arriving right around Release 6.** California's privacy regulations now impose notice, opt-out, and risk-assessment obligations on automated technology used to make employment decisions. If we build an engine that scores individual employees, GB inherits a compliance project — notices, opt-outs, formal risk assessments — right when the AI features would launch. Building output metrics instead avoids creating that obligation in the first place.

**The practical summary:** presence-tracking would take four risks GB doesn't currently carry and attach them to a system GB paid for and can be subpoenaed about. Output-tracking carries none of them and answers the question better.

---

## 4. The specific line we draw

| ✅ We build this | ❌ We don't build this |
|---|---|
| Task assignment and completion rates | Bathroom or break duration tracking |
| On-time completion %, per person and per team | Idle-time or "inactivity" detection |
| Pre-trip vehicle check compliance | Continuous location tracking, or any tracking inside the building |
| Pick accuracy and order cycle time | Reading or scanning message content |
| Clock in/out against **the property line only** | Following someone's movement around the warehouse |
| AI that drafts task lists and flags **operational** problems — *"Warehouse 2's pick accuracy dropped 12% this week"* | AI that scores, ranks, or rates individual employees |

**About the clock-in geofence.** In Release 2, clocking in checks that you're at the warehouse — a circle around the property. It does not track where you go once you're inside, and it isn't running while you're on shift. It answers "are you at work?" and nothing else.

---

## 5. Five rules every metric has to follow

These are commitments, not aspirations. They're checked in code review, and there's a specific audit against them in Sprint 4b.

**Rule 1 — Numbers about individuals are advisory only.**
The system never generates discipline, warnings, rankings, or a "bottom performer" list. It shows a human being some numbers, and the human being decides what to do. There is no automated employment action anywhere in this product.

**Rule 2 — Every number links to its evidence.**
If the dashboard says someone completed 82%, tapping it shows exactly which tasks, when, and which ones weren't finished. No number appears without the facts behind it. A number you can't check is a number you can't defend.

**Rule 3 — Employees see their own numbers, exactly as their manager sees them.**
Same screen, same figures, no lag. **No secret scores about anyone, ever.**

This one matters more than it looks. It's the difference between an employee thinking *"they're watching me"* and thinking *"I can see where I stand."* The first kills adoption. The second is the thing that makes the app worth opening.

**Rule 4 — Everyone is told what's collected, at first login.**
A plain-language notice, in English and Spanish, before anyone uses the app: what's collected, why, and who sees it. No fine print, no surprises.

**Rule 5 — The system never guesses why someone isn't at their station.**
No feature infers a reason for an absence. Not a break, not a medical issue, not slacking. It doesn't know, so it doesn't say.

---

## 6. What this means for how the app gets introduced

This is the part that decides whether the whole project works.

**The biggest risk in this entire project is the crew deciding the app is surveillance.** If that happens in week one of the trial, it's very hard to undo — and an accountability system nobody opens is worse than no system, because it produces numbers you trust that aren't real.

The policy above is what makes an honest introduction possible. Specifically:

- The manager sponsor introduces it as **"here's what you're responsible for today"** — not "here's how we'll check on you."
- Every employee can see their own numbers from day one. Point that out explicitly.
- The privacy notice is read, not skipped past.
- When someone asks *"does this track my breaks?"* — and someone will, on day one — the answer is a flat **no**, and it's true, and they can verify it by looking at their own screen.

You only get one first impression with a warehouse crew. This is what makes that first impression a truthful one.

---

## 7. What we need from GB

| # | What | By when |
|---|---|---|
| 1 | **The CEO accepts or rejects this policy in writing** | **2026-09-25** |
| 2 | GB's employment lawyer reviews this document | **2027-01-15**, before clock-in ships in Release 2 |
| 3 | GB confirms the wording of the privacy notice | **2027-01-15** |

**On item 1:** we need a clear yes or no, not silence.

If the answer is **yes**, we build as described and this document becomes the standing rule.

If the answer is **no** — if GB wants presence-monitoring built anyway — that's GB's call to make, not ours. We'd want it documented, we'd want employment counsel involved before we write any of it, and we'd want the crew-rejection risk re-examined honestly. But it is the CEO's business and the CEO's decision.

What we're not willing to do is leave it ambiguous and find out in February that we built the wrong thing.

---

## Related documents

- **[FOR-THE-CEO.md](FOR-THE-CEO.md)** — the plain-English overview of the whole project
- **[PRODUCT.md](PRODUCT.md)** — what we're building
- **[ENGINEERING.md](ENGINEERING.md)** — how it's built (decision D7 is this policy in short form)
- **[TIMELINE.md](TIMELINE.md)** — the schedule
