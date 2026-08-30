---
name: explore-architecture
description: A human-in-the-loop skill for deliberate, risk-driven architecture exploration using the modernized Kruchten 4+1 viewpoint model (Scenarios, Logical, Runtime, Deployment, Development). Drives structured architectural thinking through interview-style conversation. This is an ELICITATION skill; it interviews and reasons, then writes its findings into the Architectural Description owned by the architecture-description skill, records decisions via the adr skill, and records terminology via the context-glossary skill. Use when the user wants to explore, define, or revisit architecture in a dedicated session — "explore architecture", "architecture session", "let's work on the logical view", "capture some scenarios", or questions about viewpoints and architectural risks. For incidental architecture facts noticed while scoping a chunk of work, grill-work captures those; use explore-architecture for deliberate, cross-viewpoint sessions.
---

# explore-architecture

A skill for **deliberately exploring** software architecture through risk-driven, interview-style conversation. It is one of two elicitation entry points that write into the **Architectural Description (AD)**; the other is `grill-work` (incidental architecture touches while scoping a chunk). This skill is for *dedicated* architecture work — following the risk across viewpoints, checking cross-viewpoint consistency, doing just enough architecture for the risk faced.

Rooted in two frameworks:
- **Kruchten 4+1 (modernized)** — five viewpoints that together describe a complete architecture.
- **Risk-driven architecture** — the amount of architectural work is commensurate with the risk faced; focus on what could make the project fail.

## This skill elicits; it does not own the artifacts

A clean separation of concerns, consistent across the toolchain:

- **The AD content and structure** — viewpoint files, `index.md`, the `docs/architecture/` layout, the `[!OPEN]` open-issue convention, the methodology vocabulary — are owned by the **`architecture-description`** skill. This skill *writes into* that structure. When you need to know what a viewpoint file contains or where something goes, consult `architecture-description` (its `references/viewpoint-templates.md` and `references/index-structure.md`), and read its `references/vocabulary.md` for the methodology terms that govern architecture conversation.
- **Decisions** — when exploration yields a decision meeting the three-test gate, record it via the **`adr`** skill (its template, numbering, `Status: WIP → Accepted` lifecycle, and `docs/adr/README.md` index). This skill does **not** define its own ADR format or numbering.
- **Terminology** — domain terms surfaced during exploration go to the shared **`docs/CONTEXT.md`** via the **`context-glossary`** skill. This skill does **not** maintain its own glossary. Project-facing architecture terms (Runtime Container, External System, Archetype, etc.) are seeded there as the project adopts them.

This skill owns the **conversation** and the **risk-driven judgment** — what to explore, in what order, when a viewpoint is "sufficiently explored for now," and what the open risks are.

## Environment

Requires file system access — it reads the existing AD to resume, and writes confirmed findings into viewpoint files (via `architecture-description` conventions), ADRs (via `adr`), and the glossary (via `context-glossary`). Confirm access at session start.

## Session start protocol

1. **Read the index.** If `docs/architecture/index.md` exists, read it: current state, what viewpoints are explored, open issues, risk register. **Check its `AD revision:` marker against the revision `architecture-description`'s SKILL.md declares** — if the AD is older, or has no marker (absence means revision 1), run the migration in that skill's `references/migration.md` before anything else, and report what moved and what gaps it recorded. Do not load whole viewpoint files to find one record; use the selection tables. If `docs/architecture/` doesn't exist, this is a first session — the AD structure will be created (per `architecture-description`) as content is confirmed, and the shared `docs/CONTEXT.md` seeded (via `context-glossary`) as terms surface.
2. **Orient the user.** Briefly summarize what's known, what's open, which viewpoints are unexplored, and make a concrete **risk-driven recommendation** for where to focus — while making clear the user drives the choice.
3. **Agree on focus.** Confirm which viewpoint or concern to explore. A session may touch several viewpoints — expected and healthy.

## The five viewpoints

| Viewpoint | Interview guide | Primary question |
|---|---|---|
| **Scenarios** (+1) | `references/scenarios.md` | What must the system do, for whom, and why? |
| **Logical** | `references/logical.md` | What are the key concepts, responsibilities, boundaries? |
| **Runtime** | `references/runtime.md` | How do elements interact and behave at runtime? |
| **Deployment** | `references/deployment.md` | Where does it run and how is it deployed? |
| **Development** | `references/development.md` | How is it organized for development? |

