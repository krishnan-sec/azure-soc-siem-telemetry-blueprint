# Collection and Forwarding Architectures

This document defines **collection and forwarding architectures** for delivering Azure security telemetry to an **external SOC and SIEM**.

The goal is not “get logs out of Azure.”
The goal is to ensure that telemetry arrives at the SIEM with:

- sufficient fidelity for investigation
- predictable latency
- defensible completeness
- observable failure modes (i.e., you can tell when it breaks)
- operational ownership and repeatability

Many SOC integrations fail not because Azure cannot emit the logs, but because the forwarding architecture introduces:
- silent drops
- inconsistent parsing
- unpredictable delays
- unbounded cost
- brittle onboarding

This document describes architectures, trade-offs, and design constraints.

---

## Architectural Decision Framework

When selecting a forwarding architecture, evaluate each pattern against the following criteria:

### 1) Fidelity
Can you preserve:
- full event payload
- correlation identifiers
- identity context
- resource identifiers (resourceId, subscriptionId, tenantId)
- client IP / device details
- result codes and policy decisions

If fidelity is lost at ingestion, it cannot be recovered later.

---

### 2) Latency and Burst Handling
External SOC workflows have implicit latency assumptions:
- Identity abuse and privilege escalation require near-real-time response
- Investigation and scoping tolerate moderate latency, but not hours
- Burst events (e.g., flow logs, WAF logs) must not cause backpressure that drops critical telemetry

Design must account for peak volume, not average volume.

---

### 3) Completeness and Coverage Drift
A design that works for one subscription often fails across:
- multiple subscriptions
- multiple landing zones
- multiple teams

Your architecture must make it hard to “forget” diagnostic settings and easy to validate coverage.

---

### 4) Failure Observability
A mature telemetry pipeline has explicit answers to:
- How do we know logs stopped?
- How do we detect partial source failure?
- How do we detect parser/normalization failure?
- How do we detect delayed arrival and backlog?

If the pipeline can fail silently, it will.

---

### 5) Security and Access Boundaries
Collection and export mechanisms are security controls.
Design must enforce:
- least privilege
- separation of duties
- protected credentials/secrets
- controlled egress paths
- tamper resistance

---

## Canonical Reference Patterns

This repo uses three canonical patterns:

- Pattern A: **Azure-native collection (Log Analytics / Azure Monitor) → Export to SIEM**
- Pattern B: **Streaming transport (Event Hub) → SIEM**
- Pattern C: **Agent-based collection for workloads → SIEM**

In practice, most enterprises use a **hybrid**:
- Pattern A/B for platform telemetry
- Pattern C for workload/guest telemetry

---

## Pattern A: Azure Monitor / Log Analytics → Export to SIEM

### Description
Azure services emit:
- Activity Logs
- Resource Diagnostic Logs
- Identity-related telemetry (depending on integration path)

These are centralized into one or more Log Analytics Workspaces and then exported to an external SIEM via:
- streaming export
- connector integration
- scheduled export mechanisms 

### Where this pattern fits
This pattern is appropriate when:
- you need centralized governance
- you require consistent onboarding across subscriptions
- you want a single operational control point for diagnostic settings

### Strengths
- Native Azure integration surface
- Centralized configuration and policy enforcement options
- Easier to apply consistent retention and routing
- Common operational model for cloud platform teams

### Limitations / risks
- Export latency can be non-trivial depending on method and load
- If the export pipeline fails, you may accumulate backlog silently
- Workspace architecture decisions (single vs multi) affect cost and blast radius
- Misconfigured diagnostic settings can produce a false sense of completeness

### Common failure modes
- Activity Logs arrive, diagnostics do not (classic “control plane only” visibility)
- Identity telemetry exists in tenant, not forwarded to the SIEM
- Export is enabled but drops large/bursty categories under load
- Parser assumptions break when Azure schemas evolve

---

## Pattern B: Event Hub Streaming → SIEM

