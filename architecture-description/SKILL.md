---
name: architecture-description
description: Owns the Architectural Description (AD) — the durable, living documentation of a system's architecture under docs/architecture/, structured with the modernized Kruchten 4+1 viewpoint model (Scenarios, Logical, Runtime, Deployment, Development). Defines the file/index structure, viewpoint file conventions, the risk register, and the [!OPEN] open-issue convention that other skills reference. This is the CONTENT skill (what the AD is and where it lives); the elicitation of architecture content is done by explore-architecture (deliberate risk-driven sessions) and grill-work (incidental touches while scoping a chunk), both of which write into the structure defined here. Use this skill when reading, structuring, or directly editing architecture documentation, when another skill needs to know where architecture content lives or how an open issue is recorded, or when the user asks to set up or navigate docs/architecture/. For driving an architecture conversation, use explore-architecture.
---

# architecture-description

The **Architectural Description (AD)** is the durable, living record of a system's architecture: the best current understanding of what the system is and how it is structured, subject to revision as new information surfaces. This skill owns the AD as an *artifact* — its files, structure, and conventions. It does not drive the conversation that fills it (that's `explore-architecture` for deliberate sessions and `grill-work` for incidental touches); it defines the thing both write into and that other skills read.

Read `references/vocabulary.md` before working with architecture content — it defines the methodology terms (Viewpoint, View, Archetype, Runtime Container, Risk, etc.) that govern how architecture is described. That file is skill-internal reference, **not** a project artifact and never copied into `docs/`.

## What the AD is — and is not

- It **is** the durable "what the system is and how it's structured" — the role a `SPECIFICATION.md` used to pretend to fill. There is no separate spec; the AD (plus ADRs for *why*, user docs for the *external contract*, and code+tests for *behavior*) covers it.
- It is **living**, never "done." A viewpoint is "Current" when its risks are understood and either mitigated or captured as open issues — not when it's complete.
- It is **not** a decision log — decisions and their rationale are ADRs (the `adr` skill). Viewpoint files *reference* ADRs where relevant.
- It is **not** the domain glossary — terminology lives in the one shared `docs/CONTEXT.md` (the `context-glossary` skill). The AD references it; it does not maintain its own.
- It is **not** an execution plan — building work is Work Plans / Work Packages (transient, in `.scratch/`).

## Document structure

The AD lives under `docs/architecture/` (durable, committed — a peer of `docs/adr/`, `docs/CONTEXT.md`, `docs/ROADMAP.md`, not privileged above them):

```
docs/
├── CONTEXT.md                    # shared domain glossary (context-glossary skill) — referenced, not owned here
├── ROADMAP.md                    # backlog (roadmap skill)
├── adr/                          # ADRs (adr skill)
└── architecture/                 # the Architectural Description (this skill)
    ├── index.md                  # root: system summary, viewpoint map, risk register, open-issues summary, session log
    ├── scenarios.md              # Scenarios viewpoint (+1): Archetypes, Use Cases, navigation index
    ├── scenarios/                # individual Scenario files, globally unique SC-NNN
    │   └── SC-001-*.md
    ├── logical.md                # Logical viewpoint
    ├── runtime.md                # Runtime viewpoint
    ├── deployment.md             # Deployment viewpoint
    └── development.md            # Development viewpoint
```

The AD is **optional and grows on demand**: a project need not have one (early prototypes usually won't). Create `docs/architecture/` and `index.md` when architecture work actually begins; add viewpoint files as each viewpoint is first explored. Each view file is self-contained and navigable without the others; cross-references use relative links. The structure is meant to be consumable by coding agents with limited context — focused sections, consistent headings, key decisions never buried in prose.

For the full `index.md` template and conventions (system summary, viewpoint map, risk register, open-issues summary, session log, viewpoint status levels, risk severity), see `references/index-structure.md`.

## The five viewpoints

| Viewpoint | File | Primary question |
|---|---|---|
| **Scenarios** (+1) | `scenarios.md` (+ `scenarios/SC-NNN-*.md`) | What must the system do, for whom, and why? |
| **Logical** | `logical.md` | What are the key concepts, responsibilities, and boundaries? |
| **Runtime** | `runtime.md` | How do elements interact and behave at runtime? |
| **Deployment** | `deployment.md` | Where does the system run and how is it deployed? |
| **Development** | `development.md` | How is the system organized for development? |

This is not a waterfall; work in one viewpoint surfaces questions in others. Scenarios are the +1 thread that validates cross-viewpoint consistency. The amount of architecture work is **risk-driven** — do just enough, focused on what could cause the system to fail. The structure/template of each viewpoint file is defined in `references/viewpoint-templates.md` (load when creating or structuring a viewpoint file). The external-dependency evaluation convention (liability rubric + supply-chain posture, recorded in the Development view) is defined in `references/dependency-evaluation.md` (load when adopting or reviewing an external dependency). The interview guidance for *eliciting* each viewpoint's content lives in `explore-architecture`; this skill defines what each file *contains*.

## The `[!OPEN]` open-issue convention (owned here)

Open issues — concerns, questions, or risks identified but not yet resolved — are recorded inline in the relevant viewpoint file using this callout, and indexed in `index.md`'s open-issues summary:

```markdown
> [!OPEN]
> **Issue**: {one-line description of what is unknown or unresolved}
> **Risk**: {why this matters architecturally — what could go wrong}
> **Owner**: {who should resolve this, if known — omit if unknown}
```

When an issue is resolved, remove it from the view file and record it as closed in the `index.md` session log.

**This convention is referenced by other skills**, analogous to how `Status: WIP` is owned by the `adr` skill:
- `grill-work` and implementation work raise `[!OPEN]` issues when they discover the AD has drifted from reality (the architecture no longer matches what's being built). They **flag**; they do not fix inline. Resolution happens in an `explore-architecture` session.
- This is the downstream feedback path: implementation discovers drift → `[!OPEN]` flag → resolved deliberately later.

## Relationship to the rest of the toolchain

- **Elicitation** — `explore-architecture` (deliberate, risk-driven, cross-viewpoint sessions) and `grill-work` (incidental architecture facts noticed while scoping a chunk) both write into this structure. `grill-work` captures what's shallow and defers anything architecturally deep to an `explore-architecture` session.
- **Decisions** — when architecture work yields a decision meeting the three-test gate, it's an ADR via the `adr` skill; the viewpoint file references it. This skill does not define its own ADR format.
- **Terminology** — domain terms surfaced during architecture work go to the shared `docs/CONTEXT.md` via `context-glossary`. Project-facing architecture terms (Runtime Container, External System, Archetype, Stakeholder, Quality Property, Architecturally Significant) are seeded into that glossary *as a project actually uses them* — not dumped wholesale from the methodology vocabulary.
- **Planning (upstream)** — `to-workplan` *optionally* reads the scope-relevant viewpoint files and records the constraints they impose as a Work Plan "Constraints" section; Work Packages inherit those boundaries. The AD is the durable context a chunk is planned against.
- **Planning (downstream)** — implementation that contradicts the AD raises an `[!OPEN]` issue back here.

## When directly editing the AD

This skill is also used for direct, non-interview edits — fixing structure, navigating, or applying a confirmed change. Preserve the conventions above: keep `index.md` in sync (viewpoint map status, risk register, open-issues summary, session log), keep view files self-contained, reference ADRs and `docs/CONTEXT.md` rather than duplicating them, and never mark a viewpoint "Current" while significant open issues remain.