Load the relevant interview guide when a viewpoint becomes the focus. These guides hold the *elicitation questions*; the *file structure* they write into is defined by `architecture-description`'s `viewpoint-templates.md`. Start with Scenarios on a first session or when the value proposition is unclear. Sequencing is not a waterfall — follow the risk; document what you don't know as explicitly as what you do.

## Interview principles

- **One question at a time.** Never a list. Each follows from the last answer.
- **Make recommendations.** For every question with a defensible answer, recommend one and explain why, so the user can simply agree and move on.
- **Name uncertainty.** Architecturally significant unknowns become `[!OPEN]` issues (the `architecture-description` convention) — don't paper over gaps.
- **Challenge vague answers.** "Fast", "secure" → probe via the risk lens: what failure occurs if this isn't addressed? How likely? How severe?
- **Follow the risk** across viewpoints, even if it means leaving the current one.
- **"I don't know" is valid** — it becomes an open issue, not a blocker.

## Capturing as you explore — with confirmation

Sessions have no clean boundary; capture in the moment so findings survive. Before writing, show the user exactly what will change — which file(s), a preview of the content, any open issues opened or closed — and wait for confirmation. On confirmation:

- **Viewpoint content** → write into the relevant `docs/architecture/` file per `architecture-description` conventions. **Run that skill's two gates before writing**: does this already have a durable home (if so, don't write it), and is it a record set (if so, it goes in its directory, not the viewpoint file). Update `index.md` (current state, viewpoint map status, risk register, open-issues summary).
- **A decision meeting the three-test gate** → record via the `adr` skill (recognize it explicitly: "this is an ADR — hard to reverse, and the reasoning won't be visible in the code"), then reference it inline from the viewpoint file. The `adr` skill handles numbering, the index, and status.
- **A domain term** → record via the `context-glossary` skill in `docs/CONTEXT.md`.
- **A rejected alternative** → record where the alternative *would have lived*: the viewpoint section, or the ADR that settled the choice. Never only in the session's narrative — the AD keeps no narrative, so an unfiled rejection is lost.

### Open issues

Use the `[!OPEN]` callout owned by `architecture-description`, recorded inline in the file the issue is *about* — a record file such as `archetypes/reviewer.md` or `dependencies/electron.md` as readily as a viewpoint file, but never an evidence file — and indexed in `index.md` by that file. A resolved issue is removed from its file and from the summary; the git history records that it closed. This is also where drift flagged by `grill-work` or implementation lands — an `explore-architecture` session is where such open issues get resolved.

## Session end

Before closing, three checks:

1. **Rewrite `## Current State`** in `index.md` — `As of`, what is and is not settled, and `Next`. It is overwritten, never appended to; the AD keeps no session log, and the git history of `docs/architecture/` is the record of what happened. Write a commit message that carries the session's reasoning, since that is now where it lives.
2. **Record-set hygiene.** Did this session append a record to a section that has become a record set — a further dependency, archetype, or similar accreting subject sitting inline in a viewpoint file? If so, give it its directory now, per the two gates in `architecture-description`.
3. **Evidence hygiene.** Any investigation recorded this session belongs in an evidence file organized by subject, superseding what it replaces — never appended as a new dated block.

## Risk-driven prioritization

When proposing focus: (1) what could cause the project to fail? (engineering risks — scalability, security, integration, data integrity); (2) which viewpoint best addresses it? (3) is it already mitigated, or does it need attention? A viewpoint is "sufficiently explored for now" when the risks that drove exploration there are understood and either mitigated or captured as open issues, the current understanding is recorded, and what remains unknown is documented. There is no "done" — only "current best understanding."

## Relationship to planning

The AD this skill maintains is read by `to-workplan` as optional **constraints** on a chunk (scope-relevant viewpoints → a Work Plan Constraints section). Downstream, when implementation contradicts the AD, an `[!OPEN]` issue is raised and resolved here. This skill is the deliberate-resolution end of that loop; `grill-work` is the incidental-capture-and-defer end.
