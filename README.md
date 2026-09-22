# Stashly

> A shared inventory and borrow/return tracker for college club gear — for clubs where nobody can say with confidence who has what.

---

## The Idea

College clubs with shared physical equipment — robotics, hardware, photography, dance, music — track who has what through informal WhatsApp messages and asking around in person, which breaks down under time pressure and loses track of items over weeks. Stashly replaces that with a fast, logged borrow/return system so any member can check an item's status in seconds instead of starting a group chat. It's built for both club-owned gear and individually-owned items members lend to the club (a senior's spare sensor kit, someone's own soldering iron) — a distinction most informal processes and generic inventory tools miss entirely, even though the actual owner has the most at stake in getting their item back. The problem is first-hand and current: it's happening right now in the author's own robotics club, with zero existing tracking system.

---

## Sketch

<!-- Excalidraw or Miro only. Link the live board AND embed/link a static export as backup. -->

![Sketch](./docs/sketch.png)

[View live board (Excalidraw / Miro)](https://excalidraw.com/#json=rnyHay_yHqEtFcERw0zH2,E7Uj5Gjlof8CrTsHHWG2Ng)

---

## Documents

- [Product Requirements](./docs/PRD.md)
- [Architecture](./docs/ARCHITECTURE.md)
- [API Spec](./docs/API_SPEC.md)
- [Roadmap](./docs/ROADMAP.md)
- [Requirements](./docs/REQUIREMENTS.md)

---

## Planned Stack

| Layer | Technology | Why |
|---|---|---|
| Framework | React | Most widely taught framework in college dev/coding clubs, with the largest volume of beginner tutorials and community support to draw on while learning. |
| Database | Firebase Firestore | Schema-less NoSQL database with no server setup, a free tier that comfortably covers a single club's usage, and direct integration with the frontend. |
| Auth | Firebase Authentication (Google Sign-In) | Nearly every student already has a Google account, so there's no new password to create, and Firebase handles tokens/sessions without custom auth logic. |
| Hosting | Firebase Hosting | Free, deploys from the same Firebase project as the database and auth, and is built for exactly this kind of small static/React app. |

---

## What I'm Building Toward

**Kenshi (frontend):** The core borrow/return loop working end-to-end for a single club — an item catalog (name, category, owner, condition, optional photo), a one-tap "I'm taking this" / "I'm returning this" flow, a live status view per item, a "My Items / My Borrows" view, and Google Sign-In with member/admin roles. Overdue reminders exist as logged data but aren't actively pushed yet. This is the minimum that actually replaces the "ask around" process described in the PRD.

**Samurai (full-stack):** Overdue nudges become real notifications once an item's been held past a configurable threshold, each item gets a visible loan history instead of just a current status, condition/damage logging captures wear-and-tear on return, and duplicate items (e.g., "3x ultrasonic sensor") are tracked as distinct units instead of collapsing into one ambiguous entry. This is also where Firebase Cloud Functions get introduced as a light serverless backend layer to support the reminder system.

**Shogun (production):** The single hardcoded "Club" owner becomes a real entity so Stashly can serve more than one club, each with its own catalog, members, and admins — the step that makes the 25-real-user bar realistic across more than one community. An opt-in approval workflow is added for specific high-value items only (never the default), plus usage analytics (most-borrowed items, average loan duration) and a simple "returns on time" trust signal — this is what "done and used by real people" looks like for this idea.

---

*Submitted to Journey to Mastery — Level 1: Ronin*
