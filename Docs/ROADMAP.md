# Roadmap

Stashly's feature set is mapped below across the four levels of Journey to Mastery, with each level building a distinct layer of capability on top of what shipped before.

---

## Ronin (current level) — Product Definition

**Status:** In progress
**Target completion:** 1 week (Sept 17 – Sept 22, 2026)

No code. This level locks the problem, the MVP feature set, the architecture, and the plan for everything below. Everything in Kenshi through Shogun is built against what gets decided here — so this roadmap itself is a Ronin deliverable.

---

## Kenshi — Ship the Core Borrow/Return Loop

**What ships:** The smallest version of Stashly that a real club could actually use.

- Item catalog (add/view items with owner, category, condition, optional photo)
- Borrow flow ("I'm taking this" — one tap, logs borrower + timestamp)
- Return flow ("I'm returning this" — closes the loan)
- Live item status view (available / borrowed-by-whom-since-when)
- "My Items / My Borrows" view (per the ownership model — what I own vs. what I've borrowed)
- Basic Google Sign-In (member vs. admin role, admin assigned manually)

**What's deliberately still missing:** overdue reminders are logged as data but not yet actively pushed to users; no analytics; no damage/condition history beyond a static notes field.

**Why this is the right cut for Kenshi:** it's the minimum needed to actually replace the "ask around" process described in the PRD's problem statement — catalog + borrow/return + status visibility is the core loop everything else depends on. Nothing here requires infrastructure beyond what's in `ARCHITECTURE.md`.

**Rough timeline:** 1 week (Sept 23 – Sept 29, 2026) — first 2–3 days on React + Firebase basics needed for this specific build (since this is a first project), remaining days building and testing the core loop with real club members. Given the tight window, learning is scoped narrowly to only what the MVP features require, not general React/Firebase mastery.

---

## Samurai — Make It Reliable Enough to Trust Daily

**What's added on top of Kenshi:**

- **Overdue nudges** — the reminder system planned in the MVP feature list but deferred at Kenshi; becomes a real notification (in-app banner at minimum, possibly email via Firebase Cloud Functions) once an item has been held past a configurable threshold.
- **Loan history per item** — a visible log of past borrows for each item, not just its current status, so patterns (e.g., an item that's always late coming back) become visible.
- **Damage/condition logging** — a lightweight way to flag an item's condition changed during a loan (e.g., "returned with a cracked case"), addressing real wear-and-tear that a pure status tracker ignores.
- **Low-stock / duplicate-item awareness** — if multiple identical items exist (e.g., "3x ultrasonic sensor"), the system distinguishes individual units so status tracking stays accurate instead of collapsing them into one ambiguous entry.

**Why these and not more:** these four directly extend the trust problem from the PRD — once a club depends on the tool daily, accountability (who had it, what condition) and reliability (nudges that actually fire) matter more than new feature surface area. This is also the point where introducing Firebase Cloud Functions (a small serverless backend layer) becomes justified, as noted in `ARCHITECTURE.md`.

**Rough timeline:** 1 week (Sept 30 – Oct 6, 2026), assuming Kenshi's core loop is stable and already being used by real club members going into this level.

---

## Shogun — Scale Beyond One Club

**What's added on top of Samurai:**

- **Multi-club support** — the single hardcoded "Club" owner concept from the MVP data model becomes a real entity, letting Stashly serve more than one club's inventory (each with its own catalog, members, and admins) — this is the step that makes reaching 25+ real users across more than one community realistic.
- **Opt-in approval workflow** — for specific high-value items only (not the default flow), an item owner can require a quick approval before it's marked borrowed — added now, not at MVP, because it's a controlled exception to the friction-free design principle, not the default behavior.
- **Usage analytics** — most-borrowed items, average loan duration, which members are most active — useful once there's enough real usage data across multiple clubs to make patterns meaningful.
- **Trust/reliability signal** — a simple, visible indicator (e.g., "returns on time" ratio) for members who borrow frequently, surfaced to admins and individual owners deciding whether to lend something.

**Why these and not sooner:** multi-club support only makes sense once the single-club version has proven itself (Samurai), and analytics/trust signals need real usage history to be meaningful rather than speculative. This is also where the product's core differentiator — individually-owned item tracking — becomes most valuable, since trust between strangers across clubs matters more than trust within one tight-knit group.

**Rough timeline:** 1 week (Oct 7 – Oct 13, 2026) — multi-club support and analytics built in the first 3–4 days, remaining days spent onboarding a second club and pushing toward the 25-real-user bar.

---

## Summary Table

| Level   | Core Addition                                                              | Dates                   | Duration |
| ------- | -------------------------------------------------------------------------- | ----------------------- | -------- |
| Ronin   | Problem definition, architecture plan, no code                             | Sept 17 – Sept 22, 2026 | 1 week   |
| Kenshi  | Core borrow/return loop, single club, MVP feature set                      | Sept 23 – Sept 29, 2026 | 1 week   |
| Samurai | Overdue nudges, loan history, condition logging, per-unit tracking         | Sept 30 – Oct 6, 2026   | 1 week   |
| Shogun  | Multi-club support, opt-in approvals, analytics, trust signal, 25-user bar | Oct 7 – Oct 13, 2026    | 1 week   |

**Total program length:** ~1 month (Sept 17 – Oct 13, 2026).
