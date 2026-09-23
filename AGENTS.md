# AGENTS.md — Team Conventions for the Automated Ticket Pipeline

This file is read by every stage of the pipeline (Analyzer, Coder, Reviewer, Fixer) before doing any work. It replaces tribal knowledge that used to live only in individual chats — keep it updated as conventions change.

---

## Jira Conventions

### APP Project
- Every story follows a strict **four-section structure**: `User Story`, `Prototype`, `Required`, `Acceptance Criteria`
- User Story always begins: **"As a user,"**
- Every ticket created in APP must include the custom field `customfield_11608` with value `{'id': '207'}`
- Acceptance Criteria uses `Given / When / Then` format
- Writing style: **short, direct, easy to read** — no filler, no unnecessary hedging

### DESIGN Project
- Tickets follow a **two-section structure**: `Goal`, `Required`
- Does not use APP's four-section format (`User Story`/`Prototype`/`Required`/`Acceptance Criteria`) — do not apply APP conventions here

### General Rules
- Do not update or transition a Jira ticket other than the one explicitly being worked on, without asking first
- PR titles must include the ticket key when one exists, e.g. `APP-2200: MMS drag-and-drop — Windows`
- When no ticket exists (pasted-spec path with no ticket created), PR titles use a clear plain-language description instead

---

## Known Product Decisions

*(Intentionally left blank for now — add team decisions here before committing this file publicly or sharing broadly. Referenced by `build/SKILL.md` and `reviewer.md`.)*

---

## Design System

*(Pending — not yet built. Once it exists in the repo, add its location here, e.g. `/design-system/` or a components library path, so the Reviewer can check diffs against it for consistency. Until this is filled in, Reviewer skips the design-system check entirely.)*

---

## Address & Contact Fields
- "Address" (combined Line 1 + Line 2) is standard convention when City/State/Postal Code/Country remain separate fields — do not fold city/state into a single "Address" field.
- Address Line 1 and Line 2 should be **stored separately** even when displayed together — display-only concatenation, never merge at the data layer, especially for anything E911-adjacent.

---

## Ticket Format Preferences
- Default to `Given / When / Then` for Acceptance Criteria unless explicitly told to write plain descriptive requirements instead
- Keep Required/AC bullets short and direct — avoid restating the same point across multiple bullets
- Flag open questions and ambiguities explicitly at the end of a ticket rather than silently picking an assumption

---

*Last updated: reflects conventions established through project chat history. Update this file whenever a new team-wide decision or convention is made — don't let it go stale.*
