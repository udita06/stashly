# API Specification

## A Note on "Endpoints" in This Architecture

As described in `ARCHITECTURE.md`, this app uses Firebase Firestore's client SDK directly from the browser rather than a custom REST backend — there is no Express/Node server handling HTTP routes. However, the app still has a clear set of **logical operations** the frontend performs against the database, each gated by Firestore security rules instead of server-side route middleware.

To keep this planning-stage and gradeable in familiar terms, the table below describes these operations **as if they were REST endpoints** (method + route + auth + description). In the actual implementation, each row maps to a specific Firestore SDK call (e.g., a `GET /items` maps to a Firestore `getDocs`/`onSnapshot` query on the `items` collection) rather than an HTTP request to a server route. This mapping is intentional: it lets the planned operations be reviewed like a normal API spec now, while staying honest about the serverless implementation described in the architecture doc.

---

## Planned Endpoints (Logical Operations)

| Method | Route (logical) | Auth Required | Description |
|---|---|---|---|
| `GET` | `/items` | Yes (any signed-in member) | Fetch the full item catalog with live status (available/borrowed) and owner info. |
| `GET` | `/items/:id` | Yes | Fetch details for a single item, including current loan status if borrowed. |
| `POST` | `/items` | Yes (admin only) | Add a new item to the catalog (club-owned or individually-owned). |
| `PATCH` | `/items/:id` | Yes (admin, or the item's individual owner) | Edit item details (condition, notes, photo) — not status, which is only changed via loans. |
| `DELETE` | `/items/:id` | Yes (admin only) | Remove an item from the catalog (e.g., permanently lost or retired). |
| `POST` | `/loans` | Yes (any signed-in member) | Create a new loan — the "I'm taking this" borrow action. Fails if the item is already borrowed. |
| `PATCH` | `/loans/:id/return` | Yes (only the borrower on that loan, or an admin) | Close a loan — the "I'm returning this" action. Sets item status back to available. |
| `GET` | `/loans?user=:id` | Yes (self, or admin viewing any user) | Fetch a user's current and past loans — powers the "My Borrows" view. |
| `GET` | `/items?owner=:id` | Yes (self, or admin) | Fetch items owned by a specific user — powers the "My Items" view for individual owners. |
| `GET` | `/loans?item=:id` | Yes | Fetch the loan history for a specific item (who has borrowed it and when). |
| `GET` | `/users/me` | Yes | Fetch the signed-in user's own profile (name, role, email). |

No endpoint is unauthenticated — every operation requires a signed-in user, since even browsing the catalog is club-internal, not public.

---

## Auth Strategy

**Provider:** Firebase Authentication, using **Google Sign-In**.

**Why:** Nearly every student already has a Google account (typically their college email), so there's no new password or credential to create or forget. Firebase Auth issues a signed identity token that Firestore security rules can check directly — so "auth required" in the table above is enforced at the database layer, not by a custom server checking session cookies or JWTs.

**How it works in practice:**
1. A student signs in once with their Google account through the React app.
2. Firebase Auth returns an identity token, which the Firebase SDK automatically attaches to all subsequent Firestore requests.
3. Firestore security rules check this token to determine: (a) is this user signed in at all, and (b) for admin-only operations, does this user's `role` field (stored on their `users` document) equal `admin`.
4. There are no separate API keys or manual token management required on the frontend — the Firebase SDK handles this once sign-in has happened.

**Roles:**
- `member` — default role for any signed-in user; can browse, borrow, return, and add/edit their own individually-owned items.
- `admin` — club's inventory lead(s); can additionally add/edit/delete any item and view any member's loan history. (Assigned manually in Firestore for MVP — no self-serve admin promotion, to avoid building an approval workflow that's explicitly out of scope per the PRD.)

---

## Error Response Shape

Since there's no custom server, "errors" here means how the frontend standardizes handling of Firestore/Auth SDK errors before showing anything to the user. All errors caught in the app are normalized into a single internal shape before being displayed:

```json
{
  "error": true,
  "code": "PERMISSION_DENIED",
  "message": "You don't have permission to do that.",
  "context": "loans.create"
}
```

- **`code`** — a short, consistent string (e.g., `PERMISSION_DENIED`, `NOT_FOUND`, `ALREADY_BORROWED`, `NETWORK_ERROR`, `UNKNOWN`) mapped from whatever Firestore/Auth SDK error was thrown, so the UI can react consistently regardless of the underlying SDK's own error format.
- **`message`** — a short, human-readable string safe to show directly to the user (no raw SDK/stack trace text ever reaches the UI).
- **`context`** — which operation failed (e.g., `loans.create`, `items.update`), used for debugging and logging, not shown to the user.

**Special case — "already borrowed":** Because two users could theoretically try to borrow the same item at nearly the same moment, the `POST /loans` operation is implemented as a Firestore transaction that checks the item's current status before writing. If the item is already borrowed by the time the transaction runs, it fails with `code: "ALREADY_BORROWED"` and the UI shows a clear message ("Someone just took this — try refreshing") rather than silently creating a conflicting loan.
