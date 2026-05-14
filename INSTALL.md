# Installing the Claude Code Agent Set

**For Claude Code users who want full automated workflow discipline.**

This adds 5 sub-agents (cm-planner, cm-decider, reviewer, conventions-audit, test-author) plus the `/change` slash command to your project. The result: every change request goes through automated planning, review, audit, and test-coverage steps.

## Prerequisites

- Claude Code installed and working in your project (`claude` command available in terminal)
- An existing project (or new project) with at least an initial git commit
- Familiarity with the basic concepts from the universal framework (sub-phases, gates, decision docs)

If you haven't set up the universal framework first, do that — these agents extend it with automation. See the main README at the repo root.

## What gets installed

```
your-project/
├── .claude/
│   ├── agents/
│   │   ├── cm-planner.md       ← Architectural authority
│   │   ├── cm-decider.md       ← Question routing
│   │   ├── reviewer.md         ← Pre-commit review
│   │   ├── conventions-audit.md ← Decision-doc enforcement
│   │   └── test-author.md      ← Coverage gap filler
│   └── commands/
│       └── change.md           ← /change slash command
├── docs/
│   └── conventions/
│       └── decision-docs.md    ← Decision-doc format spec
└── CLAUDE.md                   ← Updated with routing rules (instructions below)
```

## Installation steps

### Step 1 — Copy the agent files

Copy the entire `.claude/agents/` directory and `.claude/commands/` directory from this repo into your project's root.

If your project is at `~/projects/my-app/`:

```bash
cd ~/projects/my-app
cp -r path/to/this/repo/.claude/agents .claude/agents
cp -r path/to/this/repo/.claude/commands .claude/commands
```

If you don't have a `.claude/` directory yet, create it first (`mkdir .claude`).

### Step 2 — Copy the conventions file

```bash
mkdir -p docs/conventions
cp path/to/this/repo/docs/conventions/decision-docs.md docs/conventions/
```

### Step 3 — Update your CLAUDE.md

