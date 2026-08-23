---
name: adr
description: Create and manage Architecture Decision Records (ADRs) in docs/adr/. Owns the ADR template, sequential ADR-NNNN-<slug>.md numbering, a lightweight ADR index (docs/adr/README.md), the Status lifecycle (WIP → Accepted → Superseded), and the three-test gate that decides whether a decision is even worth recording. Use this whenever a meaningful, hard-to-reverse design decision is made or deferred, or when the user says "record an ADR", "write a decision record", "is this an ADR?", "document this decision", or asks to accept/supersede an existing one. Other skills (grill-work, to-workplan) reference this skill's WIP/Accepted contract — it is the single source of truth for ADR conventions. Use standalone or as part of a grilling session.
---

# adr

Architecture Decision Records capture *why* a system is shaped the way it is — the decisions a future reader couldn't reconstruct from the code alone. This skill owns the ADR conventions that the rest of the toolchain depends on.

## The three-test gate: should this even be an ADR?

Most decisions are **not** ADRs. Before recording one, check all three tests. **An ADR is warranted only when all three are true:**

1. **Hard to reverse** — the cost of changing your mind later is meaningful.
2. **Surprising without context** — a future reader will look at the code and wonder "why on earth did they do it this way?"
3. **The result of a real trade-off** — there were genuine alternatives and you picked one for specific reasons.

If a decision is **easy to reverse**, skip it — you'll just reverse it. If it's **not surprising**, nobody will wonder why. If there was **no real alternative**, there's nothing to record beyond "we did the obvious thing."

### What qualifies (calibration examples)

- **Architectural shape.** "We're using a monorepo." "The write model is event-sourced, the read model is projected into Postgres."
- **Integration patterns between contexts.** "Ordering and Billing communicate via domain events, not synchronous HTTP."
- **Technology choices that carry lock-in.** Database, message bus, auth provider, deployment target — not every library, just the ones that would take a quarter to swap out.
- **Boundary and scope decisions.** "Customer data is owned by the Customer context; other contexts reference it by ID only." The explicit *no*s are as valuable as the *yes*es.
- **Deliberate deviations from the obvious path.** "We're using manual SQL instead of an ORM because X." Anything where a reasonable reader would assume the opposite — these stop the next engineer from "fixing" something deliberate.
- **Constraints not visible in the code.** "We can't use AWS because of compliance requirements." "Response times must be under 200ms because of the partner API contract."
- **Rejected alternatives when the rejection is non-obvious.** If you considered GraphQL and picked REST for subtle reasons, record it — otherwise someone will suggest GraphQL again in six months.

### Applying the gate at WIP vs. Accepted

The gate is applied **provisionally** when creating a WIP ADR and **definitively** when promoting to Accepted:

