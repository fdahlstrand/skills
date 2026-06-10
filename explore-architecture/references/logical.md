# Logical Viewpoint Reference

The Logical viewpoint describes the system's conceptual structure — what it is made of, what each part is responsible for, and how the parts relate. It answers the question: *what is this system, from the outside and from within its boundary?*

The Logical viewpoint is structured around two levels of the C4 model:

1. **System Context** — the system as a single box, its Archetypes, and the External Systems it interacts with
2. **Runtime Containers** — the independently runnable units inside the system boundary, their responsibilities, technologies, and relationships

Rendering of diagrams is out of scope for this skill. The output is a markdown description that is self-contained for text-based consumers — developers, coding agents, and architecture tools that generate diagrams from text descriptions.

---

## C4 Concepts Used

| C4 Term | Term Used Here | Definition |
|---|---|---|
| Person | Archetype | A named class of human who interacts with the running system |
| System | System | The software being described |
| External System | External System | A system outside the boundary that the System interacts with |
| Container | Runtime Container | An independently runnable unit inside the System boundary |
| Component | — | Out of scope — belongs to the Development viewpoint |
| Code | — | Out of scope — belongs to the Development viewpoint |

---

## Purpose

The Logical viewpoint serves:
- **Architects and developers** — shared understanding of the system's conceptual structure and boundaries
- **Coding agents** — orientation to what exists and what each part owns before reading code
- **External teams** — understanding of integration points via External Systems

It is grounded in the Scenarios viewpoint: every Runtime Container should participate in at least one architecturally significant scenario. Containers that don't appear in any scenario are candidates for removal or consolidation.

---

## Interview Guide

### Opening the Logical viewpoint

Start with the System Context — establish the boundary before opening the box:

> "Let's start from the outside. If you drew a box labelled '{System Name}' — who interacts with it, and what other systems does it talk to?"

This surfaces Archetypes and External Systems before diving into internal structure. Getting the boundary right first prevents the common mistake of treating internal elements as external ones.

### System Context — Archetypes

Archetypes should already be defined in the Scenarios viewpoint. Confirm they are complete here:

> "Are there any humans who interact with the system that we haven't captured as Archetypes yet?"

If Scenarios haven't been explored yet, identify Archetypes now using the same approach as the Scenarios viewpoint interview guide.

### System Context — External Systems

> "What other systems does {System Name} interact with — either calling them or being called by them?"

For each External System:

- **Name**: what is it called?
- **Owner**: is it owned by this team, another internal team, or a third party?
- **Interaction direction**: does the System call it, does it call the System, or both?
- **Nature of interaction**: what data or capability is exchanged?
- **Criticality**: what happens if this External System is unavailable?

> "Is this interaction something we control, or are we dependent on a third party? What's our exposure if they change their API or go down?"

External System interactions that are hard to reverse, surprising, or involve real trade-offs are ADR candidates — particularly third-party dependencies and integration patterns.

### Runtime Containers — Identifying containers

Once the System Context is clear, open the system boundary:

> "Inside that box — what are the major independently runnable parts? If you were deploying this system, what pieces would you be starting up?"

For each Runtime Container:

**Responsibility**: what does this container own? What is it the authoritative source for?

> "What does {container} do that nothing else does? What data or capability does it own exclusively?"

**Technology**: what is it built with? Keep this to the essential label — enough to understand the container's nature, not a full technology stack.

> "What technology is this container built with? Just the headline — database engine, language, framework — not the full stack."

Challenge technology labels that smuggle in implementation detail:

> "That's the framework — but what *kind* of thing is this container? A web application? A background worker? A data store?"

**Relationships**: how does it interact with other Runtime Containers?

> "How does {container A} communicate with {container B}? Does one call the other, or do they share data?"

Watch for the boundary between Logical and Development: relationships between Runtime Containers are Logical concerns. How those relationships are implemented internally — the classes, modules, and patterns — belongs in the Development viewpoint.

### Probing for architectural significance

