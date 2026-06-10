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

## Interview Guide

### Opening the Development viewpoint

Anchor to the Logical view if it exists:

> "We've talked about the logical elements of the system. How does that map onto how the code is actually organized? Where would a developer look for {element}?"

If starting without a Logical view:

> "If a new developer joined the team today — how would you describe the structure of the codebase to them? What are the major parts?"

### Identifying modules and components

"Module" and "component" mean different things in different contexts. Establish what unit of organization you're working with:

> "When you think about the major parts of the codebase — are we talking about repositories? Packages? Services? Layers? What's the right granularity?"

For each major organizational unit:

**Contents**: What code lives here? What is it responsible for?

**Dependencies**: What does it depend on? What depends on it?

> "Is the dependency between {A} and {B} intentional? Is it in the right direction?"

**Ownership**: Which team or person owns this? (If applicable)

**Maturity and stability**: Is this a stable foundation that other things build on, or a frequently-changing surface?

### Identifying layers and boundaries

> "Are there layers in the codebase — like a distinction between domain logic and infrastructure concerns? How are those enforced?"

Probe for:
- Layer violations (infrastructure code in domain logic, or vice versa)
- Circular dependencies between modules
- Boundaries that exist conceptually but aren't enforced by the build system

> "Can a developer accidentally introduce a dependency from {stable module} to {unstable module}? What prevents that?"

### Build and dependency structure

> "How is the system built? Is it a monorepo, multiple repositories, or something else?"

> "How long does a full build take? Is that acceptable? Where are the bottlenecks?"

For each significant build dependency:

**Internal dependencies**: Which modules depend on which? Is this structure intentional?

**External dependencies**: What third-party libraries or frameworks are central? Are any of them risks (licensing, maintenance, compatibility)?

### Mapping to Logical elements

> "We said {logical element} is responsible for {X}. Where does that live in the code?"

If a logical element is scattered across many modules, that is architecturally significant — it suggests the boundaries need rethinking.

If a module contains multiple logical elements, that may be appropriate (especially for small systems) or may indicate a need for decomposition.

### Team structure and Conway's Law

If team structure is relevant:

> "How is the development team organized? Do team boundaries align with module boundaries?"

Conway's Law states that a system's design tends to mirror the communication structure of the organization that built it. This is a risk if team boundaries and architectural boundaries are misaligned — probe it.

### Connecting to quality properties

The Development view has strong connections to modifiability and testability:

> "How easy is it to change {module} without affecting other parts of the system? Is there a pattern for how changes are made?"

> "How is the system tested? Are there unit tests? Integration tests? At what level of the build structure?"

---

## Output Structure

```markdown
# Development Viewpoint

> **Living document.** Last updated: {date}
> Terms used in this document are defined in [../CONTEXT.md](../CONTEXT.md).

## Overview

{One paragraph describing the overall development organization — the major structural units, their relationships, and the dominant organizational pattern (e.g. monorepo with packages, microservices in separate repositories, layered monolith). Written for a new developer or coding agent encountering the codebase for the first time.}

## Repository Structure

| Repository / Package | Contents | Primary Owner | Stability |
|---|---|---|---|
| {name} | {what it contains} | {team / person} | {stable / evolving / experimental} |

## Module / Component Structure

### {Module Name}

**Location**: `{path or repository}`
**Responsibility**: {what this module owns in the codebase}
**Realizes**: {Logical element(s) from the Logical view}
**Layer**: {domain / application / infrastructure / presentation / shared / etc.}

**Depends on**:
- `{module}`: {why}

**Depended on by**:
- `{module}`: {why}

**Key constraints**: {dependency rules, what is prohibited, how boundaries are enforced}

---

### {Module Name}
...

## Dependency Map

{Narrative description of the dependency structure — which modules are foundational, which are peripheral, where the dependency flow runs. Diagrams may accompany but the narrative must be self-contained.}

**Dependency direction**: {describe the intended flow, e.g. "domain modules have no dependencies on infrastructure modules"}

**Known violations**: {any current violations of intended dependency rules, with rationale or open issue}

## Build Topology

**Build system**: {Maven / Gradle / npm / cargo / make / etc.}
**Repository model**: {monorepo / polyrepo / hybrid}
**Build time**: {approximate full build time}
**CI/CD**: {brief description of pipeline, if architecturally relevant}

## Logical-to-Development Mapping

| Logical Element | Module(s) | Notes |
|---|---|---|
| {element} | `{module}` | {any mismatch or split worth noting} |

## Testing Structure

| Test Type | Location | Coverage Scope | Notes |
|---|---|---|---|
| Unit | `{path}` | {what it tests} | |
| Integration | `{path}` | {what it tests} | |
| End-to-end | `{path}` | {what it tests} | |

{Open issues as callouts}
```

---

## Common Pitfalls

**Development view as file listing.** The Development view describes the *meaningful* organizational structure — the boundaries, layers, and dependencies that shape how the system is built. It is not a directory listing.

**Ignoring Conway's Law.** If team structure and module structure are misaligned, that is a real risk. Changes that cross team boundaries are expensive and slow. Flag it.

**Missing dependency direction.** Listing that A depends on B is useful. Noting that this dependency is *in the wrong direction* is architecturally significant. Always consider whether dependency direction is intentional.

**Circular dependencies undiscovered.** Circular dependencies between modules are a common and serious structural problem. Ask about them explicitly.

**Testing as an afterthought.** The Development view is the right place to capture how testing is structured. If testing is difficult due to module organization (e.g. hard to test domain logic in isolation because it is coupled to infrastructure), that is an architectural problem, not a testing problem.

**Stability gradients ignored.** Not all modules are equally stable. Stable modules (domain logic, shared utilities) should not depend on unstable ones (UI, third-party integrations). If they do, that is a risk.