- **At WIP creation** the bar is **low** — two of the tests (surprising, real trade-off) often can't be fully judged until the alternatives have been explored, which may be exactly what's being deferred. If it's unclear, **err toward creating the WIP ADR.** A WIP ADR is a cheap, disposable hypothesis that a record is warranted.
- **At promotion** the gate is applied for real. If the resolved decision clears all three tests → `Status: Accepted`. If it turns out **not** to (the alternatives evaporated, it's trivially reversible, it was the obvious thing) → **delete the ADR.** Deletion is a normal, expected outcome, not a failure.

## Status lifecycle (the shared contract)

This is the contract other skills reference. The `Status` field takes exactly these values:

- **WIP** — a decision that has been *recognized* but not *resolved*. The ADR frames what needs deciding and the options in play, so it can be picked up later (e.g. by a `grill-me` deep dive). A `Status: WIP` ADR is a greppable signal of an open decision.
- **Accepted** — the decision is made and the record passes the three-test gate.
- **Superseded** — a later ADR replaces this one; note which (e.g. "Superseded by ADR-0012").

A single greppable token, `WIP`, lives in the `Status` field so `grep -rn "Status:.*WIP" docs/adr/` surfaces every open decision across the project. The pattern must tolerate the emphasis the template emits (`- **Status:** WIP`); a literal `Status: WIP` matches nothing. Promotion (WIP → Accepted) and deletion are always deliberate human-confirmed acts, never automatic.

## Location and numbering

ADRs live in `docs/adr/`, named `ADR-NNNN-<slug>.md` with a zero-padded sequential number (`ADR-0001-...`, `ADR-0002-...`) and a short kebab-case slug.

**This skill owns numbering.** To allocate the next number: list `docs/adr/`, find the highest existing `ADR-NNNN`, increment. Create `docs/adr/` if it doesn't exist. When another skill needs an ADR created, it defers to this convention rather than reimplementing it.

## The ADR index

`docs/adr/README.md` is a lightweight index so a human or LLM can grasp what decisions exist without reading every file — and, because it lists status, it doubles as the project's **open-decision dashboard** (every `WIP` row is an unresolved decision that `to-workplan` and `grill-me` care about).

Keep it **minimal** — number, title, status, and a one-line context. Resist extra columns (date, supersedes, related ADRs); they add maintenance burden and drift faster.

```markdown
# Architecture Decision Records

| ADR | Title | Status | Context |
|---|---|---|---|
| [ADR-0001](ADR-0001-event-sourced-write-model.md) | Event-sourced write model | Accepted | One line on what's being decided |
| [ADR-0002](ADR-0002-transport-framing.md) | Transport framing | WIP | One line on the open question |
```

**Maintenance (sync-on-write is the normal path):** whenever this skill creates an ADR, promotes it (WIP → Accepted), supersedes it, or deletes it, update the index in the same step — the skill is already in the directory to allocate the number. Create `README.md` lazily with the first ADR.

**Regenerable fallback:** the ADR files are authoritative; the index is a convenience cache, never a second source of truth. Because each ADR carries its number, title, and `Status` in predictable places, the index can be rebuilt from the files if it ever drifts (e.g. after a hand-edit that bypassed the skill). If you find the index inconsistent with the files, regenerate it from the files rather than trusting it.

## Template

ALWAYS use this structure. The same template serves both WIP and Accepted states — fields fill in as the decision matures; nothing is restructured on promotion.

```markdown
# ADR-NNNN: <title>

- **Status:** WIP | Accepted | Superseded
- **Date:** <YYYY-MM-DD>

## Context

<The forces at play and the question being decided. For a WIP ADR this is the most
important section: capture *what needs to be decided* and why it matters, in enough
detail that someone (or grill-me) can pick it up cold later. State constraints,
pressures, and the problem — not the answer.>

## Options Considered

<The candidate alternatives. In WIP state this is the live menu still to be explored;
in Accepted state it shows what was weighed and rejected. List each option with a
brief note on its trade-offs.>

## Decision

<WIP: leave empty, or write "Deferred — open because <reason>." (No need to record who
deferred it; the Context is what matters.)
Accepted: state the chosen option and the specific reasons it won over the
alternatives.>

## Consequences

<Accepted: what follows from this decision — new constraints, things now easier or
harder, follow-on work. WIP: may be empty.>
```

For a WIP ADR, fill **Context** and **Options Considered** richly and mark the **Decision** as deferred. Promotion just means completing **Decision** and **Consequences** and flipping the status.

## Relationship to other artifacts

ADRs record *decisions and rationale*. They are not the place for:

- **What the system is / how it's structured** — that's the Architectural Description (`docs/architecture/`, the `architecture-description` skill). A viewpoint file *references* the ADR where a decision shaped it; an ADR may note "Logical view to be updated to reflect this," but the architecture doc itself is edited there.
- **The external contract** (CLI flags, API, wire format) — that's user-facing documentation, written when the surface is built (a Work Package acceptance criterion). An ADR may capture *why* the contract is shaped as it is, not the contract itself.
- **Domain terminology** — that's the shared `docs/CONTEXT.md` (the `context-glossary` skill). A single decision (e.g. a boundary decision) commonly warrants *both* an ADR and glossary terms; record both. They are not mutually exclusive.

There is no `SPECIFICATION.md` in this toolchain — its roles are covered by ADRs (why), the Architectural Description (what/how), user docs (external contract), and code + tests (behavior).

## After writing

State the ADR path and its status, and update `docs/adr/README.md` accordingly. If you created a WIP ADR, make clear it represents an **open decision** awaiting resolution (a good candidate for a focused `grill-me` session), and that it will be deleted on resolution if it fails the three-test gate.
