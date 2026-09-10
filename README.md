# OpsLink

Planning documents for **OpsLink** — an operations app for distribution warehouses.

One iOS app that changes shape depending on who opens it. Warehouse employees see today's work on an iPhone and check it off, even with no signal. Managers build tomorrow's list on an iPad and watch it get done. The CEO sees all four warehouses at once.

**First customer:** Goldberry Distributors, a food distributor with four warehouses across the California Bay Area and Sacramento.

---

## Start here

| If you are… | Read this |
|---|---|
| **The CEO, or anyone non-technical** | **[docs/FOR-THE-CEO.md](docs/FOR-THE-CEO.md)** — the whole project in plain English, 8 minutes |
| Reviewing what we measure about employees | **[docs/WORKFORCE-POLICY.md](docs/WORKFORCE-POLICY.md)** — short, and needs a decision |
| Looking for the feature list | [docs/PRODUCT.md](docs/PRODUCT.md) |
| Looking for dates | [docs/TIMELINE.md](docs/TIMELINE.md) |
| An engineer | [docs/ENGINEERING.md](docs/ENGINEERING.md) |

---

## The documents

| Document | What's in it |
|---|---|
| [FOR-THE-CEO.md](docs/FOR-THE-CEO.md) | Plain-English overview. No jargon. The one to read first. |
| [WORKFORCE-POLICY.md](docs/WORKFORCE-POLICY.md) | What the app measures about people and what it refuses to — and why. **Awaiting the CEO's written decision.** |
| [PRODUCT.md](docs/PRODUCT.md) | Roles, features, release scope, success criteria, open questions |
| [ENGINEERING.md](docs/ENGINEERING.md) | Architecture, data model, security, offline design, and every major decision with its cost |
| [TIMELINE.md](docs/TIMELINE.md) | Sprint-by-sprint schedule, the capacity arithmetic, and what gets cut first |

---

## Where things stand

**Planning only. No code has been written yet.**

- **Scope:** a single native iOS app (Swift + SwiftUI), adaptive for iPhone and iPad. No website.
- **A website stays possible later** — all business rules live in the database rather than in the app, so a web client would inherit them rather than duplicate them. See [PRODUCT.md §9](docs/PRODUCT.md#9-the-website-question).
- **Team:** two part-time engineers, ~25 hours a week combined. Every date derives from that number.
- **Trial:** one warehouse, starting **2027-02-01**.
- **Launch:** all four warehouses, **2027-02-22**.

### Open decisions

| Decision | Owner | Due |
|---|---|---|
| Accept or reject the workforce measurement policy, in writing | CEO | **2026-09-25** |
| Swift vs. React Native — settled by a working spike, not a debate | Engineering | **2026-09-25** |
| Personal phones or company phones | CEO | 2026-10-16 |
| Whether to add a website, and when | CEO | mid-2027 |

---

## History

Version 1.0 of these documents described a web application and an iOS app sharing a TypeScript backend. Scope was narrowed to iOS only, and the entire set was rewritten. The originals are in git history.
