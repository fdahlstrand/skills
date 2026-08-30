# External Dependency Evaluation

Convention for external dependency records: every external dependency gets a
recorded evaluation *before* adoption, kept as one file per dependency in
`docs/architecture/dependencies/` and indexed from the Development view's
**External Dependencies** section. The record
has two parts — **liability** (what this dependency costs to carry and to
exit) and **supply chain** (whether its source, maintainers, and license can
be trusted). Adopting a dependency is a *risk-acceptance decision*: the
engineer is responsible for the contract with their users, and a dependency
inside that responsibility is a consciously accepted, mitigated risk.

Load this when writing or reviewing a dependency evaluation, or when
a session is deciding whether to adopt a dependency.

---

## Part 1 — Liability

### The rubric prices the risk

Every external dependency is a liability, priced on five points:

1. **Culture import.** A dependency incorporates a foreign development
   culture — its conventions, release habits, and quality bar — into your
   project.
2. **Supply-chain uncertainty.** When are security issues patched? How
   fragile is the maintainer structure? Corporate stewardship is no
   guarantee.
3. **Excess surface.** Dependencies solve *general* problems, so they contain
   far more functionality than you need — all of it shipped, all of it attack
   surface.
4. **Dependencies must point inward.** An external dependency should be an
   implementation detail — wrapped, swappable, easy to change. The seam is
   the mitigation (see the Seams & Extension Points section of the
   Development view).
5. **Contract dependencies are the strongest commitment.** A dependency built
   *upon* — one that cannot be swapped behind a seam — must be extremely
   slow-moving and stable, because there is no exit.

The rubric tells you the **price** (carry cost and exit cost), not the
verdict.

### The decision is risk acceptance

The priced liability is weighed against what buying saves: **development
cost** and **time-to-market**. How much weight the savings get scales with
the exit:

- **Inward-pointing dependencies** (point 4): savings get *full* weight. The
  seam bounds the risk — if the dependency sours, it is swapped. Most
  dependencies should live here.
- **Contract-grade dependencies** (point 5): savings get *near-zero* weight.
  The time-to-market gain is one-time; the exit cost is unbounded and
  compounding.

### The extreme-stability bar

Anything that becomes part of a **persistent, user-facing format or API** is
contract-grade by construction — the *users'* artifacts are hostage to it,
and the responsibility for that contract cannot be delegated. Such
dependencies are not banned; they face an extreme-stability bar (think Go,
POSIX, JSON — decade-scale compatibility promises). **Owning is the fallback
when no candidate passes the bar.**

---

## Part 2 — Supply chain

Two concerns, both recorded explicitly:

### Cybersecurity posture

- **Advisory history** — known vulnerabilities (OSV/CVE), and how quickly
  past issues were patched.
- **Maintainer structure** — bus factor, personal account vs organization,
  track record of the maintainers.
- **Transitive closure** — what the dependency itself pulls in; every
  transitive dependency inherits this whole evaluation.
- **Install-time and build-time behavior** — install hooks, code generation,
  network access during build. This record *prices* that exposure; what
  *bounds* it is the toolchain container, owned by the `configure-toolchain`
  skill. A residual risk accepted here should say what contains it.
- **Pinning and verification** — lockfiles, checksums, signature or
  transparency-log verification; whether upgrades are explicit or automatic.
- **Ecosystem scrutiny** — how many serious projects depend on it (more eyes
  on the code and on releases).

### Licensing

- **License identification** — the actual license(s) of the dependency *and
  its transitive closure*.
- **Compatibility** — with the project's own license and its distribution
  model (static linking, copyleft obligations, notice requirements).
- **Relicensing risk** — CLA-enabled license changes, source-available
  conversions, dual-licensing pressure.

---

## The record

Each dependency gets **its own file**, `docs/architecture/dependencies/<slug>.md`.
The Development view keeps only the convention statement and a selection
table linking to them. (The `dependencies/` directory is a *record set* — see
the two gates in `SKILL.md`; the evaluations accrete without bound and are
read one at a time, so they never live inside the viewpoint file.)

### Slugs

