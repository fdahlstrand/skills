---
name: to-workplan
description: Synthesize a Work Plan for a single chunk of work from the current conversation and the codebase, writing it to a gitignored .scratch/<workplan-slug>/ folder. A Work Plan documents one roadmap entry: its expected outcome, a checkbox-tracked list of Work Packages with dependencies, implementation notes, and any open questions. It also pulls in scope-relevant open (WIP) ADRs as research packages. Use this whenever the user wants to capture, plan, or write up a chunk of work — e.g. after a grill-work, grill-me, or design session, or when they say "make a work plan", "turn this into a work plan", "plan this out", or "write up this chunk of work". Do NOT use this to generate the detailed Work Package files — that is the to-workpackages skill.
---

# to-workplan

Turn the current shared understanding of a chunk of work into a **Work Plan** file. A Work Plan is a transient coordination artifact for one entry on the project Roadmap. It is the single source of truth for *what packages exist, in what dependency order, and their status* — but it does not duplicate durable documentation (the Architectural Description, ADRs, glossary), and it does not contain detailed implementation plans (those live in separate Work Package files, generated later by `to-workpackages`).

## Core principle: synthesize, never fabricate

Build the Work Plan from two sources only: (1) the conversation/context, and (2) the codebase. Read the relevant code before asserting how things work — don't guess at module names, interfaces, or existing behavior when you can check.

When a fact needed for the plan is **not** supported by the context and **not** discoverable in the codebase, do **not** invent it. Record it as an **Open Question** with an `OQ-n` identity. A Work Plan that honestly flags gaps is correct; one that papers over them with plausible guesses is a failure, because the gaps are exactly what the user needs to resolve before work starts. If the overall understanding is too thin to produce a meaningful plan, say so and suggest grilling/designing the chunk further rather than emitting a hollow plan.

## Where the file goes

Each Work Plan gets its own folder under a gitignored `.scratch/` at the project root:

```
.scratch/<workplan-slug>/
├── workplan.md          ← this file
├── wp1-<slug>.md        ← Work Package files (made later by to-workpackages)
├── wp2-<slug>.md
└── ...
```

Write the plan to `.scratch/<workplan-slug>/workplan.md`, where `<workplan-slug>` is a short kebab-case name for the chunk. Keeping each plan and its packages together in one folder avoids a noisy flat directory once several chunks have been planned. If `.scratch/` is not yet gitignored, add it to `.gitignore` (or tell the user to) — these files are transient and must not pollute history.

## Outcome vs. output

The Work Plan's goal is an **outcome** — a desired change in the state of the world or the system ("the expression language can evaluate user-defined functions"; "open question about transport framing is resolved and ADR-0007 accepted"). It is *not* a list of outputs (files, artifacts). The plan may mention outputs as a means, but the outcome is the point. An outcome may legitimately be a change to durable documentation (e.g. a prototyping chunk that resolves an open design question and updates the Architectural Description, or a chunk whose outcome is an Accepted ADR — the prototype itself may be disposable).

## Work Plan structure

Use this template:

```markdown
# Work Plan: <Chunk Name>

> Transient planning artifact. Roadmap entry: <reference to roadmap entry, if known>.
> Built against: <which Architectural Description viewpoint files / ADRs this plan was
>   planned against, if any — reference, do not copy. Omit if no AD exists.>

## Expected Outcome

<One or a few sentences describing the end state this chunk delivers. An outcome, not
an output. May include "Architectural Description updated" or "ADR-NNNN accepted" if
this chunk resolves an open design question.>

## Constraints

<Optional. The architectural boundaries this chunk must respect, drawn from the
scope-relevant Architectural Description viewpoint files (see "Architecture as
constraints" below). E.g. "per Logical view, Customer data referenced by ID only";
"per Runtime view, contexts communicate via domain events". Omit if no AD exists or
none are relevant. Work Packages inherit these as boundaries.>

## Phases

<Optional short prose describing the major phases/deliverables of this chunk, if it
has natural groupings. These become the `##`-level groupings in the Work Packages
list below. Omit if the chunk is small enough not to need them.>

## Work Packages

<The coordination surface. Single source of truth for package identity, name,
dependencies, and status. Group under phase headings. Each package is a checkbox:
`- [ ]` todo, `- [~]` in-progress, `- [x]` done. Annotate dependencies inline with
`deps: WP-n` (omit if none). A package is "ready" when it is unchecked and every WP
in its deps is `[x]`.>

### Phase 1: <name>
- [ ] WP-1: <name>
- [ ] WP-2: <name> — deps: WP-1

### Phase 2: <name>
- [ ] WP-3: <name> — deps: WP-1
- [ ] WP-4: <name> — deps: WP-2, WP-3
- [ ] WP-5: <name> — deps: WP-4 — blocked by OQ-1

## Open Questions

<Gaps that could not be resolved from context or codebase. Each gets an `OQ-n` ID so
packages can reference it (e.g. "blocked by OQ-1"). Phrase as a real question, never
a guessed answer. Omit the section only if there are genuinely none.>

- **OQ-1:** <question>

## Implementation Notes

