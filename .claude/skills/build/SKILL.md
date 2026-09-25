---
name: build
description: Intake and analyze a requirement — from a Jira ticket key or pasted spec text. Entry point for the team's automated ticket pipeline.
disable-model-invocation: true
---

## Team conventions

!`cat AGENTS.md`

## Input

$ARGUMENTS

## Step 1 — Determine the source

If the input looks like a Jira issue key (e.g. `APP-1234` or `DESIGN-567`):
- Fetch that ticket directly.
- Proceed to Step 3 (Analyzer).

If the input is pasted spec text instead (no recognizable ticket key):
- Ask the user: **"Create a Jira ticket for this, or proceed without one?"**
- If **no**: proceed to Step 3 (Analyzer) using the pasted text as-is. No ticket exists — remember this for Step 4 (PR title has no ticket key).
- If **yes**: ask **"APP or DESIGN project?"**
  - **APP** → create the ticket following the APP four-section structure in AGENTS.md (User Story / Prototype / Required / Acceptance Criteria), including `customfield_11608`.
  - **DESIGN** → create the ticket following the DESIGN two-section structure in AGENTS.md (Goal / Required).
  - Proceed to Step 3 (Analyzer) using the newly created ticket.

## Step 2 — Never assume

Do not guess project conventions if a field or structure isn't covered in AGENTS.md above. Stop and ask the user rather than inventing a format.

## Step 3 — Analyzer

Check the spec (ticket or pasted text) against AGENTS.md conventions:
- Correct section structure for its project (APP vs. DESIGN — or "no ticket" plain text)
- No scope ambiguity (see "Known Product Decisions" in AGENTS.md for prior calls that shouldn't be relitigated)
- Acceptance Criteria present and testable, if applicable
- Any conflicts with existing product decisions documented in AGENTS.md

**If gaps exist:** report them clearly to the user and stop. Do not proceed to build on a thin or ambiguous spec.

**If the spec is solid:** write a detailed implementation plan — not a 2-3 sentence summary. Include:
- Which files/areas of the codebase will change, and why
- Each Required bullet / Acceptance Criterion mapped to how it will be implemented
- **Conventional UI elements the spec didn't explicitly mention but a page of this kind would normally have** (e.g. a heading, standard spacing, expected labels) — include these as stated assumptions in the plan rather than omitting them for not being spec'd. Don't build only the literal letter of the spec if it would produce a page missing obvious, expected elements.
- **Consistency with existing pages already in the repo (main branch)** — check how other pages handle similar elements (headings, error-message style, layout conventions, terminology, spacing) and either follow that precedent explicitly, or flag it if this page has a reason to diverge. Always consider the overall user experience, not just the literal requirement text — note this in the plan so it's visible before approval, not discovered later.
- Any other assumptions being made where the spec is silent
- Edge cases and how they'll be handled
- What's explicitly out of scope, if that's ambiguous from the spec

Present the full plan to the user and ask: **"Approve this plan to proceed to build?"** Wait for an explicit yes. Do not treat silence, a partial response, or a vague acknowledgment as approval. If the user requests changes to the plan, revise and re-present it — don't proceed on a plan that was edited but not re-confirmed.

**Only after explicit plan approval**, hand off to the build-execute skill:

```
/build-execute <ticket-key-or-"no-ticket"> <the approved plan>
```

Pass the full approved plan, not just a one-line summary — the Coder should build against the plan you both agreed on, not reinterpret the original spec from scratch.

This runs the Coder → Reviewer → Fixer loop as a background subagent, keeping this conversation free for other work while it runs.