Open `CLAUDE.md` at your project root (create it if it doesn't exist). Add this section near the top:

```markdown
## Routing rules

Five routing rules below; the pipeline summary at the end shows the order.

### Planner routing

Before responding to ANY user message in this repository, you MUST first invoke the cm-planner subagent (`.claude/agents/cm-planner.md`) with the user's request. Wait for the cm-planner's response. Then:

- If the cm-planner's reply starts with `STOP_FOR_HUMAN`, surface the cm-planner's full message to the user verbatim and wait for the user's decision before proceeding. Do not paraphrase or strip the token.
- Otherwise, follow the cm-planner's implementation prompt to execute the work.

The cm-planner names the specialist agents required for the change under a "Specialist agents" section in its implementation prompt. The main agent invokes those agents in the order named.

Do NOT respond to the user directly without going through the cm-planner first, regardless of how simple or trivial the request appears.

Exceptions: this rule does NOT apply to:
- Direct meta-commands to Claude Code itself (`/agents`, `/hooks`, `/plugin`, etc.)
- Pure conversational messages that don't request any work (greetings, acknowledgments)

### Reviewer routing

After the executor implements the cm-planner's prompt and STAGES the change (`git add`) but BEFORE creating the commit, you MUST invoke the reviewer subagent (`.claude/agents/reviewer.md`) on the staged diff. If the reviewer's final line is `BLOCK COMMIT — N HIGH+ findings.`, do not proceed with the commit until findings are addressed or explicitly overridden by the user. If the final line is `READY TO COMMIT`, proceed.

### Conventions-audit routing

Whenever the staged diff modifies any file under `docs/decisions/` OR a sub-phase / phase is being declared closed / wrapped / complete, you MUST also invoke the conventions-audit subagent (`.claude/agents/conventions-audit.md`) — after the reviewer, before the commit.

If the conventions-audit final line is:
- `BLOCK CLOSURE — N FAIL findings` — do not proceed
- `OVERRIDES PRESENT — user re-confirmation required` — surface override block verbatim, wait for user re-confirmation
- `ALL CLEAR` — proceed

### Test-author routing

After any feature work that adds new behavior to source under your monitored paths (specify which — typically `src/`, `packages/*/src/`, or `apps/*/src/`), AND BEFORE declaring the task complete, you MUST invoke the test-author subagent (`.claude/agents/test-author.md`).

Skip for: pure markdown changes, pure schema/migration edits with no consuming code in scope.

If its final line is `TESTS ADDED — N tests, M failures (source bugs surfaced)`, do not declare the task complete until the surfaced source bug is addressed in a separate fix loop.

### Pre-push commit message review

Before any `git push` to `origin` (or any remote), invoke the cm-planner with the trigger phrase "review commits for push" (or any close paraphrase). The cm-planner audits each unpushed commit for accuracy, Conventional Commits format, WHY clarity, and subject discipline (≤72 chars, imperative, no trailing period).

Returns:
- `READY TO PUSH` — proceed with push
- `BLOCK PUSH — N issues across M commits` — surface findings table, amend/rebase, re-run review

### Pipeline summary

For non-trivial work (full pipeline):

```
user request
  → cm-planner
  → executor (Edit / Write / Bash)
  → git add (staging)
  → reviewer
  → conventions-audit (if docs/decisions/** touched OR phase closure)
  → test-author (if source under monitored paths added new behavior)
  → git commit
  → (loop back to executor for more changes if cm-planner prompt has more steps)
  → "review commits for push" → cm-planner (commit review mode)
  → git push
```

For trivial work (cm-planner returns "Trivial — executor may proceed directly with: …"):

```
user request
  → cm-planner (returns trivial directive)
  → executor (single Edit)
  → git add
  → reviewer (fast-paths in ≤10s, returns READY TO COMMIT)
  → git commit
```
```

### Step 4 — Customize monitored paths

In your CLAUDE.md, replace "your monitored paths" with the actual paths. Common patterns:

| Project type | Monitored paths |
|---|---|
| Single-package Python | `src/` |
| Single-package Node | `src/`, `lib/` |
| Monorepo (Turborepo, Nx, Yarn workspaces) | `packages/*/src/`, `apps/*/src/` |
| Next.js | `apps/web/src/`, `apps/api/src/` (or `app/`, `pages/`) |
| Django | `apps/*/`, `<project_name>/` |

Pick what matches your structure. This tells test-author which directories require coverage.

### Step 5 — Restart Claude Code

Exit and re-launch Claude Code in your project folder so it picks up the new `.claude/` directory.

```bash
# Exit current session (Ctrl+D or `exit`)
cd ~/projects/my-app
claude
```

### Step 6 — Verify

In Claude Code, run:

```
/agents
```

You should see all 5 agents listed:
- cm-planner
- cm-decider
- reviewer
- conventions-audit
- test-author

If any are missing, check the `.claude/agents/` directory for that file.

Then test the /change command:

```
/change
```

Claude Code should show the command's description and offer to use it.

## First practice run

Make a tiny test change to verify the pipeline works:

```
/change update the README's tagline to be more punchy
```

Expected flow:
1. cm-planner reads the README, proposes a tagline change, returns either an implementation prompt OR a STOP_FOR_HUMAN with options
2. You approve or pick an option
3. Main agent makes the edit
4. Main agent stages with `git add`
5. Reviewer runs (likely fast-paths as trivial)
6. Commit happens with a clean message
7. You're done

If any step fails, check that:
- The agent files have the right frontmatter (name, description, tools)
- CLAUDE.md routing rules are present
- Claude Code was restarted after agent installation

## Optional — Add hooks for tripwire discipline

For maximum protection, add `.claude/settings.json` with hooks that announce every tool call:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Edit|Write|Read",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'About to run: ${TOOL_NAME}'"
          }
        ]
      }
    ]
  }
}
```

This makes silent tool calls impossible — every operation announces itself before executing.

## Adapting to your project

The agent files are intentionally generic. To make them more useful for YOUR project:

### Add project-specific rules

Create `.claude/rules/<topic>.md` for each significant area:
- `.claude/rules/auth.md` — auth conventions
- `.claude/rules/api.md` — API conventions
- `.claude/rules/database.md` — DB conventions

Reference them from CLAUDE.md:

```markdown
**Architecture invariants by topic — `.claude/rules/`**
- [auth.md](.claude/rules/auth.md)
- [api.md](.claude/rules/api.md)
- [database.md](.claude/rules/database.md)
```

cm-planner reads these when planning work in those areas.

### Add architecture invariants

In CLAUDE.md, add an "Architecture rules — NEVER violate these" section listing absolute rules for your project:

```markdown
## Architecture rules — NEVER violate these

1. All database writes go through Prisma. No raw SQL.
2. External API calls always use the cache layer.
3. No secrets in code — only in `.env` files.
4. Authentication uses Supabase only. No alternative auth providers.
```

The reviewer references these when checking diffs.

### Document your commit conventions

If your project uses specific commit patterns, document them in `.claude/rules/commits.md` or in CLAUDE.md directly:

```markdown
## Commit conventions

- Format: `type(scope): subject`
- Types: feat, fix, chore, docs, test, refactor, style
- Subject ≤72 chars
- No `Co-Authored-By` trailer (we don't attribute AI assistants)
- Body wrapped at 72 chars, WHY-not-WHAT
```

## Troubleshooting

### "cm-planner not found"

Check that `.claude/agents/cm-planner.md` exists and starts with valid YAML frontmatter:

```yaml
---
name: cm-planner
description: ...
tools: Read, Grep, Glob, Bash
---
```

Restart Claude Code after fixing.

### "Pipeline feels too slow"

If the full pipeline takes too long on routine work, check that cm-planner is using the trivial fast-path correctly. Routine changes should return "Trivial — executor may proceed directly with: …" and skip the full pipeline.

If cm-planner is bottlenecking on trivial work, it may need tuning. Edit `cm-planner.md` and adjust the trivial fast-path criteria to match your project's pace.

### "Reviewer is flagging too many things"

If reviewer is blocking on style nits (Category C), it shouldn't be — those are advisory. Re-read reviewer.md and check that the BLOCK COMMIT criteria only fires on Category A (correctness) issues.

### "Conventions-audit blocking on decision-doc length"

If a decision genuinely needs more length, add a `conventions-override` block to the doc's frontmatter. See `docs/conventions/decision-docs.md` for the format.

### "Test-author taking too long"

If your project has slow test runs, test-author may seem slow. Consider:
- Run only the affected test files (not full suite)
- Use Jest's `--findRelatedTests` or equivalent
- Skip test-author for trivial changes (cm-planner can mark these as test-skip)

## What you can't do with this set

These agents work in Claude Code only. They use Claude Code's sub-agent dispatch system, which doesn't exist in Kimi CLI, Cursor, or other tools.

If you want similar discipline in other AI tools, use the universal prompt instead (see main repo README).

## What's next

Once installed and running:

1. **Use /change for everything** — make it your reflex
2. **Watch the pipeline catch things** — note when reviewer or conventions-audit blocks you, that's the system working
3. **Tune over time** — edit agent files to match your project's evolving needs
4. **Share back** — if you improve an agent, consider opening a PR to the main repo

## Resources

- Main repo README: <link to your GitHub repo root>
- Universal framework (no agents needed): <link to setup-prompt.md>
- Claude Code documentation: docs.claude.com/en/docs/claude-code
- Issues / questions: <link to your repo issues>

---

*This agent set was developed from real production project work and generalized for any codebase. Adapt freely.*
