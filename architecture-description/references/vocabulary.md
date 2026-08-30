# Architecture Methodology Vocabulary

Skill-internal reference. These terms define **how architecture is described** in this toolchain — the methodology language used by `architecture-description` and `explore-architecture`. This file is **not** a project artifact: it is never copied into `docs/`, and it is distinct from the project's domain glossary (`docs/CONTEXT.md`, owned by `context-glossary`).

A subset of these terms is **project-facing** — terms a given project will actually adopt in its code and conversations (e.g. Runtime Container, External System, Archetype). Those are seeded into the project's shared domain glossary via `context-glossary` **as the project uses them**, following the glossary's four rules — not dumped wholesale. They are marked `[glossary-candidate]` below. Everything unmarked is pure methodology and stays here.

---

## Core architectural concepts

**System**: The software elements needed to meet a set of requirements, plus the hardware to run them.

**Architecture**: The fundamental concepts or properties of a system in its environment, embodied in its elements, their relationships, and the principles of its design and evolution.

**Architectural Element**: A fundamental piece from which a system can be considered to be constructed. `[glossary-candidate]` — prefer over "component" (overloaded), "module" (view-specific), "service" (technology-specific).

**Interelement Relationship**: How two or more architectural elements relate. Broader than "dependency."

**Static Structure**: The design-time elements and their arrangement — what exists in the codebase.

**Dynamic Structure**: The runtime elements and their interactions — what exists while running.

**Externally Visible Behavior**: The functional interactions between the system and its environment — what the system does as observed from outside.

**Quality Property**: An externally visible, nonfunctional property such as performance, security, scalability, availability. `[glossary-candidate]` — prefer over "non-functional requirement" / "NFR" (imprecise).

**Architecturally Significant**: A concern, problem, or element is architecturally significant if it has wide impact on the system's structure or its important quality properties. `[glossary-candidate]`

---

## Stakeholders and concerns

**Stakeholder**: An individual, team, organization, or class with an interest in the realization of the system — not only users, but developers, operators, acquirers, regulators, etc. `[glossary-candidate]`

**Archetype**: A named class of human who interacts directly with a running instance of the system; a subtype of Stakeholder, and the actor in the Scenarios viewpoint. File-backed: each lives in `archetypes/<name>.md` and carries its own Use Cases. Identified by name; the slug is that name lowercased, never a number. `[glossary-candidate]` — prefer over "user", "actor" (UML), "persona".

**Concern**: A requirement, objective, constraint, intention, or aspiration a stakeholder has for the architecture. Broader than "requirement."

**Constraint**: A concern imposed on the architecture with no flexibility — a decision already made.

---

## Views and viewpoints

**Architectural Description (AD)**: The set of documents recording the architecture so stakeholders can understand it and see their concerns met. A living document. Avoid "spec", "blueprint" (imply finality).

**View**: A representation of one or more structural aspects of an architecture, addressing specific stakeholder concerns. More than a diagram.

**Viewpoint**: The patterns, templates, and conventions for constructing one type of view; defines whose concerns it reflects and how its views are built.

**Open Issue**: A concern, question, or risk identified but not yet resolved, recorded as an `[!OPEN]` callout in the file it is about — viewpoint file or record file — and indexed in `index.md`. (Convention defined in the main SKILL.) Avoid "TODO", "gap".

**Record Set**: A section of the AD whose contents accrete indefinitely as one independently-readable record per subject. Qualifies only when all three hold: unbounded accretion, keyed access (the filename discriminates), and bodied records. A record set lives in its own directory named for the record type, one level under `docs/architecture/`, from the first record — never inside a viewpoint file. Archetypes, Scenarios, and external dependencies are record sets; the Risk Register is not. (The test is defined in the main SKILL.)

**Evidence**: A dated record of what an investigation found — a dependency sweep, a parser survey, a benchmark. Distinct from a durable claim: evidence describes *now*, is organized by subject rather than by run, and is **superseded in place** when re-run. Evidence files never carry `[!OPEN]` callouts, which supersession would silently delete.

**AD Structure Revision**: The integer version of the AD's file/section structure, declared in `index.md` and matched against the `architecture-description` skill. Absence of a marker means revision 1. See `references/migration.md`.

---

## The five viewpoints (modernized Kruchten 4+1)

**Scenarios Viewpoint** (+1): Captures selected use cases/scenarios that drive and validate the four structural viewpoints — the thread stitching the others together.

**Logical Viewpoint**: The functional building blocks — key domain concepts, responsibilities, relationships.

**Runtime Viewpoint**: How logical blocks combine into runtime processes/execution units — concurrency, inter-process communication, dynamic behavior. (Kruchten's "Process view.")

**Deployment Viewpoint**: Infrastructure and deployment topology — where elements run and how they're distributed. (Kruchten's "Physical view.")

**Development Viewpoint**: How the building blocks are organized for development — modules, components, layers, packages, build and dependency structure.