### Description
Telemetry streams into Event Hub and is consumed downstream by the SIEM (or intermediate pipeline).
This is commonly used for:
- high volume logs
- near-real-time ingestion
- environments where direct streaming is preferred

### Where this pattern fits
This pattern is appropriate when:
- the SIEM ingestion architecture is streaming-native
- low latency is a requirement
- you need high-throughput transport

### Strengths
- Near-real-time transport (when properly engineered)
- Scales well with volume and burst behavior
- Decouples Azure emission from SIEM ingestion

### Limitations / risks
- Engineering complexity is higher (partitioning, consumer groups, retention)
- Operational maturity is required to prevent silent drops
- Backpressure and consumer lag can create hidden data loss or delays
- It is easy to build a “works in test” pipeline that fails at scale

### Common failure modes
- Event Hub healthy, consumer unhealthy → SOC sees nothing
- Partition/throughput limits exceeded during bursts → drop or delay
- Consumers fail silently due to schema drift
- Teams onboard logs inconsistently without governance

---

## Pattern C: Agent-Based Collection (Workloads) → SIEM

### Description
Workload telemetry (guest OS logs, container logs, application logs) is collected using:
- EDR telemetry
- agents
- forwarders
- collectors

and sent directly to the SIEM or via an intermediate pipeline.

This pattern is typically used for:
- Windows Event Logs / Linux syslog
- application runtime logs
- deep execution context needed for IR

### Where this pattern fits
This pattern is appropriate when:
- deep workload visibility is required
- you need near-real-time host telemetry
- platform logs do not capture execution context

### Strengths
- High-fidelity workload execution evidence
- Mature ingestion models for OS logging
- Can be standardized across cloud and on-prem environments

### Limitations / risks
- Operational overhead (agent lifecycle, upgrades, drift)
- Credential/secrets handling risk if mismanaged
- Coverage drift is common in autoscaling environments
- Requires strong automation to be sustainable

### Common failure modes
- Only “critical servers” have agents; cloud workloads are missed
- Agents are installed but not forwarding due to network egress constraints
- Autoscaled nodes never register in logging pipeline
- EDR-only reliance leads to limited forensic reconstruction

---

## Recommended Hybrid Architecture

A robust external SOC integration typically uses:

- Pattern A or B for:
  - Activity Logs
  - Resource Diagnostic Logs
  - Network security telemetry
- Pattern C for:
  - VM guest logs
  - workload execution logs
  - application / container telemetry

This hybrid approach recognizes that:
- platform telemetry and workload telemetry have different operational characteristics
- a single pipeline rarely satisfies all fidelity and latency needs

---

## Key Design Decisions to Document

Regardless of architecture, document these decisions explicitly:

### 1) Workspace / Hub topology
- single vs multi-workspace
- per-tenant vs per-landing-zone segmentation
- blast radius and access boundary rationale

### 2) Retention and replay strategy
- how long logs are retained at each stage
- whether you can reprocess missed events
- how you handle delayed detection investigations

### 3) Governance and onboarding
- how diagnostic settings are enforced
- how new subscriptions are onboarded
- how exceptions are approved and reviewed

### 4) Pipeline observability
- how you detect missing sources
- how you detect transport backlog
- how you detect parsing failure
- how you detect latency SLA violations

If you cannot observe these, you do not control the pipeline.

---

## Security Considerations

Telemetry pipelines are attack surfaces.

### Threats to consider
- attacker disables diagnostics at the source
- attacker blocks forwarding paths
- attacker floods pipeline to create blind spots
- attacker abuses collection identities / secrets

### Baseline mitigations
- control-plane monitoring for diagnostic setting changes
- least privilege identities for export mechanisms
- restrictive network egress patterns
- alerting on pipeline degradation and source disappearance

---

## Exit Criteria

This document is complete when you can answer, for your chosen architecture:

- Which telemetry sources flow through which path?
- What is the expected latency per source?
- How do you detect missing data per source?
- Who owns fixing each failure mode?
- How do you prove the SOC can investigate using received telemetry?

If those questions cannot be answered, the architecture is not operationally ready.

---
