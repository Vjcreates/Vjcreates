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

Delegate via the Agent tool using `subagent_type: general-purpose`. In the delegation prompt, explicitly instruct the subagent to:
1. Read `.claude/agents/reviewer.md` first
2. Follow its persona and instructions exactly — review the diff cold, check against AGENTS.md, output findings in the format it specifies

This is a deliberate workaround: Claude Code's `Agent` tool only accepts subagent types from its own fixed registry (`general-purpose`, `Explore`, `Plan`, etc.) — it does not natively discover custom agents defined in `.claude/agents/` by name. Passing `subagent_type: reviewer` directly will fail with "Agent type 'reviewer' not found." Using `general-purpose` and pointing it at the file achieves the same isolation (fresh context, no inherited reasoning) without relying on native discovery that doesn't currently work.

Pass the subagent:
- The diff (`git diff` against the base branch)
- The approved implementation plan (not just the original spec — the plan is the actual source of truth for what should have been built)

Do **not** pass your own reasoning, notes, or explanation of why you made particular implementation choices — the subagent should judge the diff against the plan on its own, with a fresh context, the same way an independent reviewer would.

## Step 3 — Fixer (you, same context as Coder)

Evaluate each of the Reviewer's findings — don't apply them automatically. You have context the Reviewer doesn't (your own implementation reasoning), so you're better positioned to judge whether a finding is right.

For each finding:
- **Agree, and it's a real gap** → fix it with a local commit
- **The diff deviates from the approved plan** → this is always a real finding, regardless of anything said in conversation afterward. A verbal request mid-session ("change X to Y") is not the same as re-approving the plan — the plan is the source of truth, and only Step 4's explicit approval can change it. Do not reject a finding on the grounds that "the user asked for this in chat." If a deviation happened, fix it back to the plan, or — if you believe the deviation is actually correct — surface it as an open item at Step 4 so the human can explicitly approve the plan change itself, rather than treating an in-conversation request as sufficient.
- **Disagree, because the Reviewer lacked context it needed** (e.g. it flagged something AGENTS.md already covers, or a constraint that isn't visible from the diff + requirement alone) — and the diff still matches the approved plan → don't apply the fix. Note in your own words why the finding doesn't hold, so it's visible at the approval gate.
- **Genuinely unclear, or a real judgment call either way** → don't silently pick a side. Leave it as an open item for the human to weigh in on at Step 4, rather than the Fixer deciding unilaterally.

Never treat "the Reviewer said so" as sufficient justification on its own — the same way the Reviewer doesn't treat "the Coder wrote it that way" as sufficient, and doesn't treat "the user said so in conversation" as equivalent to plan approval. All three need an actual reason tied back to the approved plan.

**Loop Step 2 → Step 3 up to 3 rounds.** If not clean after 3 rounds, stop looping and move to Step 4 anyway, flagging the unresolved items for the human to weigh in on.

## Step 4 — Approval gate (required, no exceptions)

**Do not open a PR until the user explicitly approves.** Present:
- A summary of the change
- The diff (or a link/way to view it)
- Fixes applied, findings rejected (with your reasoning for each), and any genuinely unclear items left for the human to decide

Ask directly: **"Ready to open a PR for this?"** Wait for an explicit yes. A lack of response, or anything short of clear approval, means stop — do not open the PR speculatively or "to save time."

## Step 5 — Prepare the PR link (only after approval)

Never create the PR yourself, even if a GitHub connector or the `gh` CLI is available. The human always clicks Create. Instead:
- Push the branch.
- Hand the human a prefilled GitHub compare link: `https://github.com/<owner>/<repo>/compare/<base>...<branch>?expand=1&title=<title>&body=<description>`, with the title and description URL-encoded. Build it from the repo's actual `origin` remote, never a guessed owner or repo.
- If a PR may already exist for the branch, say so. GitHub shows the existing PR on that page instead of a create form.

The title and description follow these rules:
- **If a ticket key was passed:** PR title is `<TICKET-KEY>: <short description>`
- **If "no-ticket" was passed:** PR title is a clear plain-language description only
- If a ticket exists, note in the PR description that merging should transition the ticket (handled by the standard GitHub-Jira integration, not this skill)
- Include screenshots, per AGENTS.md: capture them from the built branch running in a browser, not mockups. A link can't carry images, so give the description a Screenshots section with one labeled line per image, save the files, and tell the human where they are so they can drag them in before clicking Create.

## Step 6 — Hand off

Do not merge. Once the human creates the PR, leave it for engineering's review and merge — this step is never automated.

## Report back

Report to the main session:
- The prefilled PR link and where the screenshot files are, or that it's still waiting on approval
- Fixes applied, findings rejected (with reasoning), and any open items left for the human
- Number of Fixer rounds it took
