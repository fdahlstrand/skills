# Index Structure Reference

Defines the structure and content conventions for `docs/architecture/index.md` — the root file of the Architectural Description. Load when creating or updating the index.

---

## Purpose of the index

`index.md` serves three audiences:
1. **Humans** — orientation at the start of a session; understanding of current state.
2. **Coding agents** — a map to find relevant architectural detail without loading all files.
3. **Elicitation skills** — session resumption without reading every view file.

It must be navigable in isolation: every section useful without the other view files. It is a **map**, not a destination — it stays small enough that reading it is never a reason to skip reading it.

---

## File template

```markdown
# Architecture Index

> **Living document.** Best current understanding of the architecture of {System Name}.
> Expected to change as new information surfaces.
> **AD revision: 2**

## Current State

**As of**: {YYYY-MM-DD}

{One or two sentences: what the architecture has settled and what it has not.
Written so an agent picking this up cold knows whether the ground is firm.}

**Next**: {a specific viewpoint, risk, or chunk — and why it is next.}

## System

{One paragraph: what this system is, who it is for, what problem it solves. Written for a coding agent or new team member encountering it for the first time.}

## Terminology

Domain terms are defined in the shared project glossary [../CONTEXT.md](../CONTEXT.md) (owned by the context-glossary skill). All code, documentation, and conversations about this system use the terms defined there; avoid-terms are listed explicitly. The architecture methodology vocabulary (Viewpoint, Archetype, Runtime Container, etc.) is defined by the architecture-description skill's internal reference, not duplicated here.

## Viewpoint Map

| Viewpoint | Entry Point | Status | Last Updated |
|---|---|---|---|
| Scenarios (+1) | [scenarios.md](scenarios.md) | {Not started / In progress / Current} | {date or —} |
| Logical | [logical.md](logical.md) | {Not started / In progress / Current} | {date or —} |
| Runtime | [runtime.md](runtime.md) | {Not started / In progress / Current} | {date or —} |
| Deployment | [deployment.md](deployment.md) | {Not started / In progress / Current} | {date or —} |
| Development | [development.md](development.md) | {Not started / In progress / Current} | {date or —} |

Note: two viewpoints span a directory as well as a file. Scenarios: `scenarios.md` indexes the Archetypes in `archetypes/` and the Scenarios in `scenarios/`. Development: `development.md` indexes the external dependencies in `dependencies/`. Read the viewpoint file first — it holds the selection table.

## Risk Register

Active architectural risks across all viewpoints. Engineering risks — concerns that, if unaddressed, could cause the system to fail in architecturally significant ways.

| # | Risk | Viewpoint(s) | Severity | Status |
|---|---|---|---|---|
| R-001 | {description} | {viewpoint} | {High / Medium / Low} | {Open / Mitigated / Closed} |

A closed or mitigated risk keeps its row and carries its closure rationale inline. That rationale is the durable record of *why* it closed — it lives here and nowhere else.

## Open Issues Summary

Cross-viewpoint navigation index of open issues. Full detail lives as an `[!OPEN]` callout in the file that holds it.

| Issue | Viewpoint | File |
|---|---|---|
| {one-line description} | {owning viewpoint} | [{path}]({path}) |

The **File** column names the file that actually holds the callout, which may be a record file (`archetypes/reviewer.md`, `dependencies/electron.md`), not only a viewpoint file. The **Viewpoint** column names the owning viewpoint regardless.
```

---

## Conventions

### Current State
- **One block, overwritten every session. Never appended to.** The moment it grows a second dated entry it has become a session log, which is the thing revision 2 removed.
- It carries only what has no other home. `Explored` is the Viewpoint Map's Last Updated column; `Decisions` are ADRs; `Opened` is the Open Issues Summary; `Closed` is the Risk Register's own closure rationale; terminology is `CONTEXT.md`. **`Next` is the only field with nowhere else to live.**
- `As of` makes staleness visible. A `Next` under a three-month-old date reads as stale rather than current, which is exactly right.
- History is the git log of `docs/architecture/`. The AD keeps no log of its own.

### Viewpoint status
- **Not started** — no file, or empty.
- **In progress** — exploration begun; open issues remain; not yet a reliable reference.
- **Current** — sufficiently explored for current risk level; open issues documented; safe to reference. "Current" ≠ complete; it means current best understanding is recorded and associated risks are mitigated or captured.

### Risk severity
Assess by: *what is the impact if this risk materializes and we haven't addressed it architecturally?*
- **High** — system cannot be built, or must be fundamentally redesigned.
- **Medium** — significant rework of one or more viewpoints required.
- **Low** — addressable locally without structural change.

### The revision marker
`**AD revision: N**` declares which structural revision this AD follows. Absence means revision 1. An agent whose skill declares a higher revision migrates first — see `migration.md`.

---

## Coding agent guidance

When an agent reads `index.md` to understand the system:
- Read **Current State** first — it says whether the ground is firm and what is in flight.
- Read [../CONTEXT.md](../CONTEXT.md) — the shared domain glossary defining terms used throughout.
- Check the ADRs in `../adr/` to understand key decisions and rationale before making changes.
- Use **Viewpoint Map** to find the right file for the concern at hand, then its selection table to reach the right record file. Do not load a whole viewpoint to find one record.
- Check **Open Issues Summary** for what is known to be unresolved.
- Don't assume absence of documentation means absence of a decision — check `../adr/` and the git history of `docs/architecture/`.

---

## Maintenance rules

- Update `index.md` at the end of every session: **Current State** rewritten, Viewpoint Map status and dates, Risk Register, Open Issues Summary.
- Domain terms go to the shared `docs/CONTEXT.md` via context-glossary — do not maintain a separate architecture glossary.
- Every `[!OPEN]` callout anywhere under `docs/architecture/` — viewpoint files and record files alike — must appear in **Open Issues Summary**, with the **File** column pointing at the file that holds it.
- A resolved issue is removed from its file and its row from the summary. The git history records that it closed.
- **Viewpoint Map** status must reflect actual file state — do not mark "Current" while significant open issues remain.
- `index.md` stays a map. If a section is growing without bound, it is content that belongs in a viewpoint or a record file, or it is a log that should not exist.
