---
name: design-review
description: Stress-test a plan or design against domain language, docs, and code. Ask hard questions one at a time. Update CONTEXT.md and ADRs as decisions crystallize. Use when you already have a draft plan or spec and want to find gaps before implementation.
---

# Design review

Stress-test a plan or design against the project's domain language, existing docs, and code. Ask hard questions one at a time. Capture resolved terms in `CONTEXT.md` and hard-to-reverse decisions in `docs/adr/` as they crystallize.

## When to use

Use this when you **already have** a plan, spec, or design doc and want to find gaps, contradictions, and fuzzy language before implementation.

Do **not** use this when:

- The idea is still fuzzy and no draft exists yet. First agree on goal, constraints, and 2–3 approaches; write a design doc; then run design review on that doc.
- You want to review code diffs for bugs or security. That is a separate pass focused on correctness, not domain design.

## Input

`design review [PATH_OR_TOPIC]` — optional.

- **PATH**: design doc, plan, or spec
- **TOPIC**: short description when no file exists yet
- If omitted, use the current conversation topic or ask what to review.

## 0. Load domain context

Before asking anything, explore the project.

### Documentation layout

**Single context** (most repos):

```
/
├── CONTEXT.md
├── docs/
│   └── adr/
│       └── NNNN-{slug}.md
└── {source}/
```

**Multiple contexts** (when `CONTEXT-MAP.md` exists at the root):

```
/
├── CONTEXT-MAP.md
├── docs/
│   └── adr/                    ← system-wide decisions
└── {source}/
    ├── {context-a}/
    │   ├── CONTEXT.md
    │   └── docs/adr/           ← context-specific decisions
    └── {context-b}/
        ├── CONTEXT.md
        └── docs/adr/
```

Infer which structure applies:

- `CONTEXT-MAP.md` exists → read it to find contexts and their relationships
- Only a root `CONTEXT.md` exists → single context
- Neither exists → create a root `CONTEXT.md` lazily when the first term is resolved

When multiple contexts exist, infer which one the current topic relates to. If unclear, ask.

**CONTEXT-MAP.md format:**

```md
# Context Map

## Contexts

- [{Context A}]({path}/CONTEXT.md) — {one-line description}
- [{Context B}]({path}/CONTEXT.md) — {one-line description}

## Relationships

- **{A} → {B}**: {how they interact}
```

### What to read

1. **Glossary** — `CONTEXT.md` (root or per-context via `CONTEXT-MAP.md`)
2. **Decisions** — `docs/adr/*.md` and context-specific `docs/adr/` if present
3. **Plans and specs** — design docs, plans folders, and any linked wiki pages referenced in doc frontmatter
4. **Code** — search the codebase to verify how things work today; check architecture docs and module boundaries when the design crosses components

Create `CONTEXT.md` or `docs/adr/` lazily — only when you have something to write.

## 1. Grilling session

Interview relentlessly until you and the user share a clear understanding. Walk each branch of the design tree. Resolve dependencies between decisions one at a time.

### Rules

- **One question at a time.** Wait for the user's answer before the next question.
- **Explore, don't ask.** If the answer is in the codebase or docs, look it up and report what you found.
- **Recommend an answer** with every question. State your pick and why.
- **No implementation** during the session unless the user explicitly asks to update docs (see section 2).

### Challenge the plan

| Trigger | Action |
|--------|--------|
| Term conflicts with `CONTEXT.md` | Call it out immediately. Quote the existing definition and ask which meaning applies. |
| Vague or overloaded term | Propose a precise canonical term. Ask whether it maps to an existing concept or needs a new glossary entry. |
| Relationship between concepts | Invent a concrete scenario that probes an edge case. Force a clear boundary between concepts. |
| User states how something works | Cross-check the code. If it contradicts the plan, surface the mismatch and ask which is authoritative. |
| Missing non-goals or scope | Ask what is explicitly out of scope for this change. |

### Question order (adapt to the design)

Work through branches in dependency order. Typical sequence:

