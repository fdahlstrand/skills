# AD Structure Migration

The AD's structure is versioned. `docs/architecture/index.md` declares the revision it follows; `SKILL.md` declares the current revision. When they differ, the AD is migrated up.

This file has two audiences. **Migrating an AD** is for an agent working on a project's AD. **Adding a revision** is for an agent changing this skill. **Revisions** is the log both read.

---

## Migrating an AD

### Detecting the revision

The marker lives in `index.md`'s header block:

```markdown
> **AD revision: 2**
```

**An AD with no marker is revision 1.** Every AD written before versioning existed lacks one, and revision 1 is the structure those ADs genuinely have. Absence is a value, not an error.

### The rule: restructure, never invent

**A migration moves, renames, and reshapes what already exists.** Where the new structure needs a value the old AD holds, carry it across. Where the old AD holds nothing, **leave an explicit gap — never a guess.**

The AD is a durable record that later work is planned against. An inferred version number or a reconstructed verdict is a fabricated architecture fact, indistinguishable from a real one once written, and it is the worst failure this artifact has. A migration that reports its gaps honestly is a success; one that fills them plausibly is not.

Record each gap where the missing value belongs:

```markdown
> [!OPEN]
> **Issue**: Migration r1→r2 could not determine {field} for {subject}.
> **Risk**: {what cannot be checked or decided without it}
```

Then index it in `index.md`'s open-issues summary like any other. A clean migration adds no open issues; an incomplete one says exactly where it stopped.

### Procedure

1. Read the marker in `index.md`. Absent means revision 1.
2. Read every revision entry below, from the AD's revision up to the current one, in order. Migrations chain — r1→r3 runs r1→r2 then r2→r3.
3. Apply each entry's changes. Where a step needs a value the AD does not hold, record the gap and continue; do not stop, and do not infer.
4. Update the marker in `index.md` to the new revision.
5. Report to the user: what moved, and every gap recorded.

Do not mix a migration with content edits. Migrate, show the result, then continue the session's actual work.

---

## Revisions

### Revision 2 — record sets, and history moves to git

**Why**: viewpoint files had grown to where an agent had to load 50 KB to find one fact. The cause was not verbose prose but unbounded record sets and a session log kept inside the documents. See the two gates in `SKILL.md`.

**1. Add the revision marker.** In `index.md`'s header blockquote, add `> **AD revision: 2**`.

**2. Delete the Session Log; add Current State.** `## Session Log` and all its entries are removed. In their place, `## Current State` goes at the **top** of `index.md`, above the System summary — see `index-structure.md` for the template.

Before deleting, check each entry for claims that exist nowhere else. Nearly all of it is duplicated in the viewpoint files, the Risk Register (whose closed entries carry their own closure rationale), `CONTEXT.md`, and the git history of `docs/architecture/`. The exception is usually a **rejected alternative** mentioned only in a session narrative — move it to where the alternative would have lived (the relevant viewpoint section, or the ADR that settled the choice) before deleting. Carry the most recent entry's `Next:` into `Current State`; discard the rest, which are stale recommendations.

*Gap to record if it arises*: a claim in a session entry that has no home and no obvious owner. Leave it in an `[!OPEN]` in the viewpoint it concerns rather than deleting it.

**3. Move Archetypes to `archetypes/`.** Each Archetype in `scenarios.md` — its narrative, Drivers, Pains, Gains, **and its Use Cases** — becomes `archetypes/<name>.md`, slug lowercased from the Archetype name. Use Cases move with their Archetype because their `Addresses:` lines reference Pain and Gain names defined only in that Archetype's profile.

`scenarios.md` keeps the System Summary, Non-Goals, an Archetype index table, the Use Case index, the Scenario index, and Cross-Viewpoint Validation.

`scenarios/SC-NNN-*.md` files do **not** move. Use Case → Scenario is a trace relation, not composition, and `SC-NNN` numbering is global by design.

**4. Move external dependency evaluations to `dependencies/`.** Each `###` subsection of `development.md`'s External Dependencies becomes `dependencies/<slug>.md`, following the template in `dependency-evaluation.md`.

