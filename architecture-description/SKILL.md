---
name: architecture-description
description: Owns the Architectural Description (AD) — the durable, living documentation of a system's architecture under docs/architecture/, structured with the modernized Kruchten 4+1 viewpoint model (Scenarios, Logical, Runtime, Deployment, Development). Defines the file/index structure, viewpoint file conventions, the record-set rule that keeps files small enough for progressive disclosure, the risk register, and the [!OPEN] open-issue convention that other skills reference. This is the CONTENT skill (what the AD is and where it lives); the elicitation of architecture content is done by explore-architecture (deliberate risk-driven sessions) and grill-work (incidental touches while scoping a chunk), both of which write into the structure defined here. Use this skill when reading, structuring, or directly editing architecture documentation, when another skill needs to know where architecture content lives or how an open issue is recorded, or when the user asks to set up or navigate docs/architecture/. For driving an architecture conversation, use explore-architecture.
---

# architecture-description

The **Architectural Description (AD)** is the durable, living record of a system's architecture: the best current understanding of what the system is and how it is structured, subject to revision as new information surfaces. This skill owns the AD as an *artifact* — its files, structure, and conventions. It does not drive the conversation that fills it (that's `explore-architecture` for deliberate sessions and `grill-work` for incidental touches); it defines the thing both write into and that other skills read.

Read `references/vocabulary.md` before working with architecture content — it defines the methodology terms (Viewpoint, View, Archetype, Runtime Container, Risk, Record Set, etc.) that govern how architecture is described. That file is skill-internal reference, **not** a project artifact and never copied into `docs/`.

## AD structure revision: 2

`index.md` declares the revision its structure follows. **If the AD's declared revision is lower than 2, or the marker is absent (absence means revision 1), read `references/migration.md` and migrate before making other edits.** That file also defines what earns a revision bump, for anyone changing this skill.

## What the AD is — and is not

- It **is** the durable "what the system is and how it's structured" — the role a `SPECIFICATION.md` used to pretend to fill. There is no separate spec; the AD (plus ADRs for *why*, user docs for the *external contract*, and code+tests for *behavior*) covers it.
- It is **living**, never "done." A viewpoint is "Current" when its risks are understood and either mitigated or captured as open issues — not when it's complete.
- It is **not** a decision log — decisions and their rationale are ADRs (the `adr` skill). Viewpoint files *reference* ADRs where relevant.
- It is **not** the domain glossary — terminology lives in the one shared `docs/CONTEXT.md` (the `context-glossary` skill). The AD references it; it does not maintain its own.
- It is **not** an execution plan — building work is Work Plans / Work Packages (in `docs/plan/`).
- It is **not** a history. What happened is the git history of `docs/architecture/`; the AD records what *is*.

## Document structure

The AD lives under `docs/architecture/` (durable, committed — a peer of `docs/adr/`, `docs/CONTEXT.md`, `docs/ROADMAP.md`, `docs/plan/`, not privileged above them):

```
docs/
├── CONTEXT.md                    # shared domain glossary (context-glossary skill) — referenced, not owned here
├── ROADMAP.md                    # backlog (roadmap skill)
├── adr/                          # ADRs (adr skill)
└── architecture/                 # the Architectural Description (this skill)
    ├── index.md                  # root: revision marker, current state, system summary,
    │                             #   viewpoint map, risk register, open-issues summary
    ├── scenarios.md              # Scenarios viewpoint (+1): non-goals, indexes, cross-viewpoint validation
    ├── archetypes/               # record set: one file per Archetype, carrying its Use Cases
    │   └── reviewer.md
    ├── scenarios/                # record set: individual Scenarios, globally unique SC-NNN
    │   └── SC-001-*.md
    ├── logical.md                # Logical viewpoint
    ├── runtime.md                # Runtime viewpoint
    ├── deployment.md             # Deployment viewpoint
    ├── development.md            # Development viewpoint
    └── dependencies/             # record set: one file per external dependency
        ├── electron.md
        └── sweep-report.md       # evidence, superseded in place
```