> "Which of these Runtime Containers would be hardest to get wrong? Where would a bad design decision have the widest impact on the rest of the system?"

Architecturally significant containers warrant deeper exploration. Straightforward ones can be noted briefly.

### Connecting to quality properties

Quality properties often determine container boundaries:

> "Are there quality properties — security, scalability, availability — that drove how you've divided the system into these containers?"

> "Are there containers that need to scale independently of the others? Are there containers that handle sensitive data and need to be isolated for security reasons?"

### Identifying missing containers

After the initial mapping:

> "Is there anything the system needs to do that doesn't have a home in one of these containers yet?"

> "Where does {scenario} actually execute? Which container handles that?"

Tracing architecturally significant scenarios through the container structure is the best way to surface missing or misplaced responsibilities.

### Technology choices as ADR candidates

When a technology label surfaces that meets the ADR criteria — hard to reverse, surprising without context, result of a real trade-off — propose an ADR immediately:

> "The choice of {technology} for {container} sounds like it was a deliberate decision with real alternatives. Is that worth capturing as an ADR?"

---

## Output Structure

```markdown
# Logical Viewpoint

> **Living document.** Last updated: {date}
> Terms used in this document are defined in [../CONTEXT.md](../CONTEXT.md).

## System Context

{One paragraph: the system's purpose and its place in its environment —
who uses it and what it depends on. Written for a developer or coding agent
encountering this system for the first time.}

### Archetypes

| Name | Description | Interaction |
|---|---|---|
| {Archetype Name} | {brief description} | {what they do with the system} |

### External Systems

| Name | Owner | Interaction Direction | Criticality | Notes |
|---|---|---|---|---|
| {name} | {internal team / third party} | {calls / called by / both} | {High / Medium / Low} | {ADR reference if applicable} |

## Runtime Containers

### {Container Name}

**Responsibility**: {What this container owns — one focused sentence or short paragraph.
What it is the authoritative source for. What would break if it didn't exist.}

**Technology**: {Headline technology label — e.g. "PostgreSQL database", "React single-page application", "Node.js REST API". No implementation detail.}

**Archetypes served**: {which Archetypes interact directly with this container, if any}

**Relationships**:
- Calls **{Container}**: {what it requests and why}
- Called by **{Container}**: {what is requested}
- Reads from / writes to **{Container}**: {what data}

**Architecturally significant**: Yes / No — {brief rationale if yes}

{ADR reference if a technology choice or boundary decision was recorded}

---

### {Container Name}
...

## Scenarios Traceability

Which architecturally significant scenarios involve which Runtime Containers:

| Scenario | Runtime Containers Involved |
|---|---|
| {SC-NNN: name} | {Container A}, {Container B} |

Runtime Containers not appearing in any scenario:
- {Container} — {rationale for inclusion, or flag for review}

{Open issues as callouts}
```

---

## Common Pitfalls

**Technology detail leaking in.** A Runtime Container's technology label should identify its nature — "PostgreSQL database", "React SPA" — not describe its internal structure. The moment the description mentions classes, modules, design patterns, or configuration, it has crossed into the Development viewpoint.

**External Systems treated as Runtime Containers.** An External System sits outside the system boundary. If it's owned and deployed by another team or third party, it's External — even if it runs in the same infrastructure. Probe ownership and deployment authority, not just location.

**Missing External Systems.** It is common to name Runtime Containers but omit External Systems — especially internal shared services, auth providers, and third-party APIs. Always ask: "Does this container call anything outside the system boundary?"

**Runtime Containers that are really deployment units.** "The production server" is a Deployment viewpoint concern. A Runtime Container is defined by its responsibility and runtime identity, not where it is hosted.

**Archetypes interacting with multiple containers.** If an Archetype needs to know about multiple Runtime Containers, that is a design smell — Archetypes should interact with the system through a coherent surface, not navigate internal structure. Flag it.

**Component-level detail in the Logical view.** How a Runtime Container is structured internally — its layers, modules, classes — belongs in the Development viewpoint. The Logical view describes what the container is and what it owns, not how it achieves that.
