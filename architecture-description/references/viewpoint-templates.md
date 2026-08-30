# Viewpoint File Templates

Structure conventions for each viewpoint file in `docs/architecture/`. This skill (`architecture-description`) owns *what each file contains*; the *interview guidance* for eliciting that content lives in the `explore-architecture` skill's own viewpoint references. Load this when creating or structuring a viewpoint file.

All viewpoint files: start with a one-line link to the shared glossary (`see ../CONTEXT.md`), keep sections focused and consistently headed, reference ADRs inline where a decision shaped the content (`see ADR-NNNN`), and record unresolved points as `[!OPEN]` callouts (also indexed in `index.md`). Never mark a viewpoint "Current" while significant open issues remain. Diagrams are out of scope — these are self-contained markdown descriptions consumable by text-based agents and by tools that generate diagrams from text.

---

## scenarios.md (Scenarios viewpoint, +1)

The thread that drives and validates the four structural viewpoints. Captures what the system must do, for whom, and why.

This viewpoint spans two record-set directories: `archetypes/` (one file per Archetype, carrying its Use Cases) and `scenarios/` (one file per Scenario). `scenarios.md` itself is the **selection surface** — it holds what is shared across Archetypes plus the indexes, and stays small.

```markdown
# Scenarios

> Terms: see [../CONTEXT.md](../CONTEXT.md).

## System Summary
<One paragraph: what the system does, for whom.>

## Non-Goals
<What this system deliberately does not do, and why. Shared across Archetypes,
so it lives here rather than in any one archetype file.>

## Archetype Index

| Archetype | Summary | File |
|---|---|---|
| Reviewer | <one line: who they are and what they want> | [archetypes/reviewer.md](archetypes/reviewer.md) |

## Use Case Index
<Every Use Case across all Archetypes, so a reader can find one without
knowing which Archetype owns it.>

| Use Case | Archetype | Architecturally Significant | File |
|---|---|---|---|
| Review a pending change | Reviewer | Yes | [archetypes/reviewer.md](archetypes/reviewer.md) |

## Scenario Index

| Scenario | Archetype | Use Case | File |
|---|---|---|---|
| SC-001 successful login | … | … | [scenarios/SC-001-successful-login.md](scenarios/SC-001-successful-login.md) |

## Cross-Viewpoint Validation
<What walking these scenarios confirmed or broke in the other viewpoints.>
```

### `archetypes/<name>.md`

One file per Archetype, slug lowercased from the Archetype name (no number — Archetypes are named). Each carries **its own Use Cases**: a Use Case's `Addresses:` line references Pain and Gain short names defined only in its Archetype's profile, so splitting them apart would leave dangling references and cost a read rather than save one.

```markdown
# {Archetype}

> Terms: see [../../CONTEXT.md](../../CONTEXT.md).

<A narrative paragraph — who they are and their operating environment: where
and when they use the system, who else sees the result, the workflow around it.>

**Drivers** — why they engage with the system at all:
- <The higher purpose or external pressure the system is merely a tool for.
  Never a system capability.>

**Pains**:
- **<Short name>**: <obstacle, cost, or risk in pursuing their Drivers;
  situational wording — "when presenting on-site…". Tag *(social)* or
  *(emotional)* when the dimension is not functional; functional is the
  unmarked default.>

**Gains**:
- **<Short name>**: <outcome they want beyond pain removal; same optional tag.>

<No minimum lengths — each list is as long as understanding warrants. The bold
short names are the trace handles Use Cases reference.>

## Use Cases

### {Use Case name}
**Goal**: <the Archetype's intent, not the mechanism.>
**Addresses**: <Driver/Pain/Gain short names from above. An empty trace is a
visibly unjustified Use Case.>
**Architecturally Significant**: <Yes/No — and why, leaning on the trace.>
**Preconditions**: <what must hold before this begins.>
**Scenarios**: <links into scenarios/, or "None yet.">
```

Individual scenarios live flat in `scenarios/SC-NNN-<slug>.md`, globally unique, permanent numbers (gaps never reused). Each describes one path through a Use Case as a sequence of boundary Interactions (Archetype→System or System→Archetype) — externally visible only, never internal mechanics.

`scenarios/` is a **sibling** of `archetypes/`, not nested inside it. Use Case → Scenario is a *trace* relation rather than composition: a Scenario is reachable by its global `SC-NNN` without knowing which Archetype owns its Use Case, and it survives the Use Case being restructured.

