# Operating Model, RBAC, and Access

This document defines the operating model and access boundaries required for an external SOC and SIEM to operate effectively in a Microsoft Azure environment.

Telemetry alone does not enable incident response.
People, permissions, and decision boundaries determine whether investigations succeed or stall.

This document exists to ensure that access constraints are designed deliberately, rather than negotiated during incidents.

---

## Purpose of the Operating Model

The purpose of this operating model is to:

- Enable the SOC to investigate incidents independently
- Prevent ad-hoc access escalation during incidents
- Preserve separation of duties
- Minimize operational friction under time pressure
- Ensure accountability for changes and investigations

The model assumes that:
- The SOC is not the platform owner
- The SOC does not administer Azure resources
- The SOC must still be effective during high-severity incidents

---

## Core Principle

**The SOC must be able to investigate without being able to change the environment.**

Any model that violates this principle introduces:
- audit risk
- operational risk
- incident response ambiguity

---

## Roles and Responsibilities

### Platform / Cloud Engineering

Owns:
- Azure subscription and tenant configuration
- Diagnostic settings and telemetry sources
- Resource deployment and configuration
- Identity and access policy enforcement

Is accountable for:
- Ensuring required telemetry is emitted
- Maintaining pipeline reliability
- Remediating configuration drift

---

### Security Engineering

Owns:
- Telemetry architecture
- Normalization and field mapping
- SIEM ingestion and parsing
- Validation frameworks and evidence

Is accountable for:
- Ensuring telemetry is usable for investigation
- Maintaining correlation integrity
- Detecting pipeline degradation

---

### Security Operations Center (SOC)

Owns:
- Monitoring
- Investigation
- Triage
- Incident escalation

Is accountable for:
- Using provided telemetry effectively
- Identifying gaps in investigative capability
- Escalating access limitations proactively

The SOC is **not** responsible for:
- Enabling logging
- Modifying Azure resources
- Fixing pipeline failures

---

## SOC Access Requirements (Minimum Viable Access)

The SOC must be granted **read-only access** sufficient to investigate incidents end-to-end.

### Required Capabilities

At minimum, the SOC must be able to:

- Query all ingested telemetry in the SIEM
- Pivot from identities to resources
- Enumerate subscriptions and resource hierarchy
- Resolve resource IDs to meaningful context
- Validate whether telemetry gaps exist

Access that supports alerts but not investigation is **insufficient**.

---

## Azure Access Model

### Tenant-Level Access

The SOC **should not** have:
- Global Administrator
- Privileged Role Administrator
- Any role capable of modifying identity or policy

The SOC **may** have:
- Directory read permissions sufficient for identity resolution
- Access to identity metadata (users, service principals, roles)

Identity visibility without modification authority is the goal.

---

### Subscription-Level Access

Recommended baseline:
- Reader role at subscription or management group scope

This enables:
- Asset discovery
- Resource context resolution
- Correlation between telemetry and environment state

Reader access **does not** enable:
- Resource modification
- Configuration changes
- Security control bypass

---

### Resource-Level Exceptions

In some environments, additional read-only roles may be required for:
- Kubernetes cluster inspection
- Network topology understanding
- Data service metadata resolution

Any such exceptions must be:
- Explicit
- Time-bound
- Reviewed periodically

---

## Separation of Duties

The following boundaries must be enforced:

| Function | Allowed to Investigate | Allowed to Change |
|--------|-----------------------|------------------|
| SOC | Yes | No |
| Platform Engineering | Limited | Yes |
| Security Engineering | Yes | Limited |
| Incident Commander | Yes | Coordinated |

Blurring these boundaries introduces:
- audit challenges
- unclear accountability
- increased blast radius during incidents

---

## Incident Escalation Model

### Normal Operations

- SOC investigates using telemetry and read-only access
- Findings are documented and escalated as needed
- No access changes occur during normal investigation

---

### Elevated Incidents

When read-only access is insufficient:

- SOC documents the access limitation
- Escalation follows a predefined path
- Temporary access may be granted to a **separate responder role**
- All access changes are logged and reviewed post-incident

Ad-hoc access grants during incidents are an anti-pattern.

---

## Anti-Patterns (Explicitly Discouraged)

The following patterns are known to degrade incident response:

- SOC with Global Administrator access
- “Just-in-time” access negotiated mid-incident
- Screenshot-based investigation
- Platform engineers acting as proxies for SOC queries
- SOC investigations that depend on Slack or email context

These patterns scale poorly and fail under pressure.

---

## Telemetry Access vs. Platform Access

It is acceptable and often preferable for the SOC to have:

- Deep telemetry access
- Broad query capability
- Long retention visibility

…while having **no ability to change Azure state**.

Telemetry access should never be constrained by fear of platform access.

---

## Auditing and Review

Access models must be reviewed:

- During SOC onboarding
- After major platform changes
- Following high-severity incidents
- At least annually

Reviews should answer:
- Can the SOC still investigate effectively?
- Have access boundaries drifted?
- Are escalation paths clear and exercised?

---

## Exit Criteria

This operating model is considered effective when:

- SOC investigations do not require emergency access changes
- Platform teams are not embedded in SOC workflows
- Access decisions are predictable and documented
- Incident reviews focus on attacker behavior, not access blockers

---

## Final Position

- Telemetry enables visibility.
- Architecture enables correlation.
- Access enables action.

Without a deliberate operating model, even the best telemetry pipeline will fail under real incident conditions.

This document exists to ensure that failure is prevented by design.

---
