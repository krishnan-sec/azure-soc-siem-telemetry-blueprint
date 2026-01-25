# Required Telemetry for External SOC and SIEM Operations

This document defines the **minimum required security telemetry** that must be collected and forwarded from Microsoft Azure to an **external SOC and SIEM** to support credible detection, investigation, and incident response.

This is not an inventory of all available logs.
It is a definition of **operationally necessary evidence**.

Any telemetry not collected should be treated as an **explicit and documented risk acceptance**.

---

## How to Read This Document

Telemetry requirements are organized by **plane of control**, not by Azure service.

For each plane, this document describes:
- Why telemetry from that plane is required
- What questions the SOC must be able to answer
- Which telemetry sources are non-negotiable
- Common failure patterns observed during incidents

The intent is to align logging decisions with **incident reality**, not platform defaults.

---

## Identity Plane Telemetry (Microsoft Entra ID)

### Why identity telemetry is foundational

In Azure, identity is the primary security boundary.

Most impactful incidents:
- Do not exploit software vulnerabilities
- Do not bypass network controls
- Do not require malware

They exploit **legitimate identity mechanisms**:
- Token issuance
- Consent grants
- Role assignments
- Conditional Access gaps
- Automation and service principals

Without comprehensive identity telemetry, all downstream activity becomes difficult or impossible to attribute.

---

### Required telemetry

The following telemetry **must be ingested continuously** into the SIEM:

- Entra ID **Interactive Sign-in Logs**
- Entra ID **Non-Interactive Sign-in Logs**
- Entra ID **Audit Logs**
- Privileged Identity Management (PIM) logs, if PIM is in use
- Identity Protection risk events, if licensed

Logs must include full context:
- user or service principal identifiers
- authentication method
- Conditional Access evaluation results
- IP address and location
- timestamps with sufficient precision

---

### Non-interactive sign-ins are non-negotiable

Non-interactive sign-ins represent:
- Service principals
- Managed identities
- Workload automation
- Token replay and abuse
- API-driven access

Excluding these logs creates a blind spot where:
- Resources are modified
- Data is accessed
- Network paths are opened

…with **no corresponding human sign-in event**.

This gap is one of the most frequently exploited and least understood weaknesses in Azure monitoring.

---

### Common identity telemetry failures

- Only interactive sign-ins are forwarded
- Logs are retained in Entra ID but not exported
- PIM is enabled, but activation events are ignored
- Identity logs are treated as “audit-only” data

Each of these failures directly increases investigation time and reduces confidence during incidents.

---

## Control Plane Telemetry 

### Why the control plane matters

The control plane expresses **intent**.

It answers:
- What change was requested?
- By which identity?
- Against which resource?
- Was the request allowed or denied?

However, intent alone is insufficient without execution context.

---

### Required telemetry

The following telemetry is mandatory:

- **Azure Activity Logs** for all subscriptions
- **Resource diagnostic logs** for security-relevant services, including at minimum:
  - Key Vault
  - Storage Accounts
  - Azure Firewall
  - Web Application Firewall (WAF)
  - Application Gateway / Front Door
  - Azure Kubernetes Service (control plane where available)
  - SQL Database and Managed Instance auditing

Diagnostic logs must be:
- Enabled consistently
- Routed centrally
- Forwarded to the SIEM without filtering

---

### The Activity Log misconception

Azure Activity Logs capture **management operations**, not runtime behavior.

Relying solely on Activity Logs results in investigations that end at:
> “A setting was changed, but we cannot determine its impact.”

Examples:
- Firewall rule added, but no traffic logs
- Key Vault firewall opened, but no access logs
- Storage account modified, but no object access visibility

In all cases, the most important activity occurs **after** the Activity Log entry.

---

### Common control plane telemetry failures

- Partial diagnostic coverage across subscriptions
- Diagnostics enabled only for “critical” resources
- Lack of correlation between Activity Logs and diagnostics
- Short retention periods that prevent historical analysis

---

## Network Plane Telemetry

### Why network telemetry remains essential

