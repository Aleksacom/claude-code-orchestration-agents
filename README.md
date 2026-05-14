# Claude Code Orchestration Agents

> Production-grade workflow agents for Claude Code. Drop-in sub-agent set that adds automated planning, review, audit, and test-coverage to every change you make.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code only](https://img.shields.io/badge/Works%20with-Claude%20Code-orange)]()
[![No code required](https://img.shields.io/badge/Setup%20time-15%20min-green)]()

## What this is

A pre-built set of 5 Claude Code sub-agents plus a `/change` slash command that turns any project into a disciplined development environment:

- **cm-planner** — Architectural authority. Plans every change. Surfaces decisions that need your input before any code is touched.
- **cm-decider** — Routes individual questions to "decide now" vs "auto-resolve" based on 14 trigger criteria.
- **reviewer** — Reviews every staged diff before commit. Categorizes findings as blocking, medium, or nits.
- **conventions-audit** — Enforces decision-doc conventions when sub-phases close or decision files are touched.
- **test-author** — Adds tests for new behavior in monitored source paths. Surfaces source bugs that tests reveal.
- **/change** — Universal slash command that orchestrates the full pipeline.

## The problem this solves

Claude Code is fast. Without discipline, that speed becomes risk:

- The AI changes 12 files when you asked for one
- Important decisions get baked in silently
- "Why did we do this?" mysteries surface months later
- Mistakes compound because there's no paper trail

This agent set adds checkpoints throughout the workflow. Every change goes through `plan → verify → implement → review → audit → test → commit`. You stay in control of architectural decisions while routine work flows automatically.

## How it works

```
Your request
     ↓
/change <user request>
     ↓
cm-planner (plans scope, surfaces ratifications)
     ↓
[ if STOP_FOR_HUMAN: wait for your decision ]
     ↓
executor (Edit / Write / Bash with announced operations)
     ↓
git add (stage changes)
     ↓
reviewer (HIGH / MEDIUM / LOW findings on staged diff)
     ↓
conventions-audit (if docs/decisions/** touched OR sub-phase closing)
     ↓
test-author (if monitored source paths added new behavior)
     ↓
git commit (with clean conventional commit message)
     ↓
... continue or wrap sub-phase ...
     ↓
cm-planner "review commits for push" → git push
```

## Installation

See [INSTALL.md](INSTALL.md) for the complete walkthrough. Quick version:

1. Clone or copy this repo's `.claude/`, `docs/`, and other files into your project root
2. Update your project's `CLAUDE.md` with the routing rules (full text in INSTALL.md)
3. Restart Claude Code (`exit`, then `claude` again)
4. Verify with `/agents` — all 5 should appear
5. Try `/change update the README tagline` as a practice run

Total time: ~15 minutes.

## Requirements

- Claude Code installed (`claude` command available in your terminal)
- An existing project with at least an initial git commit
- Familiarity with the basic concepts of sub-phases, gates, and decision docs

If you haven't worked with that mental model before, start with the [universal framework](https://github.com/Aleksacom/ai-orchestration-framework) — same patterns but tool-agnostic. This repo extends it with automation.

## What you get

After installation, your project has:

```
your-project/
├── .claude/
│   ├── agents/
│   │   ├── cm-planner.md         ← Plans every change
│   │   ├── cm-decider.md         ← Routes questions
│   │   ├── reviewer.md           ← Reviews diffs
│   │   ├── conventions-audit.md  ← Enforces conventions
│   │   └── test-author.md        ← Fills coverage gaps
│   └── commands/
│       └── change.md             ← /change slash command
├── docs/
│   └── conventions/
│       └── decision-docs.md      ← Decision-doc format spec
└── CLAUDE.md                     ← Updated with routing rules
```

Plus an AI assistant that uses all of them automatically.

## Customization

Each agent file is plain markdown. Open and edit any of them to match your project's specific patterns:

- Add project-specific trigger phrases to cm-planner
- Adjust reviewer's blocking criteria
- Change conventions-audit's length budgets
- Update test-author's monitored paths

The files are intentionally generic — you make them yours.

## What this is NOT

- ❌ Not a Claude Code installer (you bring your own)
- ❌ Not appropriate for tools other than Claude Code (use the [universal framework](https://github.com/Aleksacom/ai-orchestration-framework) for Kimi/Cursor/ChatGPT)
- ❌ Not zero-config — you'll edit the agents to match your project over time
- ❌ Not magic — the discipline works because YOU follow the pipeline, agents enforce it

## Companion tools

This is part of a small ecosystem:

| Tool | Audience | Where |
|---|---|---|
| **Universal AI Orchestration Framework** | Any AI coding tool, any project type | [github.com/Aleksacom/ai-orchestration-framework](https://github.com/Aleksacom/ai-orchestration-framework) |
| **Claude Code Agent Set** (this repo) | Claude Code users wanting full automation | You are here |

Use whichever matches your tool. They share the same mental model.

## Why this exists

I'm a business analyst in banking-sector IT — not a developer. When AI coding tools made it possible for me to build my own projects, I loved the speed but kept losing control. The AI would change things I didn't understand. Decisions got re-debated because nothing was written down. Mistakes compounded.

This agent set is what I built for myself, refined through real production project work. Sharing it freely so others can skip the painful learning curve.

## License

MIT License — free to use, fork, modify, redistribute. See [LICENSE](LICENSE) for details.

## Issues and feedback

Found a bug? Want to suggest an improvement? [Open an issue](../../issues/new).

Or just star the repo if it helped you. ⭐

---

**Star this repo if it helped you.** ⭐  
**Found a problem?** [Open an issue](../../issues/new).  
**Want updates?** [Watch this repo](../../subscription) for releases.