---

## logical.md (Logical viewpoint)

The system's conceptual structure — what it is made of and what each part is responsible for. Structured around two C4 levels: System Context (the system as one box, its Archetypes, its External Systems) and Runtime Containers (the independently runnable units inside the boundary).

```markdown
# Logical View

> Terms: see [../CONTEXT.md](../CONTEXT.md).

## System Context
<The system as a single box: its Archetypes and the External Systems it interacts with (name, owner, interaction direction, nature, criticality).>

## Runtime Containers
<Each independently runnable unit inside the boundary: responsibility (what it alone owns), headline technology, and its relationships to other containers and External Systems.>

## Relationships
<How containers and external systems relate.>
```

Every Runtime Container should participate in at least one architecturally significant scenario; containers in no scenario are candidates for removal.

---

## runtime.md (Runtime viewpoint)

How the logical building blocks combine and interact at runtime — concurrency, inter-process communication, dynamic behavior.

```markdown
# Runtime View

> Terms: see [../CONTEXT.md](../CONTEXT.md).

## Execution Model
<Processes / execution units and how they map to Runtime Containers; concurrency model.>

## Interactions
<How containers and external systems communicate at runtime — synchronous vs. event-driven, protocols, key flows. Tie to architecturally significant scenarios.>

## Failure & Timing
<Failure modes, retries, timeouts, ordering/consistency concerns that are architecturally significant.>
```

---

## deployment.md (Deployment viewpoint)

The infrastructure and topology — where elements run and how they're distributed.

```markdown
# Deployment View

> Terms: see [../CONTEXT.md](../CONTEXT.md).

## Topology
<Environments and nodes; how Runtime Containers map onto infrastructure.>

## Distribution & Scaling
<Replication, scaling, data locality, network boundaries.>

## Operational Constraints
<Compliance, region, latency, or other deployment constraints not visible in code.>
```

---

## development.md (Development viewpoint)

How the system is organized for development — modules, components, layers, packages, build and dependency structure. In a greenfield project this view is *prescriptive*: it designs the structure the first increment instantiates, rather than describing existing code.

This template has a required core, one conditionally required section, and an optional menu. Include an optional section only when its trigger applies — an empty section is noise.

**Core (always present):**

```markdown
# Development View

> Terms: see [../CONTEXT.md](../CONTEXT.md).

## Overview
<One paragraph orienting a new developer or coding agent: the dominant
organizational pattern, the major structural units, and — greenfield —
whether this view describes or prescribes.>

## Code Organization
<Modules / components / layers and how they map to Runtime Containers.>

## Dependency Structure
<Allowed dependency directions; layering rules; build units.>

## Conventions
<Architecturally significant development conventions — only those with structural impact.>
```

**Conditionally required — mandatory whenever the project has any external dependency** (absent only in zero-dependency projects):

```markdown
## External Dependencies
<Convention statement: every external dependency gets a recorded evaluation
*before* adoption — liability plus supply chain — per the dependency
evaluation reference (references/dependency-evaluation.md in the
architecture-description skill).>

| Dependency | Scope | Verdict | Date |
|---|---|---|---|
| [Electron](dependencies/electron.md) | Shipped | Accepted | 2026-08-29 |

<Plus any paragraphs naming which ADRs decided the liability half.>
```

The full evaluations do **not** live here. Each dependency gets its own file in `dependencies/` — a record set, per the two gates in `SKILL.md`. This section keeps the convention and the selection table and links out; without that split, the evaluations grow to dominate the viewpoint and an agent must load all of them to read any of them. Evidence (a sweep report, a survey) goes in `dependencies/sweep-report.md`, not here.

Once this section exists, adopting a dependency without a recorded evaluation is a convention violation, not an oversight.

**Optional menu:**

```markdown
## Seams & Extension Points
<Include when the risk register or Logical view names elements that must stay
swappable or independently evolvable. One entry per seam: what representation
crosses it and who owns that representation, what must never cross it, and
what enforces it (compiler / build rule / discipline).>

## Test Strategy
<Include when test structure is architecturally significant. Note whether
test weight follows the stability gradient: spec-grade suites on frozen
contracts, ordinary coverage on churny internals.>

## First-Increment Instantiation
<Greenfield only. What the first roadmap increment instantiates of the
prescribed structure — the walking skeleton that makes every seam real.>
```
