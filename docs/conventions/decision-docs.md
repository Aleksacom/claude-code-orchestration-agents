# Decision Document Conventions

This file defines the format, structure, and conventions for decision documents in `docs/decisions/`.

The `conventions-audit` agent enforces these conventions.

---

## File location and naming

All decision documents live under `docs/decisions/`.

### Naming patterns

- **Phase decision**: `phase-NN-{slug}.md`
- **Sub-phase decision**: `sub-phase-NN-{letter}-{slug}.md`
- **Standalone decision**: `YYYY-MM-DD-{slug}.md`

Choose the most appropriate pattern. Sub-phase decisions identify both their parent phase (NN) and their position within the phase (letter, alphabetical).

Examples:
- `phase-22-personal-scorer.md`
- `sub-phase-22-a-event-tracking.md`
- `2026-05-15-database-migration-strategy.md`

---

## Frontmatter format

Every decision document MUST start with YAML frontmatter:

```yaml
---
decision-id: YYYY-MM-DD-short-slug
date: YYYY-MM-DD
status: active | superseded | deferred
applies-to: phase-NN OR sub-phase-NN-X OR area-name
approved-by: <user-name>
---
```

Optional frontmatter fields:

```yaml
supersedes: YYYY-MM-DD-prior-decision-slug
superseded-by: YYYY-MM-DD-newer-decision-slug
conventions-override:
  - check: length
    approved-by: <user-name>
    date: YYYY-MM-DD
    reason: <explanation>
```

---

## Required sections

In this order:

### 1. Context (or "Why we needed to decide")

1-3 sentences explaining what situation prompted this decision. Plain English.

### 2. Options considered

Numbered or bulleted list of alternatives. Each option gets:
- A brief description
- Key trade-offs

### 3. What we chose (or "Decision")

State the chosen option clearly. One line if possible.

### 4. Why (or "Reasoning")

2-5 sentences explaining the reasoning. WHY this option vs others. Reference evidence, constraints, or principles.

### 5. Implications

What this decision means downstream. Other decisions it constrains, work it unblocks, costs it incurs.

### 6. If we want to change later (or "Reversibility")

What would prompt re-opening this decision? What's the cost of changing course?

---

## Length budgets

- **Routine decision**: ≤50 lines (non-blank)
- **Substantive decision**: ≤100 lines (non-blank)
- **Paradigm-shift decision**: ≤150 lines (non-blank)

If a decision genuinely requires more length, add a `conventions-override` block (see below).

---

## Override blocks

When a decision document deviates from a convention (length, structure, content-sync), record the override explicitly:

```yaml
conventions-override:
  - check: length
    approved-by: aleksacom
    date: 2026-05-15
    reason: Paradigm-shift architectural decision; expanded reasoning required for future contributors.
  - check: content-sync
    approved-by: aleksacom
    date: 2026-05-15
    reason: Metrics auto-update mid-arc; FEATURES prose frozen until arc closure.
```

Each override entry requires:
- `check`: the convention being overridden (length, content-sync, structure, etc.)
- `approved-by`: explicit user name
- `date`: when the override was approved
- `reason`: explanation in plain English

Audit-time re-confirmation: when overrides are present, conventions-audit returns `OVERRIDES PRESENT — user re-confirmation required`. User must explicitly confirm before commit/closure proceeds.

---

## Content-sync rule

If a decision document references metrics, counts, or specific numbers (e.g., "increased catalog from 200 to 300"), those numbers MUST match the codebase at commit time.

If the doc and the code disagree, either:
- Update the doc to match reality, OR
- Add a `content-sync` override explaining the intentional drift (e.g., "METRICS auto-bumped only; FEATURES at arc close")

---

## Versioning and supersession

Decision docs are append-only. Never delete or rewrite a prior decision.

If a decision needs to be changed:
1. Create a NEW decision doc with `supersedes: <prior-decision-id>` in frontmatter
2. Update the prior decision's frontmatter with `superseded-by: <new-decision-id>` and `status: superseded`
3. Both files stay in the repo permanently

This preserves the history of HOW decisions evolved, not just what's current.

---

## Templates

Copy `docs/decisions/_template.md` as the starting point for new decisions. The template includes all required frontmatter fields and sections in the right order.

---

## What goes in a decision doc vs elsewhere

| Type of content | Where it lives |
|---|---|
| Architecture rules that apply always | `CLAUDE.md` or `.claude/rules/*.md` |
| Topic-scoped conventions | `.claude/rules/<topic>.md` |
| A specific choice made at a specific time | `docs/decisions/*.md` |
| Open questions for future | `docs/backlog/candidates.md` |
| Day-to-day documentation | `README.md` and `docs/reference/*.md` |

If a decision is small and routine (e.g., picked a library, picked a folder name), inline reasoning in the commit message is fine — don't create a decision doc.

If a decision is substantive (architectural, irreversible, affects multiple people or future work), create the decision doc.

When in doubt, create the doc. Future-you will appreciate it.

---

## Examples

### Example 1 — Minimal decision doc (~25 lines)

```markdown
---
decision-id: 2026-05-15-jwt-library
date: 2026-05-15
status: active
applies-to: auth-system
approved-by: aleksacom
---

# JWT library: jose vs jsonwebtoken

## Context
We need server-side JWT handling for auth. Two well-maintained libraries available.

## Options considered
- (a) `jose` — modern, ESM-first, supports JWE/JWS/JWT in unified API
- (b) `jsonwebtoken` — older, CommonJS, more widely used

## Decision
Selected (a) — `jose`.

## Why
Modern ESM compatibility matches our build pipeline. Smaller bundle. Better TypeScript types. Active maintenance vs `jsonwebtoken` (last major update 18 months ago).

## Implications
- Need to use ESM imports throughout auth module
- Slightly steeper learning curve for team unfamiliar with jose API

## If we want to change later
Could swap to `jsonwebtoken` in ~half-day refactor — auth surface area is small. Trigger: if jose maintenance lapses or a critical bug surfaces.
```

### Example 2 — Decision with override

```markdown
---
decision-id: 2026-05-15-database-migration-strategy
date: 2026-05-15
status: active
applies-to: database
approved-by: aleksacom
conventions-override:
  - check: length
    approved-by: aleksacom
    date: 2026-05-15
    reason: Migration strategy affects all future schema work; expanded reasoning warranted.
---

# Database migration strategy: Prisma migrate vs custom SQL

[... 130 lines of substantive content ...]
```

When conventions-audit runs on this, it returns `OVERRIDES PRESENT — user re-confirmation required`. The user must explicitly say "confirmed" or similar before the commit proceeds.
