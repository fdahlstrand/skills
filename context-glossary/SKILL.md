---
name: context-glossary
description: Build and maintain docs/CONTEXT.md, a project's domain glossary / ubiquitous language. Captures the terms specific to this project's domain — what each term IS, with an opinionated canonical word, an _Avoid_ list of synonyms to not use, and a Flagged Ambiguities section explaining contested naming choices. Use this whenever domain terminology surfaces, when the user says "add this to the glossary", "what do we call this", "update CONTEXT.md", "define this term", or when you notice the same concept being called by several different words. Strictly a glossary — no implementation details, no decisions, no spec content. Use standalone or as part of a grilling session.
---

# context-glossary

`docs/CONTEXT.md` is the project's **ubiquitous language** — the glossary of domain terms everyone (human and agent) should use consistently. It is a glossary and **nothing else**: not a specification, not a scratchpad, not a decision log, not a place for implementation detail.

It lives at `docs/CONTEXT.md` — a durable, committed document, a peer of `docs/ROADMAP.md`, `docs/adr/`, and `docs/architecture/` (not privileged at repo root). It is the **one** domain glossary for the whole project: the architecture skills reference and write into it rather than maintaining their own.

## What CONTEXT.md is for

It pins down the vocabulary of *this project's domain* so the same concept is always called the same thing, in code, in conversation, and in documents. Its highest-value job is **killing synonym drift**: when a concept is referred to by several words, the glossary picks one and tells everyone to stop using the others.

## The four rules

1. **Be opinionated.** When multiple words exist for the same concept, pick the best one and list the others under `_Avoid_`. Don't hedge — a wishy-washy ubiquitous language is worthless. (If the canonical choice is genuinely unsettled, mark the term `WIP` rather than hedging in prose.)

2. **Keep definitions tight.** One or two sentences max. Define what the term **IS**, not what it **does**. "Invoice: a request for payment sent to a customer after delivery" (what it is) — never "the Invoice service generates and sends payment requests" (what it does / how it's built).

3. **Only include terms specific to this project's context.** General programming concepts — timeouts, retries, caches, error types, utility patterns — do **not** belong, even if the project uses them heavily. Before adding a term, ask: *is this a concept unique to this context, or a general programming concept?* Only the former belongs.

4. **Group terms under subheadings when natural clusters emerge.** If all terms belong to one cohesive area, a flat list is fine.

## What never goes in CONTEXT.md

- **Implementation details** — databases, services, libraries, timeouts, data structures, "how it's built." If a term's only available description is implementation mechanics, the term is not yet properly defined: mark it `WIP` and capture the *domain meaning* it needs, rather than recording the mechanics.
- **Behavior / "what it does"** — that drifts toward spec and implementation. Define identity, not behavior.
- **Decisions and rationale** — those are ADRs (the `adr` skill).
- **What the system does / its structure** — that's the Architectural Description (`architecture-description`) and code + tests. There is no `SPECIFICATION.md`.

When grilling or conversation surfaces something like "the token service uses Redis with a 30s TTL," the *term* "Token Service" may belong (defined as *what it is* in the domain), but "Redis, 30s TTL" must be stripped — route it to an ADR if it's a decision, or simply drop it.

## WIP terms

A term is `WIP` when the concept has surfaced and been named but its definition — or the canonical-vs-avoid choice — isn't settled. Mark it with the shared `WIP` token so `grep -rn WIP docs/CONTEXT.md` finds unresolved terms (consistent with the `adr` skill's `Status: WIP` convention). Clear the marker once the definition is agreed.

## Format

Follow this structure (the `_Avoid_` line is omitted when a term has no competing synonyms):

```markdown
# <Context Name>

<One or two sentence description of what this context is and why it exists.>

## Language

**Order**:
A customer's request to buy goods or services.
_Avoid_: Purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request

**Customer**:
A person or organization that places orders.
_Avoid_: Client, buyer, account
```

A `WIP` term looks like:

```markdown
**Settlement** (WIP):
<domain meaning still to be pinned down — what it IS, once agreed>
```

When contested naming choices have been made, record the reasoning in a **Flagged Ambiguities** section at the end:

```markdown
## Flagged Ambiguities

- **"User"** was used to mean both the **Customer** (who places orders) and the **Operator** (who runs the system) — resolved: use **Customer** for the buyer, **Operator** for internal staff. Avoid "user" entirely.
- **"Account"** is avoided because it conflated **Customer** (the organization) with **Login** (the credential) — they are distinct.
```

## Flagged Ambiguities — why this section earns its place

The `_Avoid_` lists say *what* not to use; Flagged Ambiguities says *why* a contested choice was made. It is the most useful section for a reader trying to understand a non-obvious naming decision — "why is 'user' banned, what was confusing about it?" Record an ambiguity here when:

- two words were genuinely conflated and the resolution isn't self-evident from the `_Avoid_` lists alone, or
- a term was deliberately rejected for a subtle reason a future contributor would otherwise re-litigate.

Keep entries short: name the confusion, state the resolution. Don't manufacture them — only contested or genuinely confusing choices warrant an entry. (This mirrors the methodology vocabulary's own Flagged Ambiguities, which the architecture skills maintain for *their* terms.)

## Detecting synonym drift

The skill's distinctive value is noticing when several words name one concept. Definitions are usually stated outright; `_Avoid_` lists rarely are — nobody says "and don't call it a purchase." Infer them: when the same concept gets called "order," "purchase," and "transaction" across a session, pick the canonical term and record the rest as `_Avoid_`. Be opinionated about the choice (rule 1).

One caution: don't conflate genuinely *distinct* domain terms as synonyms. Conflating two real concepts corrupts the language worse than missing a synonym. If two words might be the same concept or might be distinct, and the domain doesn't make it obvious, mark the entry `WIP` and note the ambiguity rather than forcing a merge.

## Seeding project-facing architecture terms

The `architecture-description` skill defines a methodology vocabulary, a subset of which is *project-facing* — terms a project actually adopts in its code and conversation (e.g. **Runtime Container**, **External System**, **Archetype**, **Stakeholder**, **Quality Property**, **Use Case**, **Scenario**). When architecture work surfaces one of these and the project starts *using* it, seed it into `docs/CONTEXT.md` here, following the four rules — **as the project uses it**, not dumped wholesale from the methodology reference. The methodology vocabulary stays internal to the architecture skills; the project glossary gets only the terms this project has actually adopted.

## After writing

State what was added or changed. Flag any `WIP` terms as needing a settled domain definition.
