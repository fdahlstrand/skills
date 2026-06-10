# Scenarios Viewpoint Reference

The Scenarios viewpoint is the +1 in Kruchten's 4+1 model. It captures the externally visible behavior of the system — what it does for its Archetypes — organized as a three-level hierarchy: **Archetype → Use Case → Scenario**. It is the thread that drives and validates all four structural viewpoints.

Start here if this is a first session or if the system's value proposition is unclear. Return here whenever cross-viewpoint consistency needs to be validated, or whenever a structural decision raises questions about externally visible behavior.

---

## Concept Hierarchy

```
Archetype          A named class of entity that interacts with the running system
└── Use Case       The explicit goal an Archetype pursues — one job to be done
    └── Scenario   One path through that goal — happy path, error path, edge case
        └── Interaction   One directional exchange at the system boundary
```

**Interaction directions**:
- Archetype-initiated: written with the Archetype as subject ("Archetype submits the form")
- System-initiated: written with the System as subject ("System sends a confirmation email")

Direction is encoded in the subject of the sentence — no separate notation needed.

Internal system behavior is never part of a Scenario. Only what crosses the boundary between Archetype and System is captured here.

---

## File Structure

The Scenarios viewpoint uses a flat file structure. Archetypes and Use Cases are described in `scenarios.md`. Each Scenario is a single file in `scenarios/` with a globally unique sequential number.

```
docs/architecture/
├── scenarios.md                          # Archetypes, Use Cases, and navigation index
└── scenarios/
    ├── SC-001-successful-login.md
    ├── SC-002-incorrect-password.md
    ├── SC-003-account-locked.md
    └── SC-004-reset-via-email.md
```

Scenario files are identified by a globally unique `SC-NNN` number. The number is identity; the descriptive name is readability. Two scenarios may share a descriptive name — their numbers make them distinct. Scenario files are self-contained: each one states its Use Case goal and Archetype so it can be understood without opening `scenarios.md`.

---

## Interview Guide

### Opening the Scenarios viewpoint

Begin by establishing the system's reason for existence:

> "Before we get into specific scenarios — in one or two sentences, what is this system for? Who would notice if it didn't exist?"

If the user struggles to answer, that is an important signal — the problem space is not yet clear enough to drive architectural decisions. Capture it as an open issue and explore what is known before proceeding.

### Identifying Archetypes

Draw out Archetypes conversationally — don't present a checklist:

> "Who are the different kinds of people that interact with this system while it's running? Let's start with whoever you picture when you imagine it working well."

For each Archetype named:

- What distinguishes this Archetype from others? (role, permissions, context, volume of use)
- What would make the system fail them?
- How technically sophisticated are they?

Archetype names must be unique within the system. If two candidate Archetypes have the same name, probe whether they are truly distinct — different goals, permissions, or interaction patterns — or whether they are the same Archetype described twice.

Watch for **Archetype conflation** — "users" that are actually two distinct Archetypes with different goals, permissions, or interaction patterns. Probe for variation: new vs. experienced, high-volume vs. occasional, technical vs. non-technical.

Note: external systems that interact with the running system are not currently modelled as Archetypes. If the user raises an external system as an actor, acknowledge it as architecturally relevant, capture it as a named concern, and note that it is covered by the open issue for external system modelling in `scenarios.md`.

### Identifying Use Cases

For each Archetype, surface their Use Cases one at a time:

> "What are the goals {Archetype} comes to the system to achieve? Let's start with the most important one — the reason they'd be lost without this system."

For each Use Case:

- **Goal statement**: what job is the Archetype trying to get done? One sentence, no implementation detail.
- **Preconditions**: what must be true before the Archetype can pursue this goal?
- **Architectural significance**: is this Use Case hard to get right? Would failure here be damaging?

> "Is there anything about this Use Case that you'd expect to be technically difficult, or where failure would be particularly costly?"

Don't try to enumerate all Use Cases before exploring any deeply. Follow the risk — explore architecturally significant Use Cases first.