Even in identity-driven attacks, networks determine:
- Reach
- Scope
- Data movement
- Containment feasibility

Network telemetry is essential for answering **where activity propagated**.

---

### Required telemetry

At a minimum, the SOC must receive:

- Network Security Group (NSG) Flow Logs 
- Azure Firewall logs, if deployed
- WAF logs, if deployed
- Bastion, VPN, or private connectivity logs where applicable

Flow logs must cover:
- Internet-facing traffic
- East–west traffic for critical workloads

---

### Network telemetry limitations

Network logs alone do not:
- Identify the initiating identity
- Confirm execution on workloads
- Prove malicious intent

They are a **scoping and confirmation control**, not a primary detection mechanism.

---

### Common network telemetry failures

- Logging only ingress traffic
- No visibility into lateral movement
- Assuming EDR replaces flow logs

These gaps result in delayed containment and incomplete incident scoping.

---

## Workload Plane Telemetry

### Why workload telemetry is required

Control plane telemetry answers *what was configured*.  
Workload telemetry answers *what actually ran*.

Most Azure incidents involve:
- Command execution
- Process creation
- Credential access
- Persistence mechanisms

None of these are visible in control plane logs.

---

### Required telemetry

For infrastructure workloads:

- Windows:
  - Security Event Log
  - System Event Log (where relevant)
- Linux:
  - Authentication logs
  - Syslog

For platform workloads:

- AKS:
  - Control plane logs
  - Container stdout/stderr where feasible
- App Services and Functions:
  - Application logs
  - Platform diagnostics

Workload logs must be:
- Time-synchronized
- Identity-correlated
- Retained long enough to support delayed detection

---

### The EDR-only fallacy

Endpoint Detection and Response tools provide:
- Detection
- Triage signals

They do **not** replace:
- Baseline execution logging
- Historical investigation data
- Negative confirmation (proving something did *not* occur)

EDR alerts without workload logs force blind trust in tooling during incidents.

---

### Common workload telemetry failures

- No baseline OS logging
- Logs collected but not forwarded
- Over-reliance on alerts
- Inconsistent coverage across environments

---

## Data Plane Telemetry

### Why data access telemetry is critical

In mature cloud incidents:
- Infrastructure compromise is a means
- Data access is the objective

Data access often:
- Uses valid credentials
- Appears operationally normal
- Produces no malware or exploit indicators

---

### Required telemetry

The SOC must receive:

- Key Vault access and audit logs
- Storage account object-level access logs
- SQL Database and Managed Instance auditing logs
- Cosmos DB diagnostics, where applicable

Logs must capture:
- identity
- object or data asset accessed
- operation type
- result
- source location

---

### Common data plane telemetry failures

- Control plane logs without access logs
- Short retention periods
- Partial object-level visibility

These failures make it impossible to:
- Confirm data theft
- Scope impacted assets
- Support regulatory or legal response

---

## Security Control Plane Telemetry

### Role of security alerts

Security controls provide:
- Prioritization
- Early warning
- Automated response signals

They do not replace raw telemetry.

---

### Required telemetry

- Defender for Endpoint alerts
- Defender for Cloud alerts
- Identity Protection risk events
- Policy enforcement alerts

Alerts must be ingested alongside supporting telemetry to allow:
- Validation
- False positive analysis
- Post-incident learning

---

### Common security control failures

- Alerts ingested without evidence
- Vendor severity trusted without context
- No ability to reconstruct alert logic post-incident

---

## Cross-Plane Correlation Requirements

Effective investigation requires correlation across planes:

- Identity → Control Plane → Workload
- Identity → Data Plane
- Control Plane → Network → Data Exfiltration

Telemetry that cannot be correlated across planes significantly reduces SOC effectiveness.

---

## Final Position

If any of the telemetry described in this document is missing, the organization should assume:

- Increased mean-time-to-detect
- Reduced investigation confidence
- Higher containment risk
- Longer recovery timelines

These outcomes are not caused by tooling limitations.

They are the result of **telemetry design decisions**.

---