Slug from the dependency's name in the summary table, lowercased, with `@`,
`/`, and `:` dropped: `@types/node` becomes `types-node.md`. **Where the name
carries a version, slug the role instead** — `node:24-trixie-slim` becomes
`node-base-image.md`, so a version bump does not rename the file and break
every inbound link. Dependencies have natural unique keys; do not number them.

### The selection table in `development.md`

Columns: **Dependency | Scope | Verdict | Date**, each row linking to its file.
Enough to answer "what do we depend on, and is it settled" without opening
anything, and to choose which record to open.

### The per-dependency file

```markdown
# {Dependency}

|  |  |
|---|---|
| **Verdict** | Accepted |
| **Decided** | 2026-08-29 |
| **Scope** | shipped |
| **Version evaluated** | 44.0.0 |
| **License** | MIT (closure: MIT, BSD-2-Clause, ISC, Apache-2.0) |
| **Liability** | contract-grade for the shell |
| **Closure** | 11 packages |
| **Evidence** | [sweep-report.md](sweep-report.md), 2026-08-29 |

## Why this dependency
<Alternatives considered and rejected, and why this one won. Omit when there
was no real choice — an obvious decision documented is noise.>

## Liability
<Where it sits on the rubric: inward-pointing behind which seam, or
contract-grade against which stability evidence.>

## License
<Identified license for the package and its closure; compatibility
conclusion; any distribution obligation.>

## Security posture
<Free subheadings as the evidence requires — advisory history, maintainer
structure, install-time behaviour, pinning and verification, provenance,
ecosystem scrutiny, the vendor's own caveats, patch speed. Dated.>

## Residual risks accepted
<Named explicitly; an unnamed risk is not accepted, it is unnoticed. Where
something else contains a risk (the toolchain container, a seam, a lockfile),
say so.>

## Upgrade hygiene
<Baseline: upgrades are reviewed like code and never auto-adopted. Add
project-specific conditions.>
```

The **fixed sections are the checklist**; `Security posture` takes free
subheadings because the evidence is not predictable and forcing it into fixed
slots loses the findings that matter most.

### Verdict: `WIP` → `Accepted`

The same lifecycle the `adr` skill uses, and for the same reason.

- **`WIP`** — evaluation started, dependency **not yet adopted**. This is the
  state that gives the convention teeth: *"every dependency gets a recorded
  evaluation before adoption"* is unobservable without it. To an agent, `WIP`
  reads as **do not add this to the lockfile yet**.
- **`Accepted`** — evaluated and adopted.

There is no `Rejected`. **The AD records the dependencies the project has, not
the ones it considered.** A rejected candidate gets no file; the comparison
that settled the choice lives in the winner's `## Why this dependency`, or in
an ADR when the choice is architecturally significant. Where a rejection is
obvious, record nothing.

`Version evaluated`, not `pinned version`: the pin's truth is the lockfile,
and a doc restating it drifts within a week. `Version evaluated` is a
permanent claim about the evaluation, it is what makes "no advisory affects
44.0.0" checkable later, and seeing it fall behind the lockfile is the signal
to re-evaluate.

### Evidence: `dependencies/sweep-report.md`

Where a project sweeps closures mechanically, the method and its cross-cutting
findings go in `dependencies/sweep-report.md` — not in the viewpoint, and not
duplicated into each record. It holds the **method**, the **standing caveats**
(e.g. "no lockfile exists yet; re-run against the real one when it does"), and
**aggregate claims** across closures. Typically under 1 KB.

Everything else redistributes: per-closure package counts to each record's
`Closure` row, advisory and licence findings to each record's `Security
posture`, and comparisons that settled a choice to the winner's `## Why this
dependency`.

**Organize by closure, never by run.** One dated section per closure;
re-sweeping replaces that section. A file that gains a dated block per sweep
is a log — bounded by how often you look rather than by what you depend on.

**No `[!OPEN]` callout goes in an evidence file.** It would vanish silently at
the next supersession. Put it in the viewpoint or the dependency record.

---

An evaluation is a snapshot; re-evaluate when a dependency changes owner,
license, or scope (e.g. test-only becoming shipped), or when `Version
evaluated` falls meaningfully behind what is installed.
