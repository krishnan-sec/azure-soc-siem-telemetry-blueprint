# Azure Planes and Control Points

This document defines the **primary planes of control in Microsoft Azure** from a security monitoring and incident response perspective.

Rather than cataloging services, it frames Azure as a set of **distinct control planes**, each responsible for different security outcomes and each requiring **specific telemetry** to support effective SOC operations.

Understanding these planes and their boundaries is critical.  
Most security monitoring failures in Azure occur **between** planes, not within them.

---

## Why Planes Matter

Azure is often described as a collection of services.  
From a security perspective, this framing is misleading.

Incidents do not unfold “within a service.”  
They unfold across **planes of control**:

- Identity enables access
- Control plane expresses intent
- Network enables movement
- Workloads execute actions
- Data is accessed or exfiltrated
- Security controls attempt to detect or block activity

If any plane lacks visibility, investigations stall or rely on assumptions.

---

## Plane Model Used in This Repository

This repository uses the following plane model:

1. Identity Plane
2. Control Plane
3. Network Plane
4. Workload Plane
5. Data Plane
6. Security Control Plane

Each plane:
- Answers a different class of investigative question
- Produces different telemetry
- Fails in different ways when visibility is incomplete

---

## 1. Identity Plane

### Definition

The identity plane governs **who or what is allowed to act** in Azure.

It includes:
- Human users
- Service principals
- Managed identities
- Tokens and claims
- Conditional Access decisions
- Privilege elevation mechanisms

In Azure, identity is not a supporting system, it is **the primary security boundary**.

---

### What the Identity Plane Answers

During an incident, identity telemetry must answer:

- Which identity authenticated?
- Was access interactive or programmatic?
- What authentication method was used?
- What conditions were evaluated and enforced?
- Was privilege elevated before or after the action?

Without identity context, downstream telemetry loses attribution value.

---

### Control Points

Key identity control points include:

- Authentication (sign-ins, token issuance)
- Authorization (roles, scopes)
- Privileged Identity Management (PIM)
- Conditional Access policies
- Consent and OAuth grants

Each control point represents an opportunity for both defense and abuse.

---

### Common Failure Modes

- Interactive sign-ins collected, non-interactive omitted
- PIM enabled, but activations not monitored
- Conditional Access outcomes ignored
- Identity treated as “audit” rather than detection data

These failures create environments where:
- Infrastructure changes appear anonymous
- Data access cannot be attributed
- Automation abuse is invisible

---

## 2. Control Plane 

### Definition

The control plane represents **declared intent**.

It governs:
- Resource creation
- Resource modification
- Configuration changes
- Access control changes

It is expressed primarily through Azure Resource Manager (ARM).

---

### What the Control Plane Answers

Control plane telemetry must answer:

- What action was requested?
- Which identity requested it?
- Against which resource?
- Was the request allowed or denied?

This plane is critical for understanding **how the environment was shaped**.

---

### Control Points

- Azure Activity Log
- Resource-level diagnostic logs
- Azure Policy evaluations
- Role assignment changes

---

### Limitations of Control Plane Telemetry

The control plane does **not** tell you:

- Whether a resource was used
- How it was used
- What occurred inside the resource

This distinction is critical.

---

### Common Failure Modes

- Treating Activity Logs as complete visibility
- Enabling diagnostics selectively or inconsistently
- No correlation between control plane and runtime events

These failures lead to investigations that stop at:
> “Something changed, but we don’t know what happened next.”

---

## 3. Network Plane

### Definition

The network plane governs **how systems communicate**.

It includes:
- Ingress and egress paths
- East-west traffic
- Security enforcement points

While identity initiates most modern attacks, networks still determine **reach and impact**.

---

### What the Network Plane Answers

Network telemetry must answer:

- What communicated with what?
- Was traffic allowed or denied?
- Was movement lateral or external?
- Did data leave the environment?

---

### Control Points

- Network Security Groups (NSGs)
- Azure Firewall
- Web Application Firewall (WAF)
- Load balancers and gateways
- VPN and private connectivity

---

### Common Failure Modes

- Logging only internet-facing traffic
- No east-west visibility
- Assuming EDR replaces flow data

Without network telemetry:
- Scope cannot be determined
- Exfiltration cannot be confirmed or disproven
- Containment decisions are delayed

---

## 4. Workload Plane

### Definition

The workload plane represents **execution**.

It includes:
- Virtual machines
- Containers and Kubernetes
- Platform runtimes (App Services, Functions)

This is where intent becomes action.

---

### What the Workload Plane Answers

Workload telemetry must answer:

- What executed?
- Under which identity?
- On which host or container?
- With what privileges?

This plane is essential for confirming compromise.

---

### Control Points

- Guest OS logging
- Container runtime logs
- Application logs
- EDR and host-based controls

---

### Common Failure Modes

- Relying solely on EDR alerts
- No baseline OS logging
- Treating workload logs as “optional”

These failures result in:
- Inability to confirm execution
- Speculative post-incident analysis
- Unverifiable detections

---

## 5. Data Plane

### Definition

The data plane governs **access to information assets**.

It includes:
- Secrets
- Files and objects
- Databases
- Messaging systems

In most incidents, data not infrastructure, is the objective.

---

### What the Data Plane Answers

Data telemetry must answer:

- What data was accessed?
- By which identity?
- From where?
- Was access expected?

---

### Control Points

- Key Vault access logs
- Storage account access logs
- Database auditing
- Service-specific diagnostics

---

### Common Failure Modes

- Control plane logs without access logs
- No object-level visibility
- Short retention periods

These failures make data loss:
- Difficult to detect
- Impossible to scope
- Hard to prove or disprove

---

## 6. Security Control Plane

### Definition

The security control plane represents **protective and detective controls**.

It includes:
- Endpoint protection
- Cloud security posture
- Identity risk evaluation
- Threat detections

---

### What the Security Control Plane Answers

Security telemetry must answer:

- Which controls triggered?
- Why did they trigger?
- What evidence supports the alert?
- What action was taken?

---

### Control Points

- Defender for Endpoint
- Defender for Cloud
- Identity Protection
- Policy enforcement systems

---

### Common Failure Modes

- Alerts without raw telemetry
- Blind trust in vendor severity
- No ability to validate or refute detections

---

## Inter-Plane Dependencies

Incidents rarely stay within one plane.

Common attack paths:
- Identity → Control Plane → Workload → Data
- Identity → Data (direct access)
- Control Plane → Network → Exfiltration

Breakdowns most often occur at **plane boundaries**, where telemetry is incomplete or uncorrelated.

---

## Final Assertion

Effective Azure security monitoring requires **deliberate, plane-aware telemetry design**.

Failure to design for plane coverage results in:
- Partial investigations
- Delayed response
- Increased operational risk

These outcomes are not caused by missing tools, but by **missing architectural intent**.

---
