# Index Structure Reference

Defines the structure and content conventions for `docs/architecture/index.md` — the root file of the Architectural Description. Load when creating or updating the index.

---

## Purpose of the index

`index.md` serves three audiences:
1. **Humans** — orientation at the start of a session; understanding of current state.
2. **Coding agents** — a map to find relevant architectural detail without loading all files.
3. **Elicitation skills** — session resumption without reading every view file.

It must be navigable in isolation: every section useful without the other view files.

---

## File template

```markdown
# Architecture Index

> **Living document.** Best current understanding of the architecture of {System Name}.
> Expected to change as new information surfaces.

## System

{One paragraph: what this system is, who it is for, what problem it solves. Written for a coding agent or new team member encountering it for the first time.}

## Terminology

Domain terms are defined in the shared project glossary [../CONTEXT.md](../CONTEXT.md) (owned by the context-glossary skill). All code, documentation, and conversations about this system use the terms defined there; avoid-terms are listed explicitly. The architecture methodology vocabulary (Viewpoint, Archetype, Runtime Container, etc.) is defined by the architecture-description skill's internal reference, not duplicated here.

## Viewpoint Map

| Viewpoint | Entry Point | Status | Last Updated |
|---|---|---|---|
| Scenarios (+1) | [scenarios.md](scenarios.md) | {Not started / In progress / Current} | {date or —} |
| Logical | [logical.md](logical.md) | {Not started / In progress / Current} | {date or —} |
| Runtime | [runtime.md](runtime.md) | {Not started / In progress / Current} | {date or —} |
| Deployment | [deployment.md](deployment.md) | {Not started / In progress / Current} | {date or —} |
| Development | [development.md](development.md) | {Not started / In progress / Current} | {date or —} |

Note: The Scenarios viewpoint spans multiple files. `scenarios.md` describes all Archetypes and Use Cases and provides a navigation index; individual Scenario files live flat in `scenarios/` with globally unique `SC-NNN` numbers. Read `scenarios.md` first.

## Risk Register

Active architectural risks across all viewpoints. Engineering risks — concerns that, if unaddressed, could cause the system to fail in architecturally significant ways.

| # | Risk | Viewpoint(s) | Severity | Status |
|---|---|---|---|---|
| R-001 | {description} | {viewpoint} | {High / Medium / Low} | {Open / Mitigated} |

## Open Issues Summary

Cross-viewpoint navigation index of open issues. Full detail lives in each viewpoint file as an `[!OPEN]` callout.

| Issue | Viewpoint | File |
|---|---|---|
| {one-line description} | {viewpoint} | [{file}]({file}.md) |

## Session Log

Most recent first.

### {YYYY-MM-DD} — {Session Focus}

**Explored**: {viewpoints and topics covered}
**Decisions**: {key decisions made or confirmed — link ADRs}
**Opened**: {new open issues or risks}
**Closed**: {open issues or risks resolved}
**Next**: {recommended focus, with rationale}
```

---

## Conventions

### Viewpoint status
- **Not started** — no file, or empty.
- **In progress** — exploration begun; open issues remain; not yet a reliable reference.
- **Current** — sufficiently explored for current risk level; open issues documented; safe to reference. "Current" ≠ complete; it means current best understanding is recorded and associated risks are mitigated or captured.

### Risk severity
Assess by: *what is the impact if this risk materializes and we haven't addressed it architecturally?*
- **High** — system cannot be built, or must be fundamentally redesigned.
- **Medium** — significant rework of one or more viewpoints required.
- **Low** — addressable locally without structural change.

### Session log — "Next"
A recommendation, not a commitment. Name a specific viewpoint or risk and why it warrants attention. The user always drives the actual choice.

---

## Coding agent guidance

When an agent reads `index.md` to understand the system:
- Read [../CONTEXT.md](../CONTEXT.md) first — the shared domain glossary defining terms used throughout.
- Check the ADRs in `../adr/` to understand key decisions and rationale before making changes.
- Use **Viewpoint Map** to find the right file for the concern at hand.
- Check **Open Issues Summary** for what is known to be unresolved.
- Don't assume absence of documentation means absence of a decision — check the **Session Log**.

---

## Maintenance rules

- Update `index.md` at the end of every session.
- Domain terms go to the shared `docs/CONTEXT.md` via context-glossary — do not maintain a separate architecture glossary.
- Every `[!OPEN]` issue in a view file must appear in **Open Issues Summary**.
- Every resolved issue must be removed from the view file and recorded as closed in the **Session Log**.
- **Viewpoint Map** status must reflect actual file state — do not mark "Current" while significant open issues remain.