### Identifying Scenarios

For each Use Case, explore the paths through it:

> "Walk me through what happens when {Archetype} successfully achieves {Use Case goal}. Start from the moment they engage with the system."

This produces the happy path Scenario. Then probe for alternatives:

> "What could prevent {Archetype} from achieving their goal? What does the system do in that case?"

> "Are there edge cases — unusual inputs, timing issues, boundary conditions — that would cause this to play out differently?"

Each distinct path is a Scenario. Name them descriptively: `SC-001-successful-login.md`, `SC-002-incorrect-password.md`, `SC-003-account-locked.md`. Sequential numbers are assigned globally across all Scenarios in the viewpoint — never reuse a number.

### Eliciting Interactions

For each Scenario, build the Interaction sequence step by step:

> "What is the very first thing {Archetype} does? What does the system show or do in response?"

For each Interaction:
- **Direction**: is this Archetype→System or System→Archetype?
- **What**: what is communicated or triggered?
- **Observable result**: what can the Archetype see or experience as a consequence?

Challenge any Interaction that describes internal system behavior:

> "You said the system validates the token — is that something the Archetype can observe, or is that happening invisibly? What does the Archetype actually see?"

For unsolicited System→Archetype Interactions:

> "Does the system ever contact or notify {Archetype} without them having done something first? When?"

### Probing for Quality Properties

Failure modes and quality properties surface naturally from Scenarios:

> "What's the cost of this Interaction being slow? Is there a threshold where it stops being acceptable?"

> "What happens if the system goes down mid-Scenario? What does {Archetype} experience?"

Push for specificity. "It should be fast" is not architecturally useful. "The Archetype expects a response within 2 seconds or they assume it has failed" is.

Vague quality properties become open issues with a note that the threshold needs to be established.

### Risk-driven prioritization

After a set of Use Cases has been identified:

> "If we had to pick the two or three Use Cases that would be hardest to get right, or where failure would be most damaging — which would they be?"

These become **architecturally significant Use Cases** that drive the structural viewpoints. Mark them explicitly in `scenarios.md`.

### Cross-viewpoint validation

Once any structural viewpoint has been explored, return to Scenarios to validate:

> "Let's walk through {Scenario} given what we've established about the {viewpoint}. Does the Interaction sequence still make sense? Is anything missing?"

An Interaction that cannot be supported by the current structural view is an inconsistency — capture it as an open issue in the relevant viewpoint.

---

## Output Structures

### `scenarios.md` — Archetypes, Use Cases, and Navigation Index

This file is the primary reference for the Scenarios viewpoint. It describes all Archetypes and Use Cases in full, and provides a navigation index to all Scenario files.

```markdown
# Scenarios Viewpoint

> **Living document.** Last updated: {date}
> Terms used in this document are defined in [../CONTEXT.md](../CONTEXT.md).

## System Summary

{One paragraph: what this system is, who it is for, what it does. Written for a
coding agent or new team member encountering this viewpoint for the first time.}

## Archetypes

### {Archetype Name}

{One paragraph: who this Archetype is, their context, and what distinguishes
them from other Archetypes. Include relevant characteristics: technical sophistication,
frequency of interaction, permissions, volume.}

**Primary concerns**:
- {concern}: {why it matters to this Archetype}

#### Use Cases

##### {Use Case Name}

**Goal**: {One sentence — the job the Archetype is trying to get done. No implementation detail.}
**Architecturally Significant**: Yes / No — {rationale if yes}

**Preconditions**:
- {what must be true before the Archetype can pursue this goal}

**Scenarios**:

| File | Description | Path Type |
|---|---|---|
| [SC-001-successful-login](scenarios/SC-001-successful-login.md) | {brief description} | Happy path |
| [SC-002-incorrect-password](scenarios/SC-002-incorrect-password.md) | {brief description} | Error path |

---

##### {Next Use Case Name}
...

---

### {Next Archetype Name}
...

## Architecturally Significant Use Cases

Use Cases that carry the most architectural weight — hardest to get right or most
damaging to fail:

| Use Case | Archetype | Rationale |
|---|---|---|
| {Use Case Name} | {Archetype} | {why this is architecturally significant} |

## All Scenarios

Full index of all Scenario files for navigation and grep:

| File | Archetype | Use Case | Path Type |
|---|---|---|---|
| [SC-001-successful-login](scenarios/SC-001-successful-login.md) | {Archetype} | {Use Case Name} | Happy path |
| [SC-002-incorrect-password](scenarios/SC-002-incorrect-password.md) | {Archetype} | {Use Case Name} | Error path |

## Cross-Viewpoint Validation

| Scenario | Viewpoint | Status | Notes |
|---|---|---|---|
| SC-001 | Logical | Validated | — |
| SC-002 | Runtime | Open issue | See [runtime.md](runtime.md) |

{Open issues as callouts}
```

