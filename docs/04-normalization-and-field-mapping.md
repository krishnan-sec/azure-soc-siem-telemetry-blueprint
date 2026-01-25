# Normalization and Field Mapping for SOC Usability

This document defines how Azure security telemetry **must be normalized and mapped** when ingested into an external SOC SIEM.

The objective is not syntactic consistency.
The objective is **investigative fluency**.

If analysts cannot pivot across identity, control plane, network, workload, and data events using a shared mental model, the telemetry pipeline has failed — regardless of how many logs are ingested.

---

## Why Normalization Is a Security Control

Normalization is often treated as a convenience or reporting concern.  
In practice, it is a **security control**.

During incidents, analysts must:
- correlate events across disparate sources
- move quickly between planes of control
- reason under time pressure and incomplete information

Raw Azure telemetry is:
- highly verbose
- schema-diverse
- inconsistent across services
- subject to schema evolution

Without deliberate normalization, investigation time increases non-linearly with incident complexity.

---

## Design Principles

All normalization and field mapping decisions in this repository follow these principles:

1. **Preserve raw data**  
   Normalization must never destroy or overwrite original fields.

2. **Normalize for questions, not services**  
   Fields exist to answer investigative questions, not to mirror Azure schemas.

3. **Identity is the primary join key**  
   Normalization must prioritize identity correlation across planes.

4. **Ambiguity must be explicit**  
   If attribution is unclear, it must be visible, not hidden.

5. **Consistency beats completeness**  
   A smaller, reliable field set is more valuable than a large, inconsistent one.

---

## Core Investigative Questions

Normalization must support the following queries without service-specific knowledge:

- Who initiated the action?
- Was the identity human or non-human?
- What resource or data was affected?
- What operation occurred?
- Was it allowed, blocked, or partially successful?
- Where did it originate?
- How did it propagate?

If answering these questions requires deep Azure expertise, normalization has failed.

---

## Canonical Field Categories

The following canonical field categories should exist in the SIEM, regardless of source:

1. **Identity**
2. **Action / Operation**
3. **Target / Resource**
4. **Outcome**
5. **Network Context**
6. **Execution Context**
7. **Time**

Each category is described below.

---

## 1. Identity Fields

Identity fields enable attribution and correlation.

### Required normalized fields

- `actor.id`  
  Stable identifier for the acting identity (user object ID, service principal ID)

- `actor.type`  
  human | service_principal | managed_identity | unknown

- `actor.name`  
  User principal name, app display name, or workload identity label

- `actor.tenant_id`

- `actor.auth_method`  
  password | MFA | certificate | token | managed_identity | unknown

### Notes

- Normalize *who acted*, not *who exists*
- Prefer immutable identifiers over display names
- Preserve original identity claims for audit and re-evaluation

### Common failures

- Overwriting user and service principal fields
- Losing distinction between interactive and non-interactive activity
- Relying on display names that change over time

---

## 2. Action / Operation Fields

These fields describe **what happened**.

### Required normalized fields

- `action.type`  
  create | update | delete | access | execute | authenticate | authorize

- `action.name`  
  Operation name or API invoked

- `action.category`  
  identity | control_plane | network | workload | data | security_control

### Notes

- Action categorization enables plane-level reasoning
- Action names should remain close to source semantics

### Common failures

- Mapping everything to generic “activity”
- Losing operation granularity during parsing
- Mixing control plane intent with runtime execution

---

## 3. Target / Resource Fields

These fields describe **what was acted upon**.

### Required normalized fields

- `target.resource_id`
- `target.resource_type`
- `target.subscription_id`
- `target.resource_group`
- `target.region`

### Notes

- Resource identifiers are primary pivot points during scoping
- Resource hierarchy should be preserved for blast radius analysis

### Common failures

- Truncating or hashing resource IDs
- Losing subscription context
- Treating tenant-level events as resource-less

---

## 4. Outcome Fields

Outcome fields describe **what actually happened**, not what was attempted.

### Required normalized fields

- `outcome.result`  
  success | failure | partial | blocked | unknown

- `outcome.reason`  
  policy | authentication | authorization | error | unknown

### Notes

- “Failure” and “blocked” are not equivalent
- Partial success is common and often overlooked

### Common failures

- Inferring success based on absence of error
- Dropping denial context
- Collapsing all failures into a single category

---

## 5. Network Context Fields

Network fields provide **scope and propagation context**.

### Required normalized fields

- `network.source.ip`
- `network.destination.ip`
- `network.direction`  
  inbound | outbound | lateral | unknown

- `network.protocol`
- `network.port`

### Notes

- Network fields are often missing from identity and control plane logs
- Preserve network data when available; do not fabricate

### Common failures

- Dropping network fields during normalization
- Overwriting source/destination semantics
- Assuming “no network data” means “no network activity”

---

## 6. Execution Context Fields

Execution fields describe **how and where activity ran**.

### Required normalized fields (where applicable)

- `execution.host`
- `execution.process`
- `execution.command_line`
- `execution.container`
- `execution.runtime`

### Notes

- Execution context is plane-specific
- Absence of execution fields must be explicit

### Common failures

- Treating execution context as optional metadata
- Losing process lineage during parsing
- No distinction between control-plane and runtime events

---

## 7. Time Fields

Time is the backbone of investigation.

### Required normalized fields

- `event.timestamp` (UTC)
- `event.ingested_timestamp`
- `event.source_timezone` (if applicable)

### Notes

- Preserve original timestamps
- Normalize to UTC for correlation
- Track ingestion delays explicitly

### Common failures

- Overwriting event time with ingestion time
- Mixing local and UTC timestamps
- No ability to reason about latency

---

## Cross-Plane Correlation Strategy

Normalization must enable joins across planes:

- Identity ↔ Control Plane (who changed what)
- Identity ↔ Data Plane (who accessed data)
- Control Plane ↔ Workload (what configuration enabled execution)
- Network ↔ Data Plane (how data moved)

This requires:
- consistent identity identifiers
- stable resource identifiers
- explicit plane categorization

---

## Handling Ambiguity and Partial Data

Not all events will contain full context.

### Required behavior

- Missing fields must be explicit, not inferred
- Unknown attribution should be labeled as such
- Partial context should not be “filled in” heuristically

False certainty is more damaging than acknowledged uncertainty.

---

## Schema Evolution and Drift

Azure schemas evolve frequently.

Normalization pipelines must:
- tolerate new fields
- avoid hard failures on schema changes
- alert on parsing degradation
- preserve raw events for reprocessing

Schema drift is not an exception, it is normal.

---

## Final Position

Normalization is not a reporting concern.
It is not an optimization.
It is a prerequisite for effective incident response.

If normalization decisions are deferred or treated as optional, the SOC will eventually be forced to investigate directly in raw telemetry under pressure, during an incident, when time matters most.

This document exists to prevent that failure.

---
