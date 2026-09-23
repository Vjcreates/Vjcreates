---
name: build-execute
description: Runs the Coder, Reviewer, and Fixer loop for an already-analyzed, approved spec. Invoked by /build, not directly by users.
disable-model-invocation: true
context: fork
background: true
---

## Team conventions

!`cat AGENTS.md`

## Input

Ticket key (or "no-ticket") and the approved implementation plan: $ARGUMENTS

## Step 1 — Coder

Build the change on a local branch, following the approved plan — **do not open a PR yet**:
- Pull the linked Figma frame if the ticket references one
- Follow the team's tech-stack conventions (see AGENTS.md)
- Build against the plan as approved. If something in the plan turns out to be wrong or unworkable once you're implementing it, don't silently deviate — flag the deviation and your reasoning so it's visible at the approval gate.
- Commit the change locally

## Step 2 — Reviewer (delegated, independent)

Delegate to the **reviewer** subagent via the Agent tool. Pass it:
- The diff (`git diff` against the base branch)
- The approved implementation plan (not just the original spec — the plan is the actual source of truth for what should have been built)

Do **not** pass your own reasoning, notes, or explanation of why you made particular implementation choices — it should judge the diff against the plan on its own, with a fresh context, the same way an independent reviewer would.

## Step 3 — Fixer (you, same context as Coder)

Evaluate each of the Reviewer's findings — don't apply them automatically. You have context the Reviewer doesn't (your own implementation reasoning), so you're better positioned to judge whether a finding is right.

For each finding:
- **Agree, and it's a real gap** → fix it with a local commit
- **Disagree, because the Reviewer lacked context it needed** (e.g. it flagged something AGENTS.md already covers, or a constraint that isn't visible from the diff + requirement alone) → don't apply the fix. Note in your own words why the finding doesn't hold, so it's visible at the approval gate.
- **Genuinely unclear, or a real judgment call either way** → don't silently pick a side. Leave it as an open item for the human to weigh in on at Step 4, rather than the Fixer deciding unilaterally.

Never treat "the Reviewer said so" as sufficient justification on its own — the same way the Reviewer doesn't treat "the Coder wrote it that way" as sufficient. Both directions need an actual reason.

**Loop Step 2 → Step 3 up to 3 rounds.** If not clean after 3 rounds, stop looping and move to Step 4 anyway, flagging the unresolved items for the human to weigh in on.

## Step 4 — Approval gate (required, no exceptions)

**Do not open a PR until the user explicitly approves.** Present:
- A summary of the change
- The diff (or a link/way to view it)
- Fixes applied, findings rejected (with your reasoning for each), and any genuinely unclear items left for the human to decide

Ask directly: **"Ready to open a PR for this?"** Wait for an explicit yes. A lack of response, or anything short of clear approval, means stop — do not open the PR speculatively or "to save time."

## Step 5 — Open the PR (only after approval)

- **If a ticket key was passed:** PR title is `<TICKET-KEY>: <short description>`
- **If "no-ticket" was passed:** PR title is a clear plain-language description only
- If a ticket exists, note in the PR description that merging should transition the ticket (handled by the standard GitHub-Jira integration, not this skill)

## Step 6 — Hand off

Do not merge. Leave the PR ready for engineering's review and merge — this step is never automated.

## Report back

Report to the main session:
- Whether a PR was opened (and its link), or whether it's still waiting on approval
- Fixes applied, findings rejected (with reasoning), and any open items left for the human
- Number of Fixer rounds it took
