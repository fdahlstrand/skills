---
name: grill-work
description: Interrogate a chunk of work (a roadmap entry / increment) before planning it, capturing decisions, terminology, and architecture facts as you go. Like grill-me but aimed at a concrete development chunk; it harvests ADRs, domain glossary terms, roadmap candidates, and incidental architecture-description updates during the session without derailing it. Use when the user says "let's work through the next increment", "let's plan for the next item in the roadmap", "grill this chunk of work", "let's scope the next feature", or similar phrasing about taking on a defined piece of development work. For open-ended research grilling not tied to a development chunk, use grill-me instead; for deliberate architecture sessions, use explore-architecture. This skill feeds the to-workplan / to-workpackages pipeline.
---

# grill-work

Interrogate a single chunk of work — a roadmap entry, increment, or feature — until you and the user share a clear understanding of what it is, what it decides, and what it's called. Unlike `grill-me` (general, open-ended research), `grill-work` is aimed at a concrete development chunk and **captures durable material as it goes**: Architecture Decision Records, domain glossary terms, and roadmap candidates. It's the usual front door to the `to-workplan` → `to-workpackages` pipeline.

## How to grill

Interview the user relentlessly about the chunk, one question at a time, walking down each branch of the decision tree and resolving dependencies between decisions. For each question, provide your recommended answer. If a question can be answered by exploring the codebase, explore the codebase instead of asking.

The job is to **map and scope** the chunk and **inventory** its decisions and terminology — not to resolve every hard problem inline (see deferral, below).

## Capture as you go: eager write, marked WIP

The goal is that decisions and terminology **are never lost**, while the grilling **never gets derailed**. So: when material surfaces, write it to disk immediately, but mark it `WIP` so a future reader — even one who's lost all session context — can see it was still being worked and needs resolution. Don't stop to polish; drop a marked breadcrumb and continue.

Four kinds of material get harvested, each routed to the skill that owns it. These are **not mutually exclusive** — one realization (e.g. a boundary decision like "Customer data is owned by the Customer context") commonly produces an ADR, a glossary term, *and* a logical-view fact at once. Capturing all that apply is expected and common.

### ADRs → the `adr` skill

When a design decision surfaces, apply the `adr` skill's **three-test gate** (hard to reverse ∧ surprising without context ∧ result of a real trade-off). At capture time the bar is **low** — if it's unclear whether all three hold, err toward creating a `Status: WIP` ADR. It's cheap and disposable; it gets deleted later if it doesn't pass the gate.

Use the `adr` skill's template, numbering (`docs/adr/ADR-NNNN-<slug>.md`), and status contract — don't reimplement them. For a WIP ADR, fill **Context** (what needs deciding, and why it matters) and **Options Considered** (the candidates in play) richly; mark the **Decision** deferred.

### Glossary terms → the `context-glossary` skill

When domain terminology surfaces, add it to `docs/CONTEXT.md` via the `context-glossary` skill, following its four rules — opinionated canonical term + `_Avoid_`, tight "what it IS" definitions, project-specific terms only, no implementation detail. A term whose definition or canonical choice isn't settled is marked `WIP`.

### Incidental architecture facts → the `architecture-description` skill

When the chunk surfaces a fact about the system's structure or behavior that belongs in the Architectural Description — a logical-view boundary, a runtime interaction pattern, a deployment constraint — capture it in the relevant viewpoint file under `docs/architecture/` via the `architecture-description` skill's conventions. This is *incidental* capture: record the shallow fact and move on.

**Defer anything architecturally deep.** If the architecture point turns out to need real exploration — cross-viewpoint impact, a genuine architectural risk, multiple structural options — do **not** explore it here. That's a rabbit hole, and deliberate architecture work is `explore-architecture`'s job. Flag it as an `[!OPEN]` issue (the `architecture-description` convention) capturing what's unresolved, and note it wants an `explore-architecture` session. Same recognize-but-defer discipline as deep decisions (below).

### Roadmap candidates → `docs/ROADMAP.md`

When the chunk reveals *future* work that isn't part of this chunk, record it as a candidate in `docs/ROADMAP.md` via the `roadmap` skill: a placeholder for a future Work Plan. Mark a candidate `WIP` when there is elevated uncertainty about it specifically (beyond the normal "it's in the future"). The roadmap is a living document of the best current knowledge of the road ahead; candidates that stop making sense are simply removed.

## There is no spec — route "what the system does" facts to their real homes

This toolchain has no `SPECIFICATION.md`. When the grill surfaces a fact that would once have gone into a spec, route it by *kind*:

- **A decision about how the system works** (driven by a real trade-off) → an ADR via the `adr` skill.
- **A fact about the system's structure or behavior** → the Architectural Description via `architecture-description` (incidental capture, deep stuff deferred to `explore-architecture`).
- **The external contract** — CLI flags, API shape, wire format → this is *user-facing documentation*, written when the surface is actually built. Don't write it during the grill (the details aren't settled yet); instead, note that the chunk has externally-visible surface so the resulting Work Package carries "user docs updated" as an acceptance criterion. Recognize the need; don't author the doc.
- **Future work** (revise an existing doc, build something later) → a roadmap candidate.

The principle is the same as everywhere in this skill: recognize and route the material so it isn't lost, without authoring something prematurely or chasing a rabbit hole.

## Recognize deep decisions — but defer, don't chase them

The anti-derailment rule. When a decision turns out to need real research — multiple live options, no clear winner, genuine exploration required — **do not try to resolve it in this session.** That's a rabbit hole, and resolving deep decisions is `grill-me`'s job. Either the skill proposes to defer it, or the user says something like "I don't want to decide that now." Either way:

- Drop a `Status: WIP` ADR capturing **what needs to be decided** and the **options surfaced so far** (enough that a later `grill-me` session can pick it up cold).
- Return to scoping the chunk.

`grill-work` is allowed to *recognize* deep decisions but not to *resolve* them. The WIP ADR is the baton handed to `grill-me`.

## At session end: highlight what's unresolved

Close the session with an explicit summary of everything still open:

- **WIP ADRs created** — open decisions, each a candidate for a focused `grill-me` session.
- **WIP glossary terms** — terminology still to be pinned down.
- **Architecture `[!OPEN]` issues raised** — deep architecture points deferred to an `explore-architecture` session.
- **External-doc needs noted** — chunks with externally-visible surface whose Work Packages should carry "user docs updated" as an acceptance criterion.
- **Roadmap candidates** — future work surfaced.

Make clear that when `to-workplan` runs against this chunk, scope-relevant WIP ADRs will become research Work Packages (whose outcome is an Accepted ADR). A chunk dominated by open decisions is a signal the work isn't ready to build yet — say so plainly if that's the case.