Slug from the dependency's name in the summary table, lowercased, with `@`, `/`, and `:` dropped. **Where the name carries a version, slug the role instead** — `node:24-trixie-slim` becomes `node-base-image.md`, so a later version bump does not rename the file and break every inbound link.

`development.md` keeps the convention statement, the summary table (now linking to each file), and any paragraphs naming which ADRs decided the liability half.

Each record gains an overview table. Fill from what the AD holds:

| Row | Source in an r1 AD |
|---|---|
| `Verdict` | the summary table's Verdict column; an adopted dependency is `Accepted` |
| `Decided` | the summary table's Date column |
| `Scope` | the subsection's **Scope** bullet |
| `Version evaluated` | often only in prose (*"none affects 44.0.0"*) — **record a gap if absent** |
| `License` | the **License** bullet |
| `Liability` | the **Liability** bullet's classification |
| `Closure` | the sweep material's package count — **record a gap if absent** |
| `Evidence` | link to `sweep-report.md` and its date, if one exists |

`Rejected` is not a verdict in revision 2. A rejected candidate gets **no file**; its reasoning belongs in the winner's `## Why this dependency` section. Where an r1 AD has a rejected subsection, fold it into the accepted alternative and delete it.

**5. Split evidence into `dependencies/sweep-report.md`.** Sweep or survey material in `development.md` — method, closure results, cross-closure claims — moves to `dependencies/sweep-report.md`.

Redistribute first, then keep only what is genuinely shared. Per-closure package counts belong in each record's `Closure` row; advisory and licence findings belong in each record's `Security posture`; comparisons that settled a choice belong in the winner's `## Why this dependency`. What remains is the method, the standing caveats, and aggregate claims across closures — typically under 1 KB.

**Organize the report by closure, not by run.** If the r1 material is two or more dated sweep blocks, merge them: one section per closure, each carrying its own date. Re-sweeping a closure later replaces its section. A file with one dated block per run is a log, and revision 2 has no logs.

Move no `[!OPEN]` callout into `sweep-report.md`. Evidence is superseded wholesale, which would silently delete the issue. Put it in the viewpoint or the dependency record instead.

**6. Repoint the Open Issues Summary.** The **File** column now names the file that actually holds each callout — `archetypes/reviewer.md`, `dependencies/electron.md` — with the owning viewpoint in the **Viewpoint** column. Any callout that moved in steps 2–5 needs its row updated.

**7. Check for record sets revision 2 did not name.** Run Gate 2 over what remains in each viewpoint file. A section that accretes indefinitely, is picked one record at a time, and has bodied records is a record set even if this entry does not list it; give it a directory. A merely large file with no record set in it stays whole.

---

## Adding a revision

For an agent changing this skill's structural conventions.

### What earns a bump

**Bump only when an AD conforming to revision N would be non-conforming under N+1.** The revision number answers exactly one question — *does this AD need work?* A bump whose migration is empty is a false alarm that costs every AD a no-op check and teaches agents to skip the check.

| Change | Bump? |
|---|---|
| A record set moves to a directory | **yes** |
| A required section is renamed, added, or deleted | **yes** |
| A file layout or naming rule changes | **yes** |
| A new *optional* section is offered | no |
| Template wording, guidance, or examples improve | no |
| A reference file is reorganized | no |

Revisions are **integers**. The only question is whether the AD's structure is older than the skill's, which is a total order; there is no meaningful minor structural change, and semver would invite bumps that migrate nothing.

Migration is about **structure**, not content completeness. A convention that says "also record X" leaves old ADs conforming — they gain X through ordinary sessions, not through a migration.

### Writing the entry

Add a `### Revision N — <short title>` section at the **top** of Revisions, and update the `## AD structure revision:` heading in `SKILL.md` in the same change. Both, or the mechanism is broken.

An entry needs, for each change: **how to detect** the old shape, **what to do**, and **where each new value comes from** in the old AD. Name the fields that have no r(N-1) source, so the migrating agent records a gap instead of inventing one. Write it for an agent that has only the old AD and this file.

### Note on this file

`migration.md` accretes one section per revision, which looks like a record set. It is not one: the record-set rule governs `docs/architecture/`, not this skill's own references. Revisions chain, so a migration usually reads several entries at once and splitting them would cost reads rather than save them. Leave it as one file.