The AD is **optional and grows on demand**: a project need not have one (early prototypes usually won't). Create `docs/architecture/` and `index.md` when architecture work actually begins; add viewpoint files as each viewpoint is first explored. Each view file is self-contained and navigable without the others; cross-references use relative links.

For the full `index.md` template and conventions, see `references/index-structure.md`. For each viewpoint file's structure, see `references/viewpoint-templates.md`.

## Keeping files small enough to read: two gates

The AD is consumed by agents with limited context. A viewpoint file an agent must load whole to find one fact has failed, however good its prose. Two gates keep that from happening. Apply them **before** writing anything into the AD, in this order.

### Gate 1 — Non-redundancy

**Does this content already have a durable home?** The git history of `docs/architecture/`, an ADR, the Risk Register, `docs/CONTEXT.md`, or a viewpoint section. If so it does not belong in the AD *at all* — not in a smaller file, not in an appendix. Splitting redundant content only makes the duplication tidier, and every extra copy is one more thing that drifts.

The corollary: content with no home yet must be *given* one. A rejected alternative goes where the alternative would have lived — the viewpoint section, or the ADR that settled the choice. It never lives only in a narrative.

### Gate 2 — The record-set test

A **record set** is a section whose contents accrete indefinitely as one independently-readable record per subject. A section is a record set only when **all three** hold:

1. **Unbounded accretion** — records are added indefinitely as work proceeds; the set has no natural ceiling.
2. **Keyed access** — a reader normally wants *one named record*, and can pick it from the filename without opening it.
3. **Bodied** — each record is multi-paragraph and stands alone; it is not a table row.

All three, or it stays inline. Two of three is not enough — that is what stops a Risk Register (bounded, read whole, row-shaped) from being shredded into unreadable fragments.

**A record set does not live inside a viewpoint file.** It gets its own directory, named for the record type, one level under `docs/architecture/`, from the *first* record — not once it grows. The layout is then identical at record 1 and record 40, so no AD ever needs reorganizing, and no link ever breaks.

Known record sets: **Archetypes** (`archetypes/`), **Scenarios** (`scenarios/`), **external dependencies** (`dependencies/`).

### The parent/child contract

The viewpoint file keeps the **convention** (the rule the records enforce) and a **selection table** (enough to choose a record, or to decide none is needed), and links out. The directory holds one file per record, plus any evidence files the records depend on.

### Evidence is superseded in place

Only durable claims accrete. Evidence — a dependency sweep, a survey, a benchmark — describes *now*, and is organized by subject and replaced wholesale when re-run, never appended to by date. An evidence file that grows a new dated section per run has become a log, which is the failure Gate 1 exists to prevent.

Size is not a trigger. A 25 KB viewpoint file with no record set in it is correctly shaped, and splitting it into cross-referential fragments would cost reads rather than save them.

These gates govern `docs/architecture/`. They do not govern this skill's own reference files.

## The five viewpoints

| Viewpoint | File | Primary question |
|---|---|---|
| **Scenarios** (+1) | `scenarios.md` (+ `archetypes/`, `scenarios/`) | What must the system do, for whom, and why? |
| **Logical** | `logical.md` | What are the key concepts, responsibilities, and boundaries? |
| **Runtime** | `runtime.md` | How do elements interact and behave at runtime? |
| **Deployment** | `deployment.md` | Where does the system run and how is it deployed? |
| **Development** | `development.md` (+ `dependencies/`) | How is the system organized for development? |

This is not a waterfall; work in one viewpoint surfaces questions in others. Scenarios are the +1 thread that validates cross-viewpoint consistency. The amount of architecture work is **risk-driven** — do just enough, focused on what could cause the system to fail. The structure/template of each viewpoint file is defined in `references/viewpoint-templates.md` (load when creating or structuring a viewpoint file). The external-dependency evaluation convention (liability rubric + supply-chain posture, recorded in `dependencies/`) is defined in `references/dependency-evaluation.md` (load when adopting or reviewing an external dependency). The interview guidance for *eliciting* each viewpoint's content lives in `explore-architecture`; this skill defines what each file *contains*.

## The `[!OPEN]` open-issue convention (owned here)

Open issues — concerns, questions, or risks identified but not yet resolved — are recorded inline in the file the issue is *about*, using this callout, and indexed in `index.md`'s open-issues summary:

```markdown
> [!OPEN]
> **Issue**: {one-line description of what is unknown or unresolved}
> **Risk**: {why this matters architecturally — what could go wrong}
> **Owner**: {who should resolve this, if known — omit if unknown}
```

**Record files are eligible.** An issue about one Archetype's Gains belongs beside those Gains, in `archetypes/<name>.md`, not hoisted into `scenarios.md` — a record file with a known hole that does not say so misleads anyone who opens it alone. The open-issues summary indexes by **file**, with the owning viewpoint as a second column.

**Evidence files are not eligible.** An `[!OPEN]` in a sweep report or survey vanishes silently the next time that evidence is superseded. An issue such evidence raises belongs in the viewpoint or the record, both of which are durable.

When an issue is resolved, remove it from the file and remove its row from the summary. The git history records that it closed.

**This convention is referenced by other skills**, analogous to how `Status: WIP` is owned by the `adr` skill:
- `grill-work` and implementation work raise `[!OPEN]` issues when they discover the AD has drifted from reality (the architecture no longer matches what's being built). They **flag**; they do not fix inline. Resolution happens in an `explore-architecture` session.
- This is the downstream feedback path: implementation discovers drift → `[!OPEN]` flag → resolved deliberately later.

## Relationship to the rest of the toolchain

- **Elicitation** — `explore-architecture` (deliberate, risk-driven, cross-viewpoint sessions) and `grill-work` (incidental architecture facts noticed while scoping a chunk) both write into this structure. `grill-work` captures what's shallow and defers anything architecturally deep to an `explore-architecture` session.
- **Decisions** — when architecture work yields a decision meeting the three-test gate, it's an ADR via the `adr` skill; the viewpoint file references it. This skill does not define its own ADR format.
- **Terminology** — domain terms surfaced during architecture work go to the shared `docs/CONTEXT.md` via `context-glossary`. Project-facing architecture terms (Runtime Container, External System, Archetype, Stakeholder, Quality Property, Architecturally Significant) are seeded into that glossary *as a project actually uses them* — not dumped wholesale from the methodology vocabulary.
- **History** — the git history of `docs/architecture/` is the AD's session record. The AD carries no log of its own; it carries `## Current State`, overwritten each session. This assumes commits that say what changed and why — where they don't, the fix is commit hygiene, not a parallel log.
- **Planning (upstream)** — `to-workplan` *optionally* reads the scope-relevant viewpoint files and records the constraints they impose as a Work Plan "Constraints" section; Work Packages inherit those boundaries. The AD is the durable context a chunk is planned against.
- **Planning (downstream)** — implementation that contradicts the AD raises an `[!OPEN]` issue back here.

## When directly editing the AD

This skill is also used for direct, non-interview edits — fixing structure, navigating, or applying a confirmed change. Preserve the conventions above: check the revision marker before editing, run both gates before adding content, keep `index.md` in sync (current state, viewpoint map status, risk register, open-issues summary), keep view files self-contained, reference ADRs and `docs/CONTEXT.md` rather than duplicating them, and never mark a viewpoint "Current" while significant open issues remain.
