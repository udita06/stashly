# Architecture Document

## System Diagram

```
                      ┌─────────────────────────┐
                    │        Browser             │
                    │  (Student's phone/laptop)  │
                    │                            │
                    │   React Web App (UI)       │
                    │  - Item catalog view       │
                    │  - Borrow / Return buttons │
                    │  - My Items / My Borrows   │
                    └───────────┬────────────────┘
                                │
                                │  HTTPS (Firebase SDK calls)
                                │
                ┌───────────────┴────────────────┐
                │                                │
         ┌───────▼────────┐               ┌─────────▼─────────┐
        │  Firebase Auth   │             │     Firestore       │
        │ (Google Sign-In) │◄───────────►│  (NoSQL Database)   │
        │                  │  verifies   │                     │
        │  Confirms who    │  requests   │  Collections:       │
        │  the user is     │             │  - users            │
        └──────────────────┘             │  - items            │
                                         │  - loans            │
                                         └─────────────────────┘
                                                        │
                                                        │  real-time listener
                                                        │  (auto-updates UI
                                                        │   when data changes)
                                                        ▼
                                            ┌─────────────────────────┐
                                            │   Browser (React App)    │
                                            │   UI updates live        │
                                            │   without manual refresh │
                                            └──────────────────────────┘

        ┌─────────────────────────────────────────┐
        │           Firebase Hosting              │
        │  Serves the built React app as a        │
        │  static site, reachable from any phone  │
        │  browser via a public URL               │
        └─────────────────────────────────────────┘
```

The app is a client-heavy architecture: there is no custom backend server. The React app talks directly to Firebase's managed services (Auth + Firestore) using Firebase's client SDK, secured by Firestore security rules instead of custom API middleware. This is a deliberate beginner-appropriate choice, explained below.

---

## Planned Tech Stack

| Layer | Choice | Why |
|---|---|---|
| **Frontend framework** | React | Most widely taught framework in college dev/coding clubs, has the largest volume of beginner tutorials and community support, and is the framework I'm most likely to get help with from clubmates while learning. |
| **Database** | Firebase Firestore | A NoSQL document database that requires no schema migrations or server setup, has a generous free tier suitable for a 25-user club tool, and integrates directly with the frontend without needing a separate backend to write from scratch. |
| **Auth provider** | Firebase Authentication (Google Sign-In) | Nearly every student already has a Google account (usually their college email), so sign-in requires zero new credentials to remember, and Firebase Auth handles the security-sensitive parts (tokens, sessions) so I don't have to build auth logic myself as a first project. |
| **Hosting** | Firebase Hosting | Free, deploys directly from the same Firebase project as the database and auth (one dashboard, one set of docs to learn), and is built for exactly this kind of small static/React app with a public URL — no separate DevOps setup needed. |

**Why no separate backend server (no Node/Express layer):** As a first project, introducing a custom backend adds a second codebase, a second deployment target, and API design decisions before I've built anything at all. Firestore's client SDK + security rules lets the app enforce "who can read/write what" without hand-writing that logic in a server, which keeps the MVP buildable within the Kenshi timeframe. If a real backend becomes necessary later (e.g., for scheduled overdue-reminder jobs), that can be introduced at Samurai using Firebase Cloud Functions — a lighter step up than standing up a full server from scratch.

---

## Data Flow

**Example: a student borrows an item.**

1. Student opens the app in their phone browser (served as a static site via **Firebase Hosting**).
2. On first use, they sign in with **Firebase Auth** (Google Sign-In) — this happens once; the session persists after that.
3. The React app loads the current item catalog by reading the `items` collection from **Firestore**, filtered to show live status (available / borrowed).
4. Student finds the item and taps **"I'm taking this."** The React app writes a new document to the `loans` collection (item ID, borrower ID, timestamp, status: `active`), and updates the corresponding item's status field to `borrowed`.
5. Firestore confirms the write and, because the app uses a **real-time listener** (`onSnapshot`), every other open instance of the app (e.g., another club member checking the same item) sees the status flip to "borrowed" within moments — no manual refresh needed.
6. On return, the same flow runs in reverse: tapping **"I'm returning this"** updates the `loans` document (status: `returned`, return timestamp) and flips the item back to `available`.

All reads and writes go directly from the browser to Firestore over HTTPS using the Firebase SDK; Firestore security rules (not a custom server) enforce that a student can only mark items as borrowed under their own identity and can't, for example, forge a return on someone else's behalf.

---

## Key Entities / Rough Data Model

**Nouns the app cares about:**

- **User** — a club member. Has a name, email (from Google sign-in), and a role (`member` or `admin`).
- **Item** — a piece of equipment. Has a name, category, condition/notes, optional photo, an **owner** (see below), and a current status (`available` / `borrowed`).
- **Owner** — not a separate collection, but a field on Item describing who owns it: either the **Club** itself (a fixed value, since MVP supports one club) or a specific **User** (for individually-owned items lent to the club).
- **Loan** — a single borrow/return record. Links one **Item** to one **User** (the borrower), with a start timestamp, an optional return timestamp, and a status (`active` / `returned`).

**Relationships:**

- A **User** can own zero or more **Items** (if they've lent personal gear).
- A **User** can be the borrower on zero or more **Loans** (both active and past).
- An **Item** has exactly one owner (Club or a User) and, at any given time, at most one **active Loan**.
- A **Loan** always references exactly one **Item** and one borrowing **User**.

This keeps the MVP data model to three real entities (User, Item, Loan) with ownership modeled as a field rather than a fourth entity — enough to support every MVP feature (catalog, borrow/return, live status, my items/my borrows, overdue nudges) without over-building for features that are explicitly out of scope (multi-club support, financial tracking) until a later level.
