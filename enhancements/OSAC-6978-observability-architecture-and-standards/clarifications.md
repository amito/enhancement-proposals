# Clarification Log — OSAC-6978

## Status

- Rounds completed: 4
- Open gaps: 2 (non-blocking — NVIDIA MIG scope, approval process)
- Exit criteria met: Yes

---

## Round 1 — Scope and Deliverable Format

### R1.Q1: Architecture document format

The Definition of Done says "Architecture document published to OSAC docs" but doesn't specify the format. Does OSAC have an existing Enhancement Proposal, ADR, or HLD template that this document should follow? Or is establishing the document format itself part of this feature's scope?

#### Answer

The document format should be a PRD following the OSAC Enhancement Proposals template. Similarly, the Design document should follow the OSAC Design document template.

#### Impact

The deliverable for this Feature is an Enhancement Proposal PRD (not a freeform architecture doc). A Design document is a separate, subsequent deliverable. Both must follow established OSAC templates. The `/draft` phase should produce a PRD structured as an Enhancement Proposal.

#### Decision (D1)

Architecture and standards deliverables for OSAC-6978 follow the OSAC Enhancement Proposal PRD template (for the PRD gate task) and the OSAC Design document template (for the Design gate task). These are the authoritative document formats.

---

### R1.Q2: Distributed traces in scope

The DoD lists "supported protocols specified: metrics export, log shipping, trace collection" — traces are mentioned alongside metrics and logs. Are distributed traces in scope as a first-class signal type for 0.5, or deferred?

#### Answer

Traces are deferred to a later version. However, the architecture must "leave room" to be extended to support traces in the future.

#### Impact

The architecture document must define conventions for metrics and logs as first-class signal types. For traces, it must include an extensibility section noting how the conventions (naming, identity schema, label taxonomy) would accommodate traces when they are adopted. No protocol choice for trace collection is required for 0.5.

#### Decision (D2)

Distributed traces are out of scope for this version. The architecture must be designed with extensibility for traces — the conventions document must include a section on future trace support and identify which aspects of the current conventions are trace-compatible.

---

### R1.Q3: Combined vs. separate observability/auditing topology

The Outcome defers whether observability and auditing are one service or two. The DoD says the architecture must "accommodate both topologies." Does this mean the document describes both options, or must it be agnostic?

#### Answer

Both topologies should be considered and described. A shared backend is likely viable. During the Design phase, the team should look at ongoing auditing work (OSAC-63) to see whether a common backend can be established.

#### Impact

The architecture document must include a topology analysis section covering both combined and separate service models, with trade-offs. It should lean toward shared-backend compatibility as the likely direction. The Design phase (and its alignment with OSAC-63) is where the topology decision is locked.

#### Decision (D3)

The architecture document describes both combined and separate observability/auditing topologies with trade-offs. A shared storage backend is identified as the likely path. The Design phase for OSAC-6978 must explicitly review OSAC-63 progress and assess whether a common backend is achievable.

---

### R1.Q4: Architecture document approval process

Who approves the architecture document before it becomes the reference standard for all subsequent observability Features?

#### Answer

TBD.

#### Impact

The PRD should note that an approval process needs to be defined. The Definition of Done item "all subsequent observability Features reference this standard" cannot be completed until the standard is approved. This is a process dependency, not a technical one.

---

## Round 2 — Compliance, Personas, and Existing State

### R2.Q1: Auditor and compliance personas

The parent Outcome mentions Auditor, Data Custodian, and Data Owner Telemetry personas. Should the architecture address their access requirements?

#### Answer

Those are personas for the auditing features and should not be addressed as part of the observability feature set.

#### Impact

User stories and RBAC considerations for Auditor, Data Custodian, and Data Owner Telemetry are out of scope for OSAC-6978. The four standard OSAC personas (Cloud Provider Admin, Cloud Infrastructure Admin, Tenant Admin, Tenant User) are the target audience for this feature.

#### Decision (D4)

Auditor, Data Custodian, and Data Owner Telemetry personas are scoped to auditing features (OSAC-63 and related). They are out of scope for all eight observability Features under OSAC-3415.

---

### R2.Q2: Compliance retention minimums

Must NIST 800-171 and HIPAA retention minimums be baked into the retention class definitions, or is the architecture only responsible for making retention configurable?

#### Answer

Retention should be configurable for observability storage. Compliance minimums may be set as defaults in the future, but are not required to be hard-coded now.

#### Impact

The retention class definitions must specify configurable durations per tier (hot, warm, cold). The PRD does not need to mandate specific retention durations for NIST 800-171 or HIPAA compliance — only that the mechanism to configure them is present and that compliance values can be expressed.

#### Decision (D5)

Retention durations are configurable, not hard-coded. The architecture defines retention classes (hot/warm/cold) with configurable duration parameters. Compliance-mandated minimums may be expressed as recommended default values in documentation; they are not enforced constraints in the architecture itself.

---

### R2.Q3: Data residency scope

Is data residency about geographic placement, tenant isolation, or both?

#### Answer

Tenant isolation only.

#### Impact

"Data residency controls" in OSAC-6978 means ensuring one tenant's telemetry data is isolated from another's — not geographic placement. Geographic placement is out of scope. The tenancy model section of the architecture document focuses on isolation boundaries, not storage location.

#### Decision (D6)

Data residency in this feature means tenant-level data isolation. Geographic placement of telemetry data is out of scope for OSAC-6978.

---

### R2.Q4: Existing component telemetry conventions

Do OSAC components currently emit metrics/logs with informal conventions that the architecture should standardize around?

#### Answer