---

## Risk-driven architecture

**Risk**: A concern that, if unaddressed, could cause the project or system to fail in an architecturally significant way. Engineering risks (scalability, security, integration, data integrity), not management risks (schedule, budget).

**Risk Mitigation**: A decision, tactic, or investigation that reduces a risk's likelihood or impact to an acceptable level.

**Architectural Tactic**: An established, proven approach to achieve a particular quality property. Narrower than "pattern."

**Architectural Perspective**: A collection of activities, tactics, and guidelines ensuring a system exhibits a set of related quality properties across multiple views.

---

## Session and process

**Session**: A single working conversation in which one or more viewpoints are explored and the AD is updated. (Relevant to elicitation skills.)

**Current State**: A single block at the top of `index.md`, carrying an `As of` date, what the architecture has and has not settled, and what is recommended next. **Overwritten every session, never appended to.** Avoid "session log", "changelog" — the AD keeps no history of its own; what happened is the git history of `docs/architecture/`.

**Problem Space**: The domain of stakeholder needs, concerns, constraints, and risks the system must address — explored primarily through Scenarios.

**Solution Space**: The decisions, structures, and designs that address the problem space — documented across all five viewpoints.

---

## Scenarios viewpoint concepts

**Driver**: Why an Archetype engages with the system at all — the higher purpose or external pressure the system is merely a tool for. Never a system capability. `[glossary-candidate]` — prefer over "business goal", "objective".

**Pain**: An obstacle, cost, or risk an Archetype experiences in pursuing their Drivers — functional, social, or emotional. Identified by a bold short name in the Archetype profile; Use Cases trace to it via their `Addresses` line. `[glossary-candidate]` — prefer over "pain point", "problem".

**Gain**: An outcome an Archetype wants beyond pain removal — functional, social, or emotional. Identified by a bold short name; traced from Use Cases like a Pain. `[glossary-candidate]` — prefer over "benefit", "value-add".

**Use Case**: The explicit goal of an Archetype when interacting with the system. Belongs to exactly one Archetype; names intent, not mechanism. Carries an `Addresses` trace to the Drivers, Pains, and Gains it serves. `[glossary-candidate]` — prefer over "feature", "user story", "function".

**Scenario**: A specific path through a Use Case, as a sequence of Interactions at the system boundary — happy path, error paths, edge cases. Contains only externally visible interactions. `[glossary-candidate]`

**Interaction**: A single directional exchange at the boundary between an Archetype and the System (Archetype→System or System→Archetype). Never internal mechanics.

**Happy Path**: The Scenario where everything proceeds as intended and the Archetype achieves their goal without error.

**Slug**: A short, human-readable filename identifier. Only file-backed elements have slugs. Scenarios use `SC-NNN` (globally unique, permanent, never reused). Archetypes are file-backed and slugged from their name, with no number. Use Cases live inside their Archetype's file and have no slug.

**Trace**: A named reference from one element to another that it serves or exercises, without ownership. A Use Case *traces* to Drivers, Pains, and Gains via its `Addresses` line; a Scenario *traces* to its Use Case. A trace is not composition: traced elements may live in separate files, and `scenarios/` is a sibling of `archetypes/` for exactly this reason.

---

## Logical viewpoint concepts

**Runtime Container**: An independently runnable unit inside the System boundary — web app, API, database, worker, message queue. Defined by responsibility and runtime identity, not deployment location. (C4's "Container", predating Docker.) `[glossary-candidate]` — prefer over "container" (Docker-ambiguous), "service", "component" (Development viewpoint), "module".

**External System**: A system outside the System boundary that the System interacts with. Named/described in Logical; its interactions described in Runtime. `[glossary-candidate]` — prefer over "dependency", "integration", "third-party" (not all are).

---

## Flagged ambiguities (methodology)

- **"User"** — too imprecise. Use **Archetype** (runtime actor class) or **Stakeholder** (anyone with a concern). Every Archetype is a Stakeholder; not vice-versa.
- **"Component"** — varies across literature. Prefer **Architectural Element** generally; use "component" only for the Development viewpoint's code organization. (C4 level-3 Component belongs to Development, not Logical.)
- **"Process view"** → the **Runtime** viewpoint. **"Physical view"** → the **Deployment** viewpoint.
- **"Non-functional requirement"** → **Quality Property** (the property) or **Concern** (the stakeholder's interest in it).
- **"Done"** does not apply to the AD — the framing is "current best understanding."
- **"Container" (C4)** → **Runtime Container** (not a Docker container). **"Person" (C4)** / **"Actor" (UML)** → **Archetype**.

---

## Note on ADR vocabulary

Earlier versions of this methodology defined ADR and ADR-status terms here. Those are now owned by the `adr` skill, which is the single source of truth for the ADR template, numbering, and the `Status: WIP → Accepted → Superseded` lifecycle. Do not redefine ADR conventions in architecture work — reference the `adr` skill.
