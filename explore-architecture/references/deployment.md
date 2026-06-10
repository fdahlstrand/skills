# Deployment Viewpoint Reference

The Deployment viewpoint describes the infrastructure and topology on which the system runs — where processes are deployed, how they are connected, and what the physical or virtual environment looks like. It answers the question: *where does the system run, and how does that shape what it can do?*

This is Kruchten's original "Physical View," renamed here to "Deployment" for clarity. It maps the Runtime view's processes onto infrastructure.

---

## Purpose

The Deployment view is where infrastructure constraints, operational requirements, and distribution decisions become explicit. It is the primary view for:
- **Operators and infrastructure teams** — what needs to be provisioned and maintained
- **Availability and disaster recovery** — how the topology supports or limits resilience
- **Security** — network boundaries, data residency, access controls
- **Performance** — latency introduced by distribution; network topology effects
- **Scalability** — where horizontal or vertical scaling is possible

The Deployment view does not describe what processes do (Runtime) or how they are organized in code (Development). It describes where they run and how the environment is structured.

---

## Interview Guide

### Opening the Deployment viewpoint

Anchor to the Runtime view if it exists:

> "We've talked about what processes run — now let's talk about where they run. If you were provisioning infrastructure for this system today, what would you set up?"

If starting without a Runtime view:

> "Thinking about the real-world environment — what machines, services, or environments does this system need to exist?"

### Identifying deployment targets

For each deployment target (server, container, cloud service, edge device, etc.):

**Nature**: Physical server? Virtual machine? Container? Managed cloud service? Edge device? Mobile device?

**Role**: What process(es) from the Runtime view run here?

**Scale**: Fixed count? Auto-scaling? How many instances at peak?

**Location**: Single region? Multi-region? On-premises? Cloud provider? Data residency constraints?

> "Are there any constraints on where data can live — legal, regulatory, or contractual?"

### Identifying network topology

For each network connection between deployment targets:

**Nature**: Public internet? Private network? VPN? CDN?

**Security**: Is traffic encrypted? Authenticated? Firewalled?

**Latency and bandwidth**: What are the characteristics of this link? Does the system depend on low latency here?

> "Is there any communication between parts of the system that crosses an untrusted network? How is that handled?"

### Availability and resilience

> "What happens if {deployment target} goes down? Does the system stop working, degrade, or continue normally?"

Probe for:
- Single points of failure
- Redundancy and failover strategies
- Recovery time objectives (RTO) and recovery point objectives (RPO)
- Backup and data durability strategies

> "Is there a target for how quickly the system must recover from failure? How much data loss is acceptable?"

### Scaling strategy

> "If the load on {process} doubles — what happens? Is there a plan to scale?"

Probe for:
- Horizontal vs. vertical scaling
- Stateless vs. stateful processes (stateful processes are harder to scale horizontally)
- Data store scaling (often the hardest scaling problem)
- Auto-scaling triggers and limits

### Operational concerns

> "How is the system updated? Can it be updated without downtime?"

> "How is the system monitored? How would an operator know something has gone wrong?"

These are the beginning of operational concerns — capture them here but note they may warrant a dedicated operational perspective if they are architecturally significant.

### Connecting to quality properties

> "Given the availability target from {scenario} — does this topology support it? Where is the risk?"

Deployment is where quality property targets meet physical reality. Push for specifics when quality properties have been stated abstractly.

---

## Output Structure

```markdown
# Deployment Viewpoint

> **Living document.** Last updated: {date}
> Terms used in this document are defined in [../CONTEXT.md](../CONTEXT.md).

## Overview

{One paragraph describing the overall deployment topology — the major environments, their relationships, and the dominant deployment model (e.g. cloud-native, on-premises, hybrid, edge). Written for an operator or infrastructure engineer encountering this for the first time.}

## Environments

### {Environment Name} (e.g. Production, Staging, Development)

**Purpose**: {what this environment is for}
**Provider**: {cloud provider / on-premises / hybrid}
**Region(s)**: {geographic regions, if applicable}
**Data residency constraints**: {legal or regulatory constraints, if any}

#### Deployment Targets

| Target | Type | Hosts | Count / Scaling | Notes |
|---|---|---|---|---|
| {name} | {VM / container / managed service / etc.} | {Runtime process(es)} | {fixed N / auto-scales N–M} | {notes} |

---

## Network Topology

### {Network Segment Name}

**Type**: {public internet / private VPC / VPN / etc.}
**Connects**: {deployment targets or environments}
**Security controls**: {TLS / mTLS / firewall rules / etc.}
**Latency characteristics**: {expected latency range, if known}

---

## Availability Design

**Availability target**: {e.g. 99.9% uptime, defined in scenarios or quality properties}

| Single Point of Failure | Mitigation | Status |
|---|---|---|
| {component} | {redundancy strategy} | {In place / Planned / Open issue} |

**Recovery targets**:
- RTO (Recovery Time Objective): {how quickly must the system recover?}
- RPO (Recovery Point Objective): {how much data loss is acceptable?}

**Backup strategy**: {how data is backed up, frequency, tested recovery}

## Scaling Strategy

| Process | Scaling Approach | Trigger | Limits |
|---|---|---|---|
| {process} | {horizontal / vertical / none} | {metric and threshold} | {min / max instances} |

**Known scaling constraints**: {processes that cannot be scaled horizontally, data stores with scaling challenges, etc.}

## Operational Observability

{Brief description of how the system is monitored, how failures are detected, and how operators are alerted. Flag as open issue if not yet defined.}

## Scenario Mapping

| Runtime Process | Deployment Target | Environment |
|---|---|---|
| {process} | {target} | {environment} |

{Open issues as callouts}
```

---

## Common Pitfalls

**Infrastructure diagrams without process mapping.** A diagram showing AWS services without showing what runs on them is an infrastructure inventory, not a Deployment view. Always map Runtime processes to deployment targets.

**Ignoring operational concerns.** How the system is updated, monitored, and recovered from failure are deployment concerns. They are easy to defer and expensive to retrofit.

**Single points of failure left unnamed.** If a deployment target has no redundancy, name it explicitly. It may be intentional (cost trade-off) or an oversight — either way it should be visible.

**Data stores treated as afterthoughts.** Data stores are often the hardest part of any scaling or availability strategy. Probe them specifically: how are they backed up? How do they handle failure? How would they scale?

**Environment parity ignored.** If development and production environments differ significantly, that is a risk. Flag it.