1. **Problem and success** — What user or system outcome changes? How will we know it worked?
2. **Scope and non-goals** — What is in v1? What is explicitly deferred?
3. **Domain terms** — Name the nouns and verbs. Resolve collisions with existing language.
4. **Boundaries** — Which component or context owns what? What crosses a boundary and how?
5. **Data and contracts** — Inputs, outputs, persistence, events, API shapes.
6. **Failure and edge cases** — Timeouts, retries, partial success, auth, idempotency.
7. **Observability and ops** — Logging, metrics, rollout, rollback.
8. **Trade-offs** — Alternatives considered; why this path over others.

Skip branches that do not apply. Go deeper where the design is fuzzy or risky.

## 2. Update documentation inline

Do not batch glossary or ADR updates until the end. Capture decisions as they land.

### CONTEXT.md (glossary only)

- **Single context:** root `CONTEXT.md`
- **Multiple contexts:** the `CONTEXT.md` for the relevant bounded context (use `CONTEXT-MAP.md` to pick)

`CONTEXT.md` is a glossary and nothing else. No implementation details. No scratch notes. No specs.

**Format:**

```md
# {Context Name}

{One or two sentences: what this context is and why it exists.}

## Language

**{Canonical term}**:
{One or two sentence definition: what it IS.}
_Avoid_: {alternative terms to not use}

**{Another term}**:
{Definition.}
_Avoid_: {alternatives}
```

**Rules:**

- Be opinionated: pick one canonical term; list alternatives under `_Avoid_`.
- Definitions are one or two sentences: what it IS, not what it does.
- Only project-specific domain terms. Before adding a term, ask: is this unique to this context, or a general programming concept? Only the former belongs.
- Group under subheadings when natural clusters emerge. A flat list is fine when all terms belong to one area.

### ADRs (sparingly)

Only offer an ADR when **all three** are true:

1. **Hard to reverse** — meaningful cost to change later
2. **Surprising without context** — a future reader will look at the code and wonder "why did they do it this way?"
3. **Real trade-off** — genuine alternatives existed and you picked one for specific reasons

If any condition fails, skip the ADR. Easy-to-reverse decisions get reversed, not documented. Obvious choices need no record.

**What qualifies:**

- Architectural shape
- Integration patterns between contexts or services
- Technology choices with lock-in — not every library
- Boundary and scope decisions about data ownership
- Deliberate deviations from the obvious path
- Constraints not visible in code
- Rejected alternatives when the rejection is non-obvious

**Location:** `docs/adr/NNNN-slug.md`. Scan `docs/adr/` for the highest existing number and increment by one. Create the directory lazily when the first ADR is needed.

**Format:**

```md
# {Short title of the decision}

{1-3 sentences: context, decision, and why.}
```

That can be a single paragraph. The value is recording *that* a decision was made and *why*.

Optional sections — only when they add genuine value:

- `Status` frontmatter (`proposed | accepted | deprecated | superseded by ADR-NNNN`)
- **Considered Options** — when rejected alternatives are worth remembering
- **Consequences** — when non-obvious downstream effects need to be called out

### Design doc updates

When the reviewed file is a plan or spec, offer to patch it with resolved decisions, clarified terms, and scoped non-goals. Do not rewrite the whole doc. Minimal diffs for what changed during the review.

## 3. Session output

After each answered question, briefly note:

- Decision made (or still open)
- Doc updated (CONTEXT term, ADR offered/created, design doc patch)
- Code contradiction found (if any)

When the user signals the review is done (or all branches are resolved), post a closing summary:

```md
## Design review summary

**Reviewed:** {path or topic}

### Resolved
- {decision or term} → {outcome}

### Open questions
- {anything still undecided}

### Docs updated
- {file paths created or changed}

### ADRs
- {created or deferred, with reason}

### Code contradictions
- {mismatch and recommended resolution, or "None."}

### Recommended next step
- {next action}
```

## Skip

- Writing implementation code or scaffolding during the review
- Creating ADRs for obvious or easily reversed choices
- Filling `CONTEXT.md` with generic programming terms
- Batch-updating docs at the end instead of inline as terms resolve
- Reviewing code diffs for bugs — that is outside the scope of this command
