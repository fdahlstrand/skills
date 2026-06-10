# Viewpoint File Templates

Structure conventions for each viewpoint file in `docs/architecture/`. This skill (`architecture-description`) owns *what each file contains*; the *interview guidance* for eliciting that content lives in the `explore-architecture` skill's own viewpoint references. Load this when creating or structuring a viewpoint file.

All viewpoint files: start with a one-line link to the shared glossary (`see ../CONTEXT.md`), keep sections focused and consistently headed, reference ADRs inline where a decision shaped the content (`see ADR-NNNN`), and record unresolved points as `[!OPEN]` callouts (also indexed in `index.md`). Never mark a viewpoint "Current" while significant open issues remain. Diagrams are out of scope — these are self-contained markdown descriptions consumable by text-based agents and by tools that generate diagrams from text.

---

## scenarios.md (Scenarios viewpoint, +1)

The thread that drives and validates the four structural viewpoints. Captures what the system must do, for whom, and why.

```markdown
# Scenarios

> Terms: see [../CONTEXT.md](../CONTEXT.md).

## Archetypes
<Each named class of human (or, where modelled, external actor) that interacts with the running system. Name + one-line description. Identified by name, no slug.>

## Use Cases
<Per Archetype: the goals that Archetype pursues. Each Use Case belongs to exactly one Archetype and names intent, not mechanism.>

## Scenario Index
<Navigation table to the individual scenario files in scenarios/.>

| Scenario | Archetype | Use Case | File |
|---|---|---|---|
| SC-001 successful login | … | … | [SC-001-successful-login.md](scenarios/SC-001-successful-login.md) |
```

Individual scenarios live flat in `scenarios/SC-NNN-<slug>.md`, globally unique, permanent numbers (gaps never reused). Each describes one path through a Use Case as a sequence of boundary Interactions (Archetype→System or System→Archetype) — externally visible only, never internal mechanics.

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

How the system is organized for development — modules, components, layers, packages, build and dependency structure.

```markdown
# Development View

> Terms: see [../CONTEXT.md](../CONTEXT.md).

## Code Organization
<Modules / components / layers and how they map to Runtime Containers.>

## Dependency Structure
<Allowed dependency directions; layering rules; build units.>

## Conventions
<Architecturally significant development conventions — only those with structural impact.>
```
