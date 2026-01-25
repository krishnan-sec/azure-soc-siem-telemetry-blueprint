# Intent and Scope

This document defines the **intent, boundaries, and assumptions** of this repository.

It exists to ensure that readers understand **what problem this repository is solving**, **what it deliberately does not attempt to solve**, and **how its guidance should be interpreted and applied**.

---

## Intent

The intent of this repository is to define **what security telemetry must exist** for an **external SOC and SIEM** to operate effectively in a Microsoft Azure environment.

Specifically, it aims to:

- Establish a clear, defensible definition of *telemetry coverage*
- Articulate why certain logs are **non-negotiable**
- Expose common visibility gaps that only surface during incidents
- Provide a reference that can be used to justify logging decisions to:
  - platform teams
  - SOC providers
  - security leadership

The guidance in this repository is written to survive **real incidents**, not audits or design reviews.

---

## Problem Statement

In many Azure environments, security logging is treated as a configuration exercise:

- Logs are enabled opportunistically
- Defaults are accepted without scrutiny
- Telemetry is scoped based on cost rather than investigation needs
- Success is measured by “logs are on,” not “incidents are solvable”

As a result, organizations frequently discover during incidents that:

- Identity actions cannot be attributed
- Resource changes cannot be contextualized
- Network activity cannot be scoped
- Data access cannot be reconstructed
- SOC investigations stall or become speculative

These failures are rarely due to tooling limitations.  
They are the outcome of **unclear intent and undefined scope**.

---

## Definition of Scope

This repository defines scope along three dimensions:

1. **What telemetry is required**
2. **Where that telemetry must originate**
3. **Why that telemetry is operationally necessary**

It does not attempt to prescribe:
- Specific SIEM products
- Specific alert rules
- Specific Azure deployment models

Instead, it establishes **minimum viable telemetry** that enables effective security operations regardless of tooling choice.

---

## In Scope

The following areas are explicitly in scope:

### Azure Control Surfaces
- Microsoft Entra ID (identity and access)
- Azure Resource Manager (control plane)
- Network security controls
- Workload execution environments
- Data access services
- Security control outputs (e.g., Defender products)

### Operating Model
- External SOC integration
- Telemetry forwarding outside Azure
- SOC visibility without administrative control
- Validation of log presence and quality

### Security Outcomes
- Detection enablement
- Incident investigation
- Scoping and containment
- Post-incident analysis

---

## Out of Scope

The following are intentionally excluded:

- Application business logic logging
- Performance and availability monitoring
- Azure cost optimization
- Non-Azure cloud platforms
- Vendor-specific SIEM tuning or dashboards
- Blue team detection content (queries, rules)

These topics are important but orthogonal to the core problem this repository addresses.

---

## Assumptions

This repository makes several explicit assumptions.  
If these assumptions do not hold, the guidance may require adaptation.

### SOC Model
- The SOC is operated by a third party or separate internal team
- SOC analysts do not have tenant-wide administrative privileges
- SOC access is read-only and investigation-focused

### SIEM Model
- Logs are forwarded out of Azure
- Azure-native analytics are not the primary SOC platform
- Normalization and correlation occur in the SIEM

### Threat Model
- Attacks frequently leverage legitimate identities
- Control plane actions are often part of the attack chain
- Not all malicious activity triggers alerts
- Detection delays are expected and must be tolerated

---

## Definition of “Coverage”

Within this repository, **coverage** is defined as:

> The ability for an external SOC to reconstruct and reason about security-relevant activity in Azure without requiring emergency access changes during an incident.

Coverage does not mean completeness of data.  
It means **sufficiency of evidence**.

---

## Explicit Non-Goals

This repository does **not** attempt to:

- Replace Azure documentation
- Teach Azure fundamentals
- Provide one-size-fits-all solutions
- Eliminate all risk
- Guarantee detection of every attack

Its purpose is to **reduce uncertainty**, not promise perfection.

---

## How This Document Should Be Used

This document should be used to:

- Frame discussions about logging requirements
- Set expectations with SOC providers
- Justify logging costs and retention decisions
- Establish architectural standards
- Identify explicit risk acceptance

If a logging decision contradicts guidance in this repository, it should be:
- intentional
- documented
- reviewed periodically

---

## Accountability Statement

Telemetry gaps described in this repository should be treated as **architectural risk**, not operational oversight.

When incidents cannot be fully investigated, the root cause is often not:
- missing detections
- insufficient tooling

…but **undefined intent and scope at design time**.

---
