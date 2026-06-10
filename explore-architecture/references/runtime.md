# Runtime Viewpoint Reference

The Runtime viewpoint describes how the system's logical elements are realized as runtime processes or execution units, and how they interact while the system is running. It answers the question: *what is actually happening when the system executes?*

This is Kruchten's original "Process View," renamed here to "Runtime" for clarity. It bridges the Logical view (what things are) and the Deployment view (where things run).

---

## Purpose

The Runtime view is where concurrency, communication, and dynamic behavior become visible. It is the view most relevant to:
- **Performance and scalability** — how processes are structured determines throughput and latency
- **Reliability and fault tolerance** — how processes communicate determines failure modes
- **Security** — trust boundaries between processes are a runtime concern
- **Developers** — understanding how logical elements map to running processes

The Runtime view is where many architectural risks materialize. If a scenario involves two elements communicating across a process boundary, the nature of that communication (synchronous vs. asynchronous, reliable vs. unreliable, latency characteristics) is a Runtime concern.

---

## Interview Guide

### Opening the Runtime viewpoint

Anchor to architecturally significant scenarios:

> "Let's take {scenario} and think about what's actually running when that happens. What processes or services are alive and doing work?"

If the Logical view has been explored, connect to it:

> "We said the system has {element A} and {element B}. At runtime, are these running in the same process, or separate ones? Why?"

### Identifying runtime processes

For each runtime process or execution unit identified:

**Identity and role**: What logical element(s) does it realize? What is it responsible for at runtime?

**Lifecycle**: How does it start? How does it stop? Is it always running or triggered on demand?

**Concurrency model**: Is it single-threaded? Multi-threaded? Event-driven? Parallel?

> "Can {process} handle multiple requests simultaneously? How?"

### Identifying runtime interactions

For each interaction between processes:

**Communication style**: Synchronous (caller waits) or asynchronous (caller continues)?

> "When {process A} needs something from {process B}, does A wait for the answer, or does it continue and handle the response later?"

**Coupling**: What happens to A if B is unavailable?

> "If {process B} goes down, what happens to {process A}? Does it fail? Queue? Degrade gracefully?"

**Data in flight**: What is being communicated? Is it sensitive? Does it need to be durable?

**Latency and volume**: How often does this interaction happen? What are the latency expectations?

### Concurrency and parallelism

> "Are there parts of the system that need to do multiple things at the same time? Where does concurrency come in?"

Probe for:
- Race conditions and shared state
- Locking and coordination
- Event ordering guarantees

### Failure and recovery

The Runtime view is where failure modes become concrete:

> "What happens when {process} fails mid-operation? Is there any work in progress that could be lost or corrupted?"

> "How does the system recover from {failure mode}? Is that automatic or manual?"

### Connecting to quality properties

Runtime is the primary view for performance, scalability, and reliability:

> "Given the volume and latency expectations from {scenario} — does this runtime structure support that? Where could it break down?"

---

## Output Structure

```markdown
# Runtime Viewpoint

> **Living document.** Last updated: {date}
> Terms used in this document are defined in [../CONTEXT.md](../CONTEXT.md).

## Overview

{One paragraph describing the runtime topology — what processes exist and how they relate at the highest level. Include the dominant communication style (e.g. request/response, event-driven, batch).}

## Runtime Processes

### {Process Name}

**Realizes**: {Logical element(s) from the Logical view}
**Lifecycle**: {always-on / triggered / scheduled / on-demand}
**Concurrency model**: {single-threaded / multi-threaded / event-driven / actor-based / etc.}

**Responsibilities at runtime**:
{What this process does while running}

**Quality properties**:
- {property}: {how it is addressed by this process's design}

---

### {Process Name}
...

## Runtime Interactions

### {Interaction Name}: {Process A} → {Process B}

**Style**: {Synchronous / Asynchronous}
**Protocol**: {HTTP/REST / gRPC / message queue / event stream / shared memory / etc.}
**Coupling**: {what happens to A if B is unavailable}
**Data**: {what is being communicated}
**Volume and latency**: {frequency and latency characteristics, if known}

---

## Trust Boundaries

{Description of which processes trust each other and which do not. Trust boundaries are security-relevant — communication across a trust boundary requires authentication and/or authorization.}

| Boundary | Processes | Authentication Required | Notes |
|---|---|---|---|
| {name} | {A} / {B} | Yes / No | {notes} |

## Scenario Walkthroughs

Trace of architecturally significant scenarios through the runtime structure:

### S-001: {Scenario Name}

1. {Process A} receives {trigger}
2. {Process A} calls {Process B} synchronously via {protocol}
3. {Process B} responds with {data}
4. ...

**Bottlenecks or risks identified**: {any concerns surfaced by this walkthrough}

{Open issues as callouts}
```

---

## Common Pitfalls

**Conflating Logical and Runtime elements.** A logical element (e.g. "Order Management") may be realized by multiple runtime processes, or one runtime process may realize multiple logical elements. These are distinct concerns — keep them separate.

**Ignoring failure modes.** Runtime is where failure becomes real. Every interaction should have a named failure mode and a known or intended recovery strategy.

**Synchronous by default.** Many systems default to synchronous communication without questioning whether it's appropriate. Probe: does the caller need to wait? What's the cost of waiting if the callee is slow?

**Missing concurrency concerns.** Shared state between concurrent processes is a risk. If two processes write to the same data, ask: how is consistency maintained?

**Treating latency as a deployment concern.** Network latency between processes is affected by deployment topology, but the decision to make a call synchronous vs. asynchronous is a Runtime decision. Capture it here.
