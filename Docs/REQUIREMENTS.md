# Requirements

## Functional Requirements

1. A signed-in user must be able to view a list of all catalog items, each showing name, category, owner, and current status (available or borrowed).
2. A signed-in user must be able to view the details of a single item, including its condition/notes and, if borrowed, who currently has it and since when.
3. An admin must be able to add a new item to the catalog, specifying name, category, owner (club or a specific student), condition/notes, and an optional photo.
4. An admin, or the individual owner of an item, must be able to edit that item's condition/notes and photo.
5. An admin must be able to remove an item from the catalog.
6. A signed-in user must be able to mark an available item as borrowed by themselves in one action ("I'm taking this"), which records their identity and a timestamp.
7. The system must prevent an item from being marked borrowed if it is already borrowed by someone else.
8. A signed-in user must be able to mark an item they currently have borrowed as returned in one action ("I'm returning this"), which closes the loan and makes the item available again.
9. A signed-in user must be able to view a "My Borrows" list showing every item they currently have out, and their past return history.
10. A signed-in user who owns individual items must be able to view a "My Items" list showing what they own and who (if anyone) currently has each item.
11. The system must send or display a reminder when an item has been borrowed for longer than a configurable threshold (default: 7 days).
12. A user must be able to sign in using their Google account, with no separate password to create or remember.
13. The system must distinguish between two roles — member and admin — and restrict item-management actions (add/edit/delete) to admins, except individually-owned items, which their owner can also edit.
14. The system must display, for every item, whether it is club-owned or owned by a specific individual student.
15. The system must show a clear, human-readable error message (not a raw technical error) whenever an action fails — e.g., trying to borrow an item that was just taken by someone else.

---

## Non-Functional Requirements

**Performance**
- The borrow and return actions must complete and reflect in the UI within a median of ~15 seconds end-to-end, matching the "as fast as texting" design goal from the PRD — this is the single most important performance commitment, since high friction directly undermines adoption.
- The catalog view must load within a few seconds on a typical mobile data connection (college Wi-Fi or 4G), since most usage will happen on phones, not laptops.

**Security**
- No item or loan data is accessible without signing in — there is no public/unauthenticated view of the catalog.
- Item-management actions (add/edit/delete) are restricted to admins at the database rule level (Firestore security rules), not just hidden in the UI — a non-admin user must not be able to perform these actions even by calling the underlying database operation directly.
- A user can only create a loan under their own identity and can only mark a return on a loan they are the borrower of (or as an admin) — this is enforced the same way, at the security-rule level, not just in the interface.
- No passwords are stored or managed by the app — all authentication is delegated to Google via Firebase Auth.

**Accessibility**
- All interactive elements (borrow/return buttons, forms) must be usable via touch on a small mobile screen, since most real usage will happen mid-activity on a phone, not a desktop.
- Text and buttons must maintain sufficient color contrast and legible font sizes by default, using standard accessible component patterns rather than custom low-contrast styling — this is a baseline commitment appropriate for a first project, not a full WCAG audit.
- The app must remain usable with a slow or intermittent connection — actions should clearly show a loading/pending state rather than appearing to silently fail.

---

## Assumptions and Constraints

**Time budget:** This is a solo, first-time project built alongside a full course load and multiple club commitments (coding, dev, dance, robotics). Given the compressed 1-week-per-level schedule (Ronin through Shogun, Sept 17 – Oct 14, 2026), the realistic weekly time budget is limited — the plan assumes focused, part-time effort (evenings/weekends) rather than full-time availability, and MVP scope has been deliberately kept small (see PRD's Out of Scope section) specifically to fit this constraint.

**Tools already known vs. to be learned:**
- This is the author's first real project — no prior hands-on experience with React, Firebase, or building/deploying a web app end-to-end.
- Basic programming fundamentals are assumed (from BTech ECE coursework and coding club exposure), but not framework-specific experience.
- React and Firebase were deliberately chosen (see `ARCHITECTURE.md`) specifically because they are widely taught with strong beginner documentation and community support (including within the author's own coding/dev clubs) — the learning curve is a known, accepted constraint, not an oversight.
- Given the 1-week Kenshi window, learning is scoped narrowly to only what the MVP's six core features require — not general mastery of either tool.

**Other constraints shaping scope:**
- No budget for paid infrastructure — the architecture relies entirely on Firebase's free tier, which comfortably supports a single club's usage (well under free-tier limits for a ~15–40 member club).
- No dedicated design or QA support — this is a solo build, so functional and non-functional requirements above are intentionally kept realistic for one person to implement and verify, not enterprise-grade.
- Real users (the author's own robotics club, and potentially other clubs the author has access to) are assumed to be available for feedback throughout Kenshi–Shogun, which is what makes the 25-user success metric in the PRD achievable rather than aspirational.
