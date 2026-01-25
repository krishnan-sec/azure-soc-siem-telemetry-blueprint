# Validation Tests and Acceptance Criteria

This document defines **how required telemetry is validated** and **what constitutes acceptable coverage** for an external SOC and SIEM operating in Microsoft Azure.

Telemetry that is merely *enabled* is not sufficient.
Telemetry must be **provably present, timely, complete, and usable**.

Validation is not a one-time activity.
It is a recurring security control.

---

## Why Validation Is Mandatory

Most Azure logging failures are not caused by:
- platform limitations
- missing features
- lack of intent

They are caused by:
- configuration drift
- partial onboarding
- silent pipeline failures
- schema or parsing breakage
- assumptions that “it’s still working”

These failures are rarely detected during normal operations.
They surface **during incidents**, when recovery time is already constrained.

Validation exists to ensure that:
- telemetry gaps are detected early
- responsibility is clear
- investigations do not begin with uncertainty

---

## Validation Principles

All validation activities in this repository adhere to the following principles:

1. **Evidence over configuration**  
   Validation is based on observable events in the SIEM, not Azure configuration state.

2. **Plane-aware testing**  
   Each plane of control must be validated independently.

3. **Positive and negative confirmation**  
   Tests must prove both that expected events appear and that prohibited actions are blocked or logged.

4. **Repeatability**  
   Validation steps must be deterministic and repeatable.

5. **Clear ownership**  
   Every failed validation must have a defined owner and remediation path.

---

## When Validation Must Occur

Validation is mandatory at the following points:

- Initial SOC onboarding
- Onboarding of a new subscription or landing zone
- Introduction of new Azure services
- Changes to telemetry pipelines or SIEM parsers
- Periodic reassessment (recommended quarterly)
- Post-incident review

Skipping validation at any of these points should be treated as **explicit risk acceptance**.

---

## Validation Scope

Validation applies to:

- Telemetry presence
- Telemetry timeliness
- Telemetry completeness
- Telemetry usability by the SOC

Validation does **not** attempt to:
- prove detection efficacy
- test SOC response quality
- validate alert tuning

Those activities are downstream of telemetry validation.

---

## Plane-by-Plane Validation Tests

### 1. Identity Plane Validation

#### Objective
Confirm that all identity activity required for attribution and investigation is present in the SIEM.

#### Test Scenarios

- Perform an interactive user sign-in
- Trigger a non-interactive sign-in (e.g., service principal or managed identity)
- Activate a privileged role using PIM (if applicable)
- Trigger a Conditional Access evaluation

#### Expected Evidence

The SIEM must show:
- distinct interactive and non-interactive events
- stable identity identifiers (not just display names)
- authentication method and Conditional Access outcome
- correct timestamps with minimal delay

#### Acceptance Criteria

- Events appear within defined latency SLA
- Identity fields are populated and normalized
- Interactive and non-interactive activity is distinguishable

Failure of any condition constitutes **identity visibility failure**.

---

### 2. Control Plane Validation

#### Objective
Confirm visibility into resource management intent and configuration changes.

#### Test Scenarios

- Create and delete a test resource
- Modify a resource configuration
- Change an RBAC assignment
- Enable or disable a diagnostic setting

#### Expected Evidence

The SIEM must show:
- Azure Activity Log entries for each action
- Correlated resource identifiers
- Actor identity and operation result
- Diagnostic logs for services where applicable

#### Acceptance Criteria

- All actions are visible with attribution
- Resource identifiers are intact
- Control-plane and diagnostic events are correlated

Control-plane visibility without diagnostic follow-through is **insufficient**.

---

### 3. Network Plane Validation

#### Objective
Confirm visibility into allowed and denied traffic flows.

#### Test Scenarios

- Generate allowed ingress traffic
- Generate denied traffic (where safe)
- Generate east–west traffic between workloads
- Generate outbound traffic to an external endpoint

#### Expected Evidence

The SIEM must show:
- flow or firewall events for each scenario
- source and destination context
- allow/deny outcome
- timestamps aligned with test execution

#### Acceptance Criteria

- Network direction is correctly inferred
- East–west traffic is visible for in-scope workloads
- No unexplained gaps during test window

Lack of lateral visibility should be explicitly documented as risk.

---

### 4. Workload Plane Validation

#### Objective
Confirm execution-level visibility for workloads.

#### Test Scenarios

- Execute a benign command on a VM
- Trigger a process creation event
- Generate authentication events on Linux/Windows
- Generate application-level logs (where applicable)

#### Expected Evidence

The SIEM must show:
- host or container identity
- process or execution context
- correlation to initiating identity (where possible)
- accurate timestamps

#### Acceptance Criteria

- Execution context is visible
- Events are queryable by SOC analysts
- Gaps are explicitly identified

Reliance on alerts alone is **not acceptable** validation.

---

### 5. Data Plane Validation

#### Objective
Confirm visibility into sensitive data access.

#### Test Scenarios

- Read a Key Vault secret
- Access a storage object
- Perform a database query (where applicable)

#### Expected Evidence

The SIEM must show:
- identity accessing the data
- data asset or object identifier
- operation type and result
- source context (where available)

#### Acceptance Criteria

- Access is attributable
- Object-level context is present
- Logs are retained and queryable

Inability to prove data access is a **critical gap**.

---

### 6. Security Control Plane Validation

#### Objective
Confirm that security alerts and their supporting evidence are visible.

#### Test Scenarios

- Trigger a benign test alert (e.g., test malware, identity risk)
- Generate a known Defender alert where safe

#### Expected Evidence

The SIEM must show:
- alert metadata
- severity and classification
- linkage to raw telemetry
- timestamps aligned with event occurrence

#### Acceptance Criteria

- Alerts are ingested reliably
- Supporting telemetry is available
- Alerts are not isolated artifacts

---

## Telemetry Timeliness and Latency

Validation must include measurement of:

- source event time
- ingestion time
- availability time in the SIEM

### Acceptance Guidance

- Identity and control plane: near-real-time
- Network and workload: acceptable bounded delay
- Backlog or burst delays must be visible and monitored

Unbounded or unexplained delays are unacceptable.

---

## Failure Classification

All validation failures must be classified as one of:

- **Source failure** (logs not emitted)
- **Configuration failure** (diagnostics misconfigured)
- **Transport failure** (pipeline issue)
- **Parsing/normalization failure**
- **Access failure** (SOC cannot query)

Each category must have a defined owner and remediation path.

---

## Documentation and Evidence

Validation results must be:

- documented
- timestamped
- retained for audit and post-incident review

Screenshots alone are insufficient.
Queryable evidence is required.

---

## Ownership and Accountability

Validation ownership must be explicitly defined:

- Platform team: source configuration
- Security engineering: pipeline and normalization
- SOC: queryability and investigative usability

Unowned validation failures represent unmanaged risk.

---

## Exit Criteria

Telemetry validation is complete when:

- All required planes pass validation
- Failures are documented with accepted risk or remediation
- SOC confirms investigative usability
- Validation cadence is scheduled

If these conditions are not met, telemetry coverage should be considered **not operationally ready**.

---

## Final Position

Validation is the difference between:
- *believing* telemetry exists
- *knowing* telemetry exists

In cloud environments, belief is insufficient.

This document exists to ensure that telemetry failures are discovered deliberately, not during an incident.

---
