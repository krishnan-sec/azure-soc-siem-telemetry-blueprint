# Azure SOC / SIEM Telemetry Blueprint

This repository defines the security telemetry required for an external SOC and SIEM to perform credible monitoring, detection, investigation, and incident response in Microsoft Azure.

It is written from the perspective of security architecture and operational reality, not from tooling documentation or vendor guidance.

The focus is not on enabling logs, but on ensuring that when an incident occurs, the SOC can reconstruct what happened with confidence.

---

## Why this repository exists

In many Azure environments:

- Logging is enabled inconsistently
- Activity Logs are mistaken for full visibility
- Identity telemetry is partial
- Network and workload telemetry are treated as optional
- SOCs receive alerts without investigative context

These conditions do not usually surface during design reviews.  
They surface during incidents, when attribution is unclear, timelines are incomplete, and containment decisions are delayed.

This repository exists to prevent that outcome.

---

## Core premise

Security monitoring in Azure must be designed around planes of control, not individual services.

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

## Design principles

The guidance in this repository follows these principles:

1. **Investigation over detection**  
   Detection without context increases mean-time-to-respond.

2. **Identity-first threat model**  
   Most Azure incidents leverage legitimate identities, not exploits.

3. **Telemetry is a security control**  
   Missing logs represent explicit risk acceptance.

4. **Validation is mandatory**  
   Enabled logging without verification creates false confidence.

5. **Least privilege applies to SOC access**  
   Visibility should not require the ability to change the environment.

---
## Repository structure

### `docs/`

Authoritative architectural guidance.  

These documents define *why* telemetry is required, *what* must be collected, *how* it is transported, *how* it is made usable, and *how* it is validated.

- `00-intent-and-scope.md`  
  Defines purpose, assumptions, and boundaries.

- `01-planes-and-control-points.md`  
  Establishes the plane-based security model used throughout the repository.

- `02-required-telemetry.md`  
  Defines non-negotiable telemetry required for credible investigation.

- `03-collection-and-forwarding-architectures.md`  
  Describes telemetry transport patterns and their operational trade-offs.

- `04-normalization-and-field-mapping.md`  
  Defines how telemetry must be structured for SOC usability and correlation.

- `05-validation-tests-and-acceptance-criteria.md`  
  Establishes how telemetry is proven to be present, timely, and usable.

- `06-operating-model-rbac-and-access.md`  
  Defines SOC access boundaries, separation of duties, and escalation models.

These documents are intended to be read in order.

---

## How this repository should be used

This repository is intended to be used as:

- a reference architecture for Azure telemetry design
- a standard for onboarding subscriptions and SOC providers
- a review artifact during security architecture assessments
- a defensive justification for logging, retention, and access decisions

It is most effective when treated as a living standard, not a one-time deliverable.


---

## Final note


If telemetry defined in this repository is missing, the organization should assume:

- increased detection latency
- reduced investigation confidence
- slower containment
- higher operational risk during incidents

These outcomes are not tooling failures.
They are **design decisions**.

---

## License and usage 

This repository is intended for reference and adaptation.
Modify it to fit your environment, operating model, and risk appetite.
