# Azure SOC / SIEM Telemetry Blueprint

This repository defines the **security telemetry required** for an **external SOC and SIEM** to perform **credible monitoring, detection, investigation, and incident response** in Microsoft Azure.

It is written from the perspective of **security architecture and operational reality**, not from tooling documentation or vendor guidance.

The focus is not on enabling logs, but on ensuring that when an incident occurs, the SOC can **reconstruct what happened with confidence**.

---

## Why this repository exists

In many Azure environments:

- Logging is enabled inconsistently
- Activity Logs are mistaken for full visibility
- Identity telemetry is partial
- Network and workload telemetry are treated as optional
- SOCs receive alerts without investigative context

These conditions do not usually surface during design reviews.  
They surface **during incidents**, when attribution is unclear, timelines are incomplete, and containment decisions are delayed.

This repository exists to prevent that outcome.

---

## Core premise

Security monitoring in Azure must be designed around **planes of control**, not individual services.

Each plane answers a different class of investigative question:

- **Identity plane**: who authenticated, how access was obtained, and under what conditions
- **Control plane**: who changed the environment and what intent was expressed
- **Network plane**: how systems communicated and where traffic flowed
- **Workload plane**: what executed and under which identity
- **Data plane**: what data was accessed and whether access was expected
- **Security control plane**: which protections triggered and why

If telemetry from any plane is missing, incident response becomes speculative.

---

## What “coverage” means in this context

Coverage does **not** mean:
- All available logs are enabled
- Logs exist somewhere in Azure
- Alerts are firing

Coverage means that an external SOC can reliably answer, without ad-hoc access requests:

1. **Who** initiated the action (human, workload, automation)
2. **What** action actually occurred at runtime
3. **Where** it occurred (tenant, subscription, resource, region)
4. **How** access was obtained or privileges were escalated
5. **What** systems or data were impacted
6. **Whether** activity was allowed, anomalous, or malicious

If any of these cannot be answered with evidence, coverage is incomplete.

---

## Assumptions

This repository assumes:

- The SOC is **external** to the Azure tenant
- The SIEM is **not Azure Sentinel as the primary SOC**
- SOC analysts do **not** have administrative access to Azure
- Telemetry is forwarded out of Azure to an external platform
- Logging decisions must survive real incidents, not just audits

Where these assumptions do not hold, guidance may need adaptation.

---

## Design principles used throughout this repo

1. **Investigation over detection**  
   Detection without context increases mean-time-to-respond.

2. **Identity-first threat model**  
   Most Azure incidents leverage legitimate identities, not exploits.

3. **Telemetry is a security control**  
   Missing logs represent an explicit risk acceptance.

4. **Validation is mandatory**  
   Enabled logging without verification is a false sense of security.

5. **Least privilege applies to SOC access**  
   Visibility should not require write access.

---

## Repository structure

### `docs/`
Authoritative architectural guidance and rationale.

- `00-intent-and-scope.md`  
  Purpose, assumptions, and definition of coverage.

- `01-planes-and-control-points.md`  
  Azure planes of control and their investigative role.

- `02-required-telemetry.md`  
  **Core document** defining non-negotiable telemetry per plane.

- `03-collection-and-forwarding-architectures.md`  
  Log routing patterns and their operational trade-offs.

- `04-normalization-and-field-mapping.md`  
  How telemetry must be structured for effective SOC use.

- `05-validation-tests-and-acceptance-criteria.md`  
  Tests that prove telemetry is present, timely, and usable.

- `06-operating-model-rbac-and-access.md`  
  SOC access patterns, separation of duties, and escalation models.

- `07-common-failures-and-risk-acceptance.md`  
  Observed failure modes and how they manifest during incidents.

- `08-reference-service-matrix.md`  
  Service-by-service telemetry requirements and validation guidance.

---

### `checklists/`
Operational artifacts intended for repeated use.

- Baseline telemetry checklist
- Subscription onboarding checklist
- SOC readiness checklist
- Periodic validation checklist

These are designed to **enforce standards**, not replace architecture.

---

### `diagrams/`
Mermaid diagrams illustrating:
- Azure planes of control
- Telemetry data flows
- SOC onboarding lifecycle

Diagrams are explanatory, not decorative.

---

### `templates/`
Reusable templates for extending the repo:
- Per-service documentation
- Validation runbooks
- SOC onboarding artifacts

---

## How this repository should be used

Recommended approach:

1. **Architecture teams** use this repo to define telemetry standards
2. **Platform teams** implement logging accordingly
3. **SOC teams** validate ingestion and investigative usability
4. **Security leadership** uses it to understand risk acceptance

This repository is most effective when treated as a **living standard**, not a one-time deliverable.

---

## Final note

If telemetry described in this repository is missing, the organization should assume:

- Increased detection latency
- Reduced investigation confidence
- Slower containment
- Higher operational risk during incidents

This is not a tooling limitation.  
It is a **design decision**.

---

## License and usage 

This repository is intended for reference and adaptation.
Modify it to fit your environment, operating model, and risk appetite.
