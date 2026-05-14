---
description: Entry point for any change request. Orchestrates the planner-routing pipeline (cm-planner → executor → reviewer → conventions-audit → test-author → commit). Use for any work that touches code or docs.
---

# /change — Universal change orchestrator

Use this command for any work request: feature additions, bug fixes, refactors, renames, doc updates, schema changes, configuration edits.

## What this command does

`/change <user request>` triggers the full planner-routing pipeline:

```
user request
  → cm-planner (architectural authority — plans, surfaces ratifications)
  → [if STOP_FOR_HUMAN: wait for user decision]
  → executor (Edit, Write, Bash — implements per cm-planner's prompt)
  → git add (stage changes)
  → reviewer (review staged diff — HIGH/MEDIUM/LOW findings)
  → [if BLOCK COMMIT: address findings, re-stage, re-review]
  → conventions-audit (if docs/decisions/** touched OR sub-phase closing)
  → [if BLOCK CLOSURE: address findings or surface override block to user]
  → test-author (if monitored source paths added new behavior)
  → [if TESTS ADDED with failures: fix source bug in separate loop]
  → git commit
  → (loop back to executor if cm-planner prompt has more steps)
  → (when all done) cm-planner in "review commits for push" mode
  → [if BLOCK PUSH: amend / rebase flagged commits]
  → git push
```

## Trigger paraphrases

This command activates on ANY of:
- `/change <request>` (explicit)
- "implement X" / "add X" / "fix X" / "refactor X" / "rename X"
- "update the docs to X"
- "change how X works"
- "modify X"
- "remove X" / "delete X"

If user just asks a question without requesting work (e.g., "how does X work?", "what does this do?"), still run through cm-planner for routing — most questions still need planner involvement for the "lookup" workflow.

## Pipeline rules

### Rule 1 — Planner first, ALWAYS

Before responding to ANY user request in this repository, invoke cm-planner.

Exceptions:
- Direct Claude Code meta-commands (`/agents`, `/hooks`, `/plugin`)
- Pure conversational messages (greetings, acknowledgments)

For every other category — including one-line edits, lookups, and "what does X do" — go through cm-planner.

### Rule 2 — STOP_FOR_HUMAN is verbatim

If cm-planner (or any agent) returns a response starting with `STOP_FOR_HUMAN`:
- Surface the full message to the user verbatim
- Wait for user's decision
- Do not paraphrase, summarize, or strip the token

### Rule 3 — Specialist agents per cm-planner's prompt

The cm-planner's implementation prompt includes a "Specialist agents required" section. Invoke them in order:
1. Executor (you, the main agent)
2. Reviewer (always)
3. Conventions-audit (conditional)
4. Test-author (conditional)

Do not skip any specialist agent that cm-planner names.

### Rule 4 — Tripwire discipline

Announce every Bash, Edit, Write, Read tool call BEFORE invoking it. The user sees what's about to happen. This is enforced by hooks in `.claude/settings.json` and by your own discipline.

Silent operations = process failure. If you find yourself about to act silently, stop and announce first.

### Rule 5 — Atomic commits

Each commit is a single logical unit:
- One conceptual change
- All related files in the same commit
- Reviewer passed before commit
- Conventions-audit passed (if applicable) before commit
- Test-author passed (if applicable) before declaring complete

Do NOT chain unrelated changes in one commit.

### Rule 6 — Commit message discipline

Format: `type(scope): subject`
- Subject ≤72 chars
- Imperative mood ("add X" not "added X")
- No trailing period
- Body wrapped at 72 chars
- WHY in body, not just WHAT

Types: feat, fix, chore, docs, test, refactor, style, perf, ci, build

### Rule 7 — Pre-push commit review

Before any `git push`, invoke cm-planner with phrase "review commits for push" (or close paraphrase). This audits all unpushed commits for accuracy and convention compliance.

If `BLOCK PUSH — N issues` returned, address flagged commits via amend/rebase, then re-run review.

## Trivial fast-path

If cm-planner returns `Trivial — executor may proceed directly with: …`:
- Skip the question-batching pipeline
- Make the change as described
- Stage with `git add`
- Reviewer self-fast-paths (returns READY TO COMMIT in ≤10s)
- Commit

This path is for changes ≤5 lines, single file, no logic/type/import/control-flow change.

## Error recovery

| Situation | Action |
|---|---|
| cm-planner returns STOP_FOR_HUMAN | Surface verbatim, wait for user |
| Reviewer returns BLOCK COMMIT | Address HIGH findings, re-stage, re-review |
| Conventions-audit returns BLOCK CLOSURE | Address FAIL findings OR get user re-confirmation for overrides |
| Conventions-audit returns OVERRIDES PRESENT | Surface override block to user, wait for re-confirmation |
| Test-author returns failures (source bugs) | DO NOT complete. Open separate fix loop, re-run /change for the bug fix. |
| cm-planner (review mode) returns BLOCK PUSH | Amend/rebase flagged commits, re-run review |

## When NOT to use /change

- Pure conversational messages ("hi", "thanks", "wait")
- Direct Claude Code meta-commands (`/agents`, `/clear`, `/help`)

For everything else, use /change.

## Example flows

### Example 1 — Simple bug fix

```
User: /change fix the off-by-one error in pagination

Main agent: [invokes cm-planner]
cm-planner: Trivial — executor may proceed directly with: change `< total` to `<= total` in apps/web/src/lib/pagination.ts:42. Specialist agents: reviewer (will fast-path). Commit: fix(pagination): off-by-one in last page check.

Main agent: [announces Edit before running]
Main agent: [Edit pagination.ts]
Main agent: [git add]
Main agent: [invokes reviewer]
reviewer: REVIEW: Trivial diff fast-path. Lines: 1. READY TO COMMIT.

Main agent: [git commit -m "fix(pagination): off-by-one in last page check"]
Main agent: Done.
```

### Example 2 — Feature with ratification needed

```
User: /change add Stripe payments to the checkout flow

Main agent: [invokes cm-planner]
cm-planner: STOP_FOR_HUMAN

Open questions:
Q1. Pricing model: (a) one-time charges (b) subscriptions (c) both
- Recommendation: (a) — checkout flow implies one-time
- Trigger: load-bearing, affects schema

Q2. Webhook handling for failed payments: (a) retry 3x (b) immediate refund (c) hold for review
- Recommendation: defer (downstream of Q1)

Awaiting answer to Q1.

[Main agent surfaces verbatim to user. User answers. Pipeline continues.]
```

## Diagnostics

If the pipeline seems stuck or behaving unexpectedly:
1. Check `.claude/agents/` files exist and are well-formed
2. Run `/agents` in Claude Code to see which agents are loaded
3. Check CLAUDE.md for routing rules
4. Verify hooks in `.claude/settings.json` (if applicable)

If still stuck, ask the user how they'd like to proceed.
