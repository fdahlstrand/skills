# Development Viewpoint Reference

The Development viewpoint describes how the system is organized for development — its modules, components, layers, packages, repositories, and their build and dependency structure. It answers the question: *how is this system organized so that developers can work on it effectively?*

This is Kruchten's "Development View," sometimes called the "Implementation View." It is the view closest to the code and most relevant to the development team's day-to-day work.

---

## Purpose

The Development view serves:
- **Developers** — understanding how to navigate, extend, and modify the codebase
- **Team leads and architects** — managing complexity, enforcing boundaries, enabling parallel work
- **Coding agents** — understanding where things live and how the codebase is organized before reading individual files
- **Build and CI/CD systems** — understanding dependency structure and build topology

The Development view maps the Logical view's elements onto code — but the mapping is rarely 1:1. One logical element may span multiple modules; one module may realize parts of multiple logical elements. The Development view makes these mappings explicit.

---

## Two modes: descriptive and prescriptive

Establish the mode before asking anything else:

- **Descriptive (brownfield)** — code exists; the session recovers and records how it is actually organized.
- **Prescriptive (greenfield)** — little or no code exists; the session *designs* the structure that the first roadmap increment will instantiate. The view is a prescription until the code catches up, and says so in its Overview.

**The inversion rule.** The interview sections below are written descriptively. In prescriptive mode, flip every question from *what is* to *what should be, and what makes it real*: "where does X live?" becomes "where should X live, and what will enforce that?" Most of the inversion is mechanical — apply it as you go. Where it is *not* mechanical, the sections below carry an explicit **Prescriptive:** variant.

In prescriptive mode the Logical view is not an optional anchor but the input: the session's job is to design the code structure that realizes the Runtime Containers and respects the risk register.

---

## Interview Guide

### Opening the Development viewpoint

Anchor to the Logical view if it exists:

> "We've talked about the logical elements of the system. How does that map onto how the code is actually organized? Where would a developer look for {element}?"

If starting without a Logical view:

> "If a new developer joined the team today — how would you describe the structure of the codebase to them? What are the major parts?"

**Prescriptive:** the Logical anchor is effectively mandatory — do the Logical view first if it doesn't exist. Open with:

> "What code structure realizes these Runtime Containers? What is the spine of the codebase — the small set of representations or units everything else hangs off?"

### Identifying modules and components

"Module" and "component" mean different things in different contexts. Establish what unit of organization you're working with:

> "When you think about the major parts of the codebase — are we talking about repositories? Packages? Services? Layers? What's the right granularity?"

For each major organizational unit:

**Contents**: What code lives here? What is it responsible for?

**Dependencies**: What does it depend on? What depends on it?

> "Is the dependency between {A} and {B} intentional? Is it in the right direction?"

**Ownership**: Which team or person owns this? (If applicable)

**Maturity and stability**: Is this a stable foundation that other things build on, or a frequently-changing surface?

**Prescriptive:** stability becomes the *intended stability gradient* — design it, don't discover it:

> "Which parts of this system are frozen external contracts — formats, grammars, APIs your users' artifacts depend on? Which internals are free to churn? Is any internal representation under pressure to become semi-stable because many things couple to it?"

A common healthy shape is contract–churn–contract: frozen contracts at the system's edges, churning internals between them, and any many-dependent internal representation given a deliberate, owned surface.

### Identifying layers and boundaries

> "Are there layers in the codebase — like a distinction between domain logic and infrastructure concerns? How are those enforced?"

Probe for:
- Layer violations (infrastructure code in domain logic, or vice versa)
- Circular dependencies between modules
- Boundaries that exist conceptually but aren't enforced by the build system

> "Can a developer accidentally introduce a dependency from {stable module} to {unstable module}? What prevents that?"

**Prescriptive:** enforcement is a design choice, not an observation:

> "What *mechanism will* enforce this boundary — compiler visibility, package layout, a build rule? If the answer is 'discipline', is that acceptable, or is there a cheap mechanical enforcement?"