---

### `scenarios/SC-NNN-descriptive-name.md` — Scenario File

Each Scenario file is self-contained. It states its Archetype and Use Case so it can be understood without reading `scenarios.md`. The filename is the identifier — it is not repeated inside the file.

```markdown
# {Scenario Name}

> Terms used in this document are defined in [CONTEXT.md](../CONTEXT.md).

**Archetype**: {Archetype Name}
**Use Case**: {Use Case Name}
**Use Case Goal**: {The goal statement from the Use Case}
**Path type**: {Happy path / Error path / Edge case}
**Trigger**: {What causes this Scenario to begin — what the Archetype does or what the System detects}

## Interactions

| # | Interaction | Observable Result |
|---|---|---|
| 1 | {subject + verb + object, e.g. "Archetype submits username and password"} | {what the Archetype can observe, or — if none} |
| 2 | {e.g. "System displays the dashboard"} | {what the Archetype experiences} |
| 3 | {next interaction} | {observable result} |

## Outcome

{What the Archetype experiences at the end of this Scenario — success, failure, or partial result.}

## Architectural Notes

{Decisions, risks, or open issues this Scenario surfaces for the structural viewpoints.
Cross-reference open issues in the relevant viewpoint files. Omit if none.}

{Open issues as callouts}
```

---

## Common Pitfalls

**Use Cases as feature lists.** A Use Case names a goal, not a capability. "Password reset" is a feature. "Regain access to an account after forgetting credentials" is a goal. The latter reveals context and urgency that the former hides.

**Internal system behavior in Scenarios.** If an Interaction describes what the system does internally — validates, processes, stores, routes — it is not a visible Interaction. Ask: "Can the Archetype observe this?" If not, it doesn't belong in the Scenario.

**Single-Scenario Use Cases by default.** Most real Use Cases have multiple paths. Probe for error cases, edge cases, and alternative flows before marking a Use Case as sufficiently explored.

**Archetype conflation.** "Users" is rarely a single Archetype. A registered user and an administrator have different goals, permissions, and failure modes — they are different Archetypes. Probe for variation: new vs. experienced, high-volume vs. occasional, technical vs. non-technical.

**External systems treated as Archetypes.** External systems that interact with the running system are not currently modelled as Archetypes. If one surfaces during exploration, capture it as a named concern linked to the open issue in `scenarios.md` rather than modelling it as an Archetype.

**Unsolicited interactions missed.** Many important system behaviors are System→Archetype without a preceding Archetype→System trigger — notifications, alerts, session expiry warnings. Always ask whether the system ever contacts the Archetype unprompted.

**Quality properties left vague.** "Fast", "secure", and "available" are not Quality Properties — they are aspirations. Push for thresholds. Vague quality properties become open issues.

**Closing Use Cases before error paths.** The happy path is the easiest to capture. The error paths and edge cases are where architectural risk lives. A Use Case with only a happy path Scenario is incomplete by definition.

**Reusing SC numbers.** Scenario numbers are globally unique and permanent. Never reassign a number from a deleted Scenario — leave the gap. This preserves the integrity of cross-references in other viewpoint files.
