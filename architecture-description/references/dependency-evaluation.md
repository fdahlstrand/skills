# External Dependency Evaluation

Convention for the Development view's **External Dependencies** section: every
external dependency gets a recorded evaluation *before* adoption. The record
has two parts — **liability** (what this dependency costs to carry and to
exit) and **supply chain** (whether its source, maintainers, and license can
be trusted). Adopting a dependency is a *risk-acceptance decision*: the
engineer is responsible for the contract with their users, and a dependency
inside that responsibility is a consciously accepted, mitigated risk.

Load this when writing or reviewing an External Dependencies section, or when
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

Kept in the Development view's External Dependencies section: a summary table
plus one subsection per dependency.

Summary table columns: **Dependency | Scope | Verdict | Date**.

Per-dependency subsection:

- **Scope** — test-only / build-only / shipped. Scope bounds the residual
  exposure: a test-only dependency never reaches users.
- **License** — identified license and compatibility conclusion.
- **Liability** — where it sits on the rubric: inward-pointing behind which
  seam, or contract-grade against which stability evidence.
- **Security posture** — the findings from Part 2, dated.
- **Verdict + date** — Accepted / Rejected, when.
- **Residual risks accepted** — named explicitly; an unnamed risk is not
  accepted, it is unnoticed.
- **Upgrade hygiene** — how upgrades are handled. Baseline: upgrades are
  reviewed like code and never auto-adopted; add project-specific conditions
  (e.g. checksum verification stays on).

An evaluation is a snapshot; re-evaluate when a dependency changes owner,
license, or scope (e.g. test-only becoming shipped).