Prefer boundaries the toolchain checks (e.g. language-level visibility such as Go's `internal/`, module systems, build-graph rules) over boundaries that live in a document.

### Seams and extension points

Seams exist to serve identified risks. Source the list from the risk register and the Logical view, not from the code:

> "Which elements does the risk register (or Logical view) say must stay swappable or independently evolvable?"

A seam nobody can name a risk for is suspect. For each seam:

**What crosses it**:

> "What representation crosses this seam — what data shape, precisely? Which side defines and owns it?"

**What is kept out** — the negative space is often the highest-value content:

> "What must *never* cross this seam?"

**Enforcement**:

> "Is the seam compiler-enforced, build-enforced, or discipline-enforced? If discipline — is that acceptable, or is there a cheap mechanical enforcement?"

**Stability pressure**:

> "Is the crossing representation under pressure to become a contract — how many things depend on it? Is that acknowledged and owned?"

### Build and dependency structure

> "How is the system built? Is it a monorepo, multiple repositories, or something else?"

> "How long does a full build take? Is that acceptable? Where are the bottlenecks?"

> "Does the toolchain run on the developer's host, or in a project-owned container? What bounds what dependency-supplied code can reach during a build?"

Repository boundaries and where tool configuration is allowed to live are owned by the `structure-source-workspace` skill; containment of build-time execution is owned by `configure-toolchain`. Elicit the fact here and record it; route the design work to those skills.

**Internal dependencies**: Which modules depend on which? Is this structure intentional?

### External dependencies

Every external dependency gets a recorded evaluation *before* adoption — liability plus supply chain — recorded one file per dependency in `docs/architecture/dependencies/`, indexed by a selection table in the Development view's External Dependencies section. The convention and record format are defined in the `architecture-description` skill's `references/dependency-evaluation.md`; load it when a dependency is on the table. Elicit per dependency:

> "What is its scope — test-only, build-only, or shipped? What reaches your users?"

> "Is it inward-pointing — wrapped behind a seam, swappable — or are you building *on* it with no exit? If contract-grade, what evidence says it meets an extreme-stability bar?"

> "What is its license, and the licenses of everything it pulls in? Compatible with how you ship?"

> "What does its security posture look like — advisory history, maintainer structure, transitive closure, install-time behavior, pinning and verification?"

> "What residual risks are you accepting, by name? How are upgrades handled — reviewed like code, never auto-adopted?"

Write the record **before** the dependency is adopted, with `Verdict: WIP` — that is what the "before adoption" convention means in practice, and it reads to a later agent as *do not add this to the lockfile yet*. Flip it to `Accepted` when the decision is made.

Where candidates were compared, the losers get **no file of their own**. Their reasoning goes in the winner's `## Why this dependency` section, or in an ADR when the choice is architecturally significant. Where the choice was obvious, record nothing — the AD holds the dependencies the project has, not the ones it considered.

In descriptive mode, sweep the existing dependency manifest and flag any dependency with no recorded evaluation.

### Mapping to Logical elements

> "We said {logical element} is responsible for {X}. Where does that live in the code?"

If a logical element is scattered across many modules, that is architecturally significant — it suggests the boundaries need rethinking.

If a module contains multiple logical elements, that may be appropriate (especially for small systems) or may indicate a need for decomposition.

### Team structure and Conway's Law

*Solo project or single team: skip this section.*

> "How is the development team organized? Do team boundaries align with module boundaries?"

Conway's Law states that a system's design tends to mirror the communication structure of the organization that built it. This is a risk if team boundaries and architectural boundaries are misaligned — probe it.

### Connecting to quality properties

The Development view has strong connections to modifiability and testability:

> "How easy is it to change {module} without affecting other parts of the system? Is there a pattern for how changes are made?"

> "How is the system tested? Are there unit tests? Integration tests? At what level of the build structure?"

A named principle to probe for: **test weight follows the stability gradient** — spec-grade suites on frozen contracts (they *are* the contract's executable definition), ordinary coverage on churny internals.

> "Does your test investment match your stability gradient? Are the frozen contracts the most heavily specified parts of the suite — and are you over-testing internals that are free to churn?"

### First-increment instantiation

**Prescriptive mode only.** A prescribed structure that never says what makes it real is a wish:

> "What does the first roadmap increment instantiate of this structure? Which packages exist on day one, even in trivial form?"

> "Does the walking skeleton pass through every seam — so each boundary is exercised before the parts behind it get big?"

Record the answer in the view's First-Increment Instantiation section.

---

## Output Structure

The structure of `development.md` is owned by the `architecture-description` skill — see its `references/viewpoint-templates.md` (required core, a conditionally required External Dependencies section that indexes the `dependencies/` record set, and an optional menu). This guide elicits the content; it does not define the file.

---

## Common Pitfalls

**Development view as file listing.** The Development view describes the *meaningful* organizational structure — the boundaries, layers, and dependencies that shape how the system is built. It is not a directory listing.

**Ignoring Conway's Law.** If team structure and module structure are misaligned, that is a real risk. Changes that cross team boundaries are expensive and slow. Flag it.

**Missing dependency direction.** Listing that A depends on B is useful. Noting that this dependency is *in the wrong direction* is architecturally significant. Always consider whether dependency direction is intentional.

**Circular dependencies undiscovered.** Circular dependencies between modules are a common and serious structural problem. Ask about them explicitly.

**Testing as an afterthought.** The Development view is the right place to capture how testing is structured. If testing is difficult due to module organization (e.g. hard to test domain logic in isolation because it is coupled to infrastructure), that is an architectural problem, not a testing problem.

**Stability gradients ignored.** Not all modules are equally stable. Stable modules (domain logic, shared utilities) should not depend on unstable ones (UI, third-party integrations). If they do, that is a risk.

**Unrecorded dependency adoption.** A dependency that arrived without an evaluation is an unpriced liability. In descriptive sessions, sweep the manifest; in prescriptive sessions, evaluate before adopting — never after.

**Prescription mistaken for description.** A greenfield view must say it is prescriptive. Otherwise a later reader (or agent) treats the intended structure as fact and never notices the code has drifted from it.