KubeVirt, HyperShift, MaaS (OpenShift AI), and BMC all emit metrics, logs, and health signals. OSAC code also generates metrics and logs. The architecture can start fresh — existing conventions are not a constraint.

#### Impact

The architecture document defines new conventions. Existing components (KubeVirt, HyperShift, MaaS/OpenShift AI, BMC, OSAC services) are dependencies — they must conform to the new conventions. The PRD should note migration or adoption of these conventions by existing components as a follow-on concern.

#### Decision (D7)

The architecture defines new, fresh conventions. Existing component telemetry signals (KubeVirt, HyperShift, MaaS/OpenShift AI, BMC, OSAC services) are not a constraint on the standard — they are identified as components that must eventually adopt it.

---

## Round 3 — Protocols, Technology Direction, and GPU

### R3.Q1: Protocol technology choices

Does the architecture standardize on specific protocols, or defer protocol choice to Design?

#### Answer

Prometheus (exposition format) and OTLP (OpenTelemetry Protocol) are the widely adopted standard and are the target protocols.

#### Impact

The supported protocols section of the architecture document specifies Prometheus exposition format for metrics scraping and OTLP for push-based collection. Log shipping and health signal protocols must also be specified (e.g., Loki, OpenSearch, or structured syslog — this is a remaining design input, but OTLP logs are a reasonable default given the OTLP adoption decision).

#### Decision (D8)

Prometheus exposition format and OTLP are the adopted protocols for metrics. The architecture extends this choice to the full signal pipeline. Log and health signal protocols should default to OTLP where applicable.

---

### R3.Q2: GPU vendor scope and MIG configurations

Should GPU telemetry conventions be vendor-agnostic? Are NVIDIA MIG configurations in scope?

#### Answer

The schema should be vendor-agnostic with NVIDIA prioritized higher. NVIDIA MIG configurations are an open question — left for future exploration.

#### Impact

The GPU telemetry conventions use a vendor-agnostic schema (e.g., `gpu.vendor`, `gpu.model` labels) with NVIDIA-specific guidance provided as the primary reference implementation. The PRD should note NVIDIA MIG (Multi-Instance GPU) as a known future consideration requiring a separate design decision.

---

### R3.Q3: Observability query interface and OSAC API surfaces

Should the architecture integrate with OSAC's existing fulfillment API, or define a separate observability query API?

#### Answer

OSAC has an existing fulfillment API that serves the REST API, and through that the CLI and UI. However, a dedicated observability-only API server may be more aligned with standards. This needs further exploration.

#### Impact

The query interface question is not yet resolved. Based on the Round 4 decision (D10), the architecture document will take a position on this and document trade-offs. This is now an in-scope deliverable for OSAC-6978.

---

## Round 4 — Scale Expectations and Query Interface

### R4.Q1: Scale the architecture must accommodate

What order of magnitude should the architecture's conventions and cardinality guidelines target?

#### Answer

Tens of tenants and hundreds of resources (pilot scale).

#### Impact

Telemetry cardinality guidelines should be designed for pilot scale: tens of tenants × hundreds of resources × signal types. This constrains label cardinality guidance (e.g., high-cardinality labels like resource ID are acceptable at this scale). The architecture should note that higher-scale deployment may require additional cardinality controls.

#### Decision (D9)

Target scale for this version: tens of tenants, hundreds of resources. Cardinality guidelines and retention class sizing reflect pilot-scale deployment. Scale-out considerations are noted as future work.

---

### R4.Q2: Query interface — architecture takes a position

Should the architecture document take a position on the query interface (fulfillment API extension vs. dedicated observability API server), or remain agnostic?

#### Answer

Yes, take a position and document trade-offs.

#### Impact

The architecture document must include a query interface section that analyzes both options (extending the fulfillment API vs. a dedicated observability API server), presents trade-offs, and makes a recommendation. This is an in-scope deliverable for the architecture document.

#### Decision (D10)

The architecture document takes a position on the observability query interface and documents trade-offs between extending the fulfillment API and deploying a dedicated observability API server.

---

## Round 5 — Shelf product signal adaptation

### R5.Q1: Existing component signal adoption

The first draft assumed KubeVirt, HyperShift, OpenShift AI (MaaS), and BMC controllers would adopt OSAC's new telemetry conventions directly.

#### Answer

Shelf products (KubeVirt, HyperShift, MaaS, BMC) will not have their own signal conventions changed. Instead, OSAC will adapt their signals to the standard — through metric renaming, relabeling, or other collection-layer normalization methods.

#### Impact

The conventions document must include guidance on how shelf-product signals are adapted to the OSAC standard at the collection layer. The assumption that these components "will adopt" the conventions is incorrect — they will continue emitting signals in their own formats; the adaptation responsibility belongs to the OSAC collection pipeline. This distinction affects In Scope (adaptation strategy is a deliverable), the Problem Statement (the gap includes no adaptation strategy, not just no conventions), and the first Assumption in the PRD.

#### Decision (D11)

Shelf products (KubeVirt, HyperShift, OpenShift AI/MaaS, BMC) will not change their signal conventions. The OSAC observability architecture must specify how their signals are adapted — through collection-layer mechanisms such as metric relabeling, renaming, or transformation — to conform to the OSAC telemetry standard.

---

## Remaining Gaps

1. **NVIDIA MIG support** — Whether NVIDIA MIG (Multi-Instance GPU) configurations are in scope for the GPU telemetry conventions. Marked as a future consideration; the PRD should note it explicitly.

2. **Architecture approval process** (R1.Q4) — Who approves the architecture document before it becomes the reference standard. TBD; the PRD should flag this as a process dependency.
