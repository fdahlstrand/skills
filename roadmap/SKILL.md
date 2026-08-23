---
name: roadmap
description: Create and maintain ROADMAP.md, a persistent, high-level backlog of the work ahead (and a record of work behind). Entries are prose describing chunks of future work, tracked with checkboxes and an optional WIP marker, optionally grouped into theme sections. Use when the user wants to capture future work, sketch the path ahead, add or update a roadmap item, mark a chunk done, or organize the backlog into themes — or says "add to the roadmap", "what's next", "update the roadmap", "roadmap this". The roadmap is optional; many projects start from a single Work Plan and only grow a roadmap once there's enough road ahead to index. Do NOT use this for the detailed plan of one chunk (that's to-workplan) or for decisions (that's the adr skill).
---

# roadmap

`docs/ROADMAP.md` is the project's **high-level backlog**: a list of chunks of work to be done, described well enough that a human or an LLM can pick one up and turn it into a Work Plan. It describes the **path ahead** and, as work completes, also becomes a record of the **path behind**.

## Resolving the docs root

This skill writes documentation paths relative to the project's **docs root**. Resolve it before reading or writing any of them. It is either a `docs/` folder at the project root, or a sibling `docs/` repository beside the source repository. When the repositories are siblings and you are running from another one, the docs root is `../docs/`.

## Optional and persistent

Two defining properties:

- **Optional.** The roadmap is *not* a mandatory artifact. Projects often start from a single Work Plan to get a prototype off the ground, with only a loose idea of what comes next, and never need a roadmap until there's enough road ahead to be worth indexing. Create `docs/ROADMAP.md` when that point arrives — don't manufacture one prematurely.
- **Persistent.** Unlike Work Plans (scoped to one chunk, in `docs/plan/`), `docs/ROADMAP.md` is a durable, committed document. It lives at `docs/ROADMAP.md` — a peer of `docs/CONTEXT.md`, `docs/adr/`, `docs/architecture/`, and `docs/plan/`, not privileged at repo root. It **outlives** the Work Plans it mentions: a plan is spun up, executed, and deleted, but its roadmap entry persists and gets checked off.

## What the roadmap is NOT

- Not an execution script. It does **not** carry per-chunk deliverables, file lists, acceptance criteria, or validation steps — that detail belongs in the Work Plan for each chunk (`to-workplan`). Keeping execution detail here is the failure mode that makes a roadmap try to be two things at once; resist it.
- Not aware of Work Plans. It does **not** hard-link to plans, hold back-references, or track whether a plan exists. An entry is just prose; the matching `docs/plan/<slug>/` plan may not exist yet, may exist, or may have been created and deleted. The roadmap neither knows nor cares.
- Not a decision log (that's the `adr` skill) and not a glossary (that's `context-glossary`).

## Entries are prose

Each entry is a short prose description of a chunk of future work — enough that a reader (human or LLM) can **reasonably identify or create the corresponding Work Plan** from the entry's name/description. No required fields, no structured schema, no slug ceremony. "Event-sourced write model for the ordering context" is a complete, usable entry.

## Entry state: checkbox + WIP

Reuse the toolchain's checkbox + `WIP` idiom (the same convention as Work Plans, ADRs, and the glossary), so there is one mental model across every artifact:

- `- [ ] <prose>` — firm future work on the road ahead.
- `- [ ] <prose> (WIP)` — a candidate with *elevated uncertainty about this entry specifically* (beyond the normal "it's in the future"). This is what `grill-work` drops when it surfaces possible future work that isn't yet firm. The `WIP` token is greppable, consistent with the rest of the toolchain.
- `- [x] <prose>` — done.

### Keep history; prune only the abandoned

- **Done entries are kept**, checked off. The roadmap is deliberately a record of where the project has been, not just where it's going. Never prune completed work.
- **Removal is only for entries that were abandoned** — ideas that no longer make sense and were *never done*. Those were never part of the road, so they leave no trace. (This is the one case of editing history, and it's honest: a never-travelled path.)

Because completed entries accumulate permanently, the document grows over a long project — which is exactly when theme grouping (below) earns its keep.

## Themes (optional grouping)

Group entries under `##` theme sections to steer strategic direction and keep a growing backlog navigable. Themes are a structural tool, not a requirement:

- If the backlog is small and cohesive, a **flat list** is fine.
- As entries accumulate (especially the growing history of done items), introduce **theme sections** when natural clusters emerge.

This is the same default-flat, cluster-when-it-helps principle the `context-glossary` skill uses for `CONTEXT.md`.

## Format

Flat:

```markdown
# Roadmap

The path ahead (and behind) for <project>. High-level backlog; each entry becomes a
Work Plan when picked up.

- [x] draw.io parser and emitter with round-trip tests
- [ ] Layout engine: hierarchical (Sugiyama) layout
- [ ] Scheme interpreter for constraints and style expressions
- [ ] DSL lexer and parser (WIP) — shape still uncertain, needs a grill-me on the meta-model layering
```

Themed:

```markdown
# Roadmap

The path ahead (and behind) for <project>.

## Rendering pipeline
- [x] draw.io parser and emitter with round-trip tests
- [ ] Layout engine: hierarchical (Sugiyama) layout

## Modelling language
- [ ] Scheme interpreter for constraints and style expressions
- [ ] DSL lexer and parser (WIP) — shape still uncertain
```

## Relationship to the rest of the toolchain

- **`grill-work`** writes candidates here — future work surfaced while scoping a chunk, including deferred spec-change work (a roadmap candidate is the future Work Plan that will revise the spec). New candidates from grilling are typically `(WIP)`.
- **`to-workplan`** treats a roadmap entry as the *chunk* it elaborates into a Work Plan. The link is by recognizable prose/name, not a formal pointer.
- The roadmap is **durable**; the `docs/plan/` Work Plans it references are **chunk-scoped**. The roadmap survives them.

## After writing

State what changed (entries added, checked off, themed, or removed). If you created `docs/ROADMAP.md` for the first time, note that it's a persistent committed artifact under `docs/` — distinct from the chunk-scoped plans in `docs/plan/`.
