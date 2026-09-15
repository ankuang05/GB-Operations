# OpsLink

Planning documents for **OpsLink** — an operations app for distribution warehouses.

One iOS app that changes shape depending on who opens it. Warehouse employees see today's work on an iPhone and check it off, even with no signal. Managers build tomorrow's list on an iPad and watch it get done. The CEO sees all four warehouses at once.

**First customer:** Goldberry Distributors, a food distributor with four warehouses across the California Bay Area and Sacramento.

---

## Start here

| If you are… | Read this |
|---|---|
| **The CEO, or anyone non-technical** | **[docs/FOR-THE-CEO.md](docs/FOR-THE-CEO.md)** — the whole project in plain English, 9 minutes |
| Trying to understand how GB works today | **[docs/OPERATIONS-TODAY.md](docs/OPERATIONS-TODAY.md)** — the current workflow, with confidence labels |
| Reviewing what we measure about employees | **[docs/WORKFORCE-POLICY.md](docs/WORKFORCE-POLICY.md)** — short, and needs a decision |
| Looking for the feature list | [docs/PRODUCT.md](docs/PRODUCT.md) |
| Looking for dates | [docs/TIMELINE.md](docs/TIMELINE.md) |
| An engineer | [docs/ENGINEERING.md](docs/ENGINEERING.md) |
| Starting implementation | [.claude/prds/opslink.prd.md](.claude/prds/opslink.prd.md) → then `/plan` |

---

## The documents

| Document | What's in it |
|---|---|
| [FOR-THE-CEO.md](docs/FOR-THE-CEO.md) | Plain-English overview. No jargon. The one to read first. |
| [OPERATIONS-TODAY.md](docs/OPERATIONS-TODAY.md) | How GB actually runs today — sales orders, purchasing, routing — and **which parts are verified versus assumed** |
| [WORKFORCE-POLICY.md](docs/WORKFORCE-POLICY.md) | What the app measures about people and what it refuses to — and why. **Awaiting the CEO's written decision.** |
| [PRODUCT.md](docs/PRODUCT.md) | Roles, features, phase scope, success criteria, open questions |
| [ENGINEERING.md](docs/ENGINEERING.md) | Architecture, data model, security, offline design, and every major decision with its cost |
| [TIMELINE.md](docs/TIMELINE.md) | Phase-by-phase schedule, the capacity arithmetic, and what gets cut first |
| [.claude/prds/opslink.prd.md](.claude/prds/opslink.prd.md) | The requirements-phase PRD — problem, evidence, hypothesis, milestones |

---

## Where things stand

**Planning only. No code has been written yet.**

- **Scope:** a single native iOS app (Swift + SwiftUI), adaptive for iPhone and iPad. No website.
- **Roadmap:** the CEO's **six phases** — interface → logins → Dynamics data → management dashboard → customer experience → quality of life.
- **Roles:** three login buckets — Management, Employee, Customer — with the specific job carried as a tag.
- **Dynamics:** **read-only.** Write-back is a candidate later stage, gated on a spike. See [ENGINEERING.md D12](docs/ENGINEERING.md#d12--dynamics-write-back-is-deferred-not-refused-new-in-v3).
- **A website stays possible later** — business rules live in the database rather than in the app, so a web client would inherit them. See [PRODUCT.md §9](docs/PRODUCT.md#9-the-website-question).
- **Team:** two part-time engineers, ~25 hours a week combined. Every date derives from that number.
- **Trial:** one warehouse, starting **2027-02-08**.
- **Launch:** all four warehouses, **2027-03-01**.

### Open decisions

| Decision | Owner | Due |
|---|---|---|
| Accept or reject the workforce measurement policy, in writing | CEO | **2026-09-25** |
| Swift vs. React Native — settled by a working spike, not a debate | Engineering | **2026-09-25** |
| Personal phones or company phones | CEO | 2026-10-16 |
| Walkthrough of the real purchasing and receiving process | CEO | 2026-12-04 |
| Who builds the product catalog, and by when | CEO | 2027-03-01 |
| Whether to add a website, and when | CEO | mid-2027 |

---

## History

**v3.0 (2026-09-14)** — A fuller CEO brief arrived. The roadmap was rebuilt on his six phases (launch moved 2027-02-22 → 2027-03-01; the week is accounted for in [TIMELINE.md §3](docs/TIMELINE.md#3-what-the-phase-order-costs)), six roles collapsed to three login buckets with job tags, Dynamics write-back was deferred rather than refused outright, the current-state workflow was documented for the first time, and a request for computer-vision worker monitoring was recorded and declined.

**v2.0 (2026-09-09)** — Scope narrowed from a web application plus an iOS app sharing a TypeScript backend, down to iOS only. The entire set was rewritten. The originals are in git history.
