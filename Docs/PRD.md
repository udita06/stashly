# Product Requirements Document (PRD)

## Product Name (working title)
**Stashly** — a shared inventory and borrow/return tracker for college club gear, built to handle both club-owned and individually-owned items.

---

## Problem Statement

College clubs that work with physical equipment — robotics, coding/hardware, photography, dance (props/costumes), music (instruments) — routinely share gear across members. In practice, almost none of these clubs have a real system for tracking who has what. The default process is asking around in person or over WhatsApp: "does anyone have the multimeter?", "who took the ultrasonic sensor last week?"

This breaks down in three specific, observable ways:
1. **Person availability** — the person who has the item may not be reachable when someone else needs it or needs to confirm its whereabouts.
2. **Time lag** — even when someone does respond, resolving "who has X" through informal messaging takes minutes to hours, disrupting build sessions that often happen under time pressure (competition prep, project deadlines).
3. **Lost tracking over time** — with no log, items drift between members over weeks, and nobody — including the club's own inventory-conscious members — can say with confidence where a given item currently is, or who is actually responsible for it.

This is a first-hand, currently-experienced problem in the author's own robotics club, where inventory tracking is 100% informal (no spreadsheet, no log, no owner role in most cases) and all three failure modes above occur regularly. It generalizes naturally to other clubs with shared physical resources, because the underlying cause — no shared source of truth, high friction to log anything — is structural, not specific to one club's habits.

A second, less obvious version of this problem makes it worse: **not everything shared within a club is actually club property.** Members frequently lend personal gear — a senior's leftover sensor kit, someone's own soldering iron or multimeter — because the club doesn't own enough equipment to go around. These individually-owned items get tracked even less carefully than club-owned ones, even though losing or damaging them matters more to the person who owns them. No existing informal process (or generic inventory tool) distinguishes "club asset" from "personal item on loan," which means the person who cares most about an item's safe return has the least visibility into where it is.

---

## Target User

**Primary user:** Active members of college technical or hobby clubs (robotics, coding/hardware, photography, dance, music) — typically second- and third-year students — who regularly borrow or lend physical equipment as part of club activities, project work, or event/competition prep.

**Two specific sub-roles within this group:**
- **Borrowers** — any club member who needs to use a shared item temporarily and currently has no reliable way to check its status before asking around.
- **Item owners** — either the club itself (as an entity, for club-purchased equipment) or an individual student who has made their personally-owned item available to other members. Individual owners specifically need visibility into who currently has their item and for how long.

This is deliberately not "students" or "club members" in general — it is students in clubs where **physical, shared, borrowable equipment** is a routine part of activity, and where borrowing currently happens through informal, unlogged communication.

---

## Core Features for MVP

The smallest version that is actually useful — this is the feature list Kenshi will be graded against.

1. **Item catalog** — Add an item with name, category, owner (club or a specific student), condition/notes, and an optional photo. Forms the single source of truth for "what exists."
2. **Borrow flow** — A fast, low-friction action ("I'm taking this") that logs the borrower and timestamp against the item. Designed to take seconds, not require a form to be filled out mid-build.
3. **Return flow** — An equally fast counterpart action ("I'm returning this") that closes the open loan and marks the item available again.
4. **Live item status view** — For any item, show at a glance: available, or borrowed (by whom, since when).
5. **"My items / my borrows" view** — Each student can see (a) items they personally own and who currently holds them, and (b) items they currently have borrowed from others.
6. **Overdue nudges** — A simple, configurable reminder (e.g., after 7 days) when an item has been held longer than expected — particularly important for individually-owned items, where the real owner has a stake in getting it back.

These six features directly address the three problems named above: the catalog and status view solve "lost tracking," the borrow/return flow solves "time lag" (by replacing a conversation with a lookup), and the owner model plus overdue nudges solve the person-availability and accountability gap — especially for individually-owned items, which is this product's key differentiator from a generic inventory tool.

---

## Out of Scope (for MVP, and why)

- **Purchasing / procurement workflows** — Not the problem being solved; clubs already handle purchasing decisions separately, and adding it would dilute the MVP's focus on tracking what already exists.
- **Multi-club / multi-chapter support** — MVP targets a single club's inventory. Supporting multiple clubs adds real complexity (permissions, cross-club item visibility) with no MVP-stage benefit; this is explicitly planned for a later level (Shogun), not now.
- **Financial tracking** (item value, depreciation, damage cost estimation) — Adds bookkeeping complexity unrelated to the core "who has what" problem. Clubs that need this already use other tools (or spreadsheets) for finances.
- **Approval workflows** (e.g., admin sign-off required before borrowing) — Deliberately excluded because approval steps add friction, and friction is the exact thing this product is trying to remove. If added later, it would be opt-in per item (e.g., for genuinely expensive gear), not a default.
- **Custom QR/barcode hardware integration** — MVP will support simple scannable codes (e.g., a printed QR image per item/bin) at most, but not custom hardware, scanners, or physical infrastructure. Keeps the MVP buildable without physical deployment dependencies.

---

## Success Metrics

The product is working if it measurably replaces the informal ask-around process with actual logged usage — not just installs.

- **Adoption:** At least 25 real users (club members) actively using the tool by Shogun, consistent with the program's user bar. "Active" means they have completed at least one real borrow or return action, not just created an account.
- **Engagement depth:** At least 70% of a club's tracked items have at least one logged borrow/return cycle within a month of the club onboarding — evidence the tool is actually replacing the old process, not sitting unused alongside it.
- **Friction proof:** Median time to complete a borrow or return action stays under ~15 seconds in real usage (self-measured via timestamps between action start and confirmation), validating the "as fast as texting" design goal.
- **Reduced "who has this" incidents:** A qualitative/self-reported check-in with club members (e.g., a short survey at Samurai and Shogun) showing a clear reduction in "does anyone know where X is" moments compared to before the tool was introduced.
- **Owner trust signal:** For individually-owned items specifically, owners report (via the same check-in) that they have better visibility into their own lent items than before — this is the feature that differentiates the product, so it needs its own success signal, not just aggregate usage numbers.