<Lightweight, transient navigation aid: gotchas, ordering advice, pointers to
relevant ADRs. NOT the system of record — durable decisions with rationale belong in
ADRs, not here. May reference ADRs by ID. Keep it brief.>
```

## How to size and shape the Work Packages

You are listing packages here, not detailing them — but the list must reflect sound decomposition, because `to-workpackages` will elaborate exactly these. Each Work Package should be:

- **Cohesive** — one coherent thing. This is the non-negotiable top priority.
- **Single-sitting** — completable by one developer (human or agent) in one sitting. This is the strong default and the review gate.
- **Non-trivial** — not a cosmetic change; it should carry real content.
- **Value-bearing where possible** — ideally expressible as a user story ("As a…, I want…, so that…"). Some packages will instead be *architectural runway* (no standalone user value); that is acceptable **as long as the Work Plan as a whole cashes out to a user-visible outcome.**

When these collide, the priority order is **cohesion > single-sitting > user-value**. If a vertical, user-visible slice won't fit one sitting, split it into single-sitting packages — accepting that some are runway whose value accrues to a later package in the plan. Don't split so aggressively that the plan stops delivering visible value; that's what the plan-level outcome guards against.

Expect a typical plan to have **more than four** packages, sometimes a long list. That's why dependencies (not a flat sequence) are recorded: ordering is generally not unique, and a developer should be able to pick any *ready* package next.

## Dependencies, not sequence

Record dependencies per package (`deps: WP-n`), not a global numeric order. The WP identities (WP-1, WP-2, …) are just stable handles, not an execution order. "What's next" is computed: any unchecked package whose deps are all done is pickable. Convergence points (where runway packages feed a vertical one) show up as packages with several inbound deps.

## Open (WIP) ADRs become research packages

Open architectural decisions are tracked as `Status: WIP` ADRs in `docs/adr/` (created by `grill-work` or the `adr` skill). An unresolved decision that bears on this chunk is a real blocker, so fold it into the plan — but only if it actually impacts *this* chunk's scope.

Process:

1. **Read from disk, not just the conversation.** Grep `docs/adr/` for `Status: WIP` (e.g. `grep -rln "Status: WIP" docs/adr/`). Disk is authoritative — this is why WIP markers exist, so the signal survives lost context. The current conversation is a *supplement* that catches decisions not yet flushed to disk. Because of this, `to-workplan` is useful standalone: pointed at a project with WIP ADRs lying around, it will surface them even with no grill in context.

2. **Filter by scope relevance, biased to include.** Openness alone does not pull an ADR in — it must impact this chunk's scope. But the relevance judgment is fuzzy, and the two failure modes are asymmetric: a *false include* is visible and you can cut it; a *false exclude* leaves a blocking decision on disk and the work hits it mid-execution. So **bias toward inclusion** and never silently drop an open ADR.

3. **Show and confirm.** List the WIP ADRs found. For each, state whether you judged it in-scope and why (one line), and let the user veto. In-scope ones become **research Work Packages** whose *outcome* is the ADR reaching `Status: Accepted` (not code). Also surface the ones judged **out of scope** ("found these open ADRs, judged out of scope for this chunk: …") so a false-exclude is caught now rather than during execution.

4. **Flag readiness.** If research packages dominate the plan, say so plainly — a chunk that's mostly unresolved decisions isn't ready to build, and the fuzziness of scoping it is itself evidence of that. Suggest resolving the ADRs (a focused `grill-me` session per ADR) before planning build work.

A research Work Package sits in the package list like any other, e.g.:

```
### Phase 0: Resolve open decisions
- [ ] WP-1: Resolve transport framing (deliver ADR-0007 Accepted)
```

and downstream packages that depend on the decision carry `deps: WP-1`.

## Architecture as constraints (optional input)

If an Architectural Description exists (`docs/architecture/`, owned by `architecture-description`), it is the durable context this chunk is planned against — the role a `SPECIFICATION.md` once pretended to fill. Read it as a source of **binding constraints**, not just background.

1. **Optional.** Many projects — especially early prototypes — have no AD. If `docs/architecture/` is absent, skip this entirely; it is not an error.

2. **Read only the scope-relevant viewpoints.** Same discipline as the WIP-ADR filter, for the same lightweight reason: pull in the Logical view if the chunk touches structure, the Runtime view if it touches interaction, the Deployment view if it touches topology — not the whole AD every time.

3. **Capture constraints in the Constraints section.** Record the architectural boundaries the chunk must respect (e.g. "per Logical view, Customer data referenced by ID only"). Work Packages inherit these; `to-workpackages` does not re-derive them from the AD — the plan's Constraints section is the single source of truth the packages cite.

4. **Record what you built against, for staleness detection.** Note in the header which viewpoint files (and ideally their state) the plan was built against. Work Plans are short-lived and there's normally at most one in progress, so significant AD drift under a live plan is rare — but if this plan is picked up later and those viewpoint files have since changed, surface "the architecture this plan was built against has changed — review the Constraints section." Flag it; don't auto-replan. The human judges whether to revise.

## What the Work Plan does NOT contain

- It does not restate the Architectural Description, ADRs, or glossary — it references them.
- It does not contain detailed task/subtask implementation plans — those are Work Package files, made by `to-workpackages`.
- It does not store per-package dependencies or status *inside the WP files* — that lives here, in the plan, the coordination surface.
- It does not record durable architectural decisions with rationale — those go to ADRs.

## After writing

Tell the user the path and summarize the outcome and package count. **Call out any Open Questions explicitly** — those block `to-workpackages` until resolved. Separately, report the **WIP ADRs** you found: which became research packages (in-scope) and which you judged out of scope, so the scope calls are visible and vetoable. If an AD exists, note which viewpoint files you drew **Constraints** from. If research packages dominate, flag that the chunk isn't ready to build and suggest resolving the open decisions first (e.g. a `grill-me` session per ADR).
