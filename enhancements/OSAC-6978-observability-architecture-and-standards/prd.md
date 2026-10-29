# Observability Architecture and Standards

| Field     | Value                                                              |
|-----------|--------------------------------------------------------------------|
| Author(s) | Amit Oren                                                          |
| Jira      | https://redhat.atlassian.net/browse/OSAC-6978                     |
| Date      | 2026-10-29                                                         |

## Problem Statement

OSAC's component ecosystem — KubeVirt, HyperShift, OpenShift AI (MaaS), BMC controllers, and OSAC's own services — emits metrics and logs today, but without shared naming conventions, a common resource identity schema, or tenant isolation contracts. Shelf products (KubeVirt, HyperShift, MaaS, BMC) emit signals in their own formats and will not change; no strategy exists for adapting those signals to a common schema at the collection layer. Platform operators cannot correlate signals across components to diagnose incidents, and tenant data cannot be reliably isolated in multi-tenant deployments. Without a shared foundation and a defined adaptation strategy, each downstream observability capability — metrics pipelines, log aggregation, alerting, dashboards — would be implemented inconsistently, compounding the problem at every layer. [Clarify: R2.Q4, R5.Q1]

## In Scope

- Published telemetry conventions document specifying signal types, metric-naming patterns, units, and cardinality guidelines for metrics and logs, with a defined extensibility path for distributed traces in a future version. [Clarify: R1.Q2, D2]
- Resource identity schema defining how all OSAC-managed resources (clusters, VMs, bare metal, GPUs) are identified in telemetry data.
- Label and attribute taxonomy specifying standard labels for tenant, owner, resource type, region, and component. [Clarify: R2.Q3, D6]
- GPU telemetry conventions: vendor-agnostic metric names and labels for GPU utilization, temperature, power draw, and error counts, with NVIDIA as the primary reference implementation. NVIDIA MIG (Multi-Instance GPU) partitioning is a known open question — to be determined in a follow-up. [Clarify: R3.Q2]
- Protocol standards: Prometheus exposition format and OTLP as the adopted collection protocols for metrics and logs. [Clarify: R3.Q1, D8]
- Shelf-product signal adaptation guidance: how signals from KubeVirt, HyperShift, OpenShift AI (MaaS), and BMC controllers are normalized to the OSAC standard through collection-layer mechanisms (e.g., metric relabeling, renaming, transformation). [Clarify: R5.Q1, D11]
- Tenancy model specifying how tenant data isolation is enforced at the collection layer, not only at query time. [Clarify: R2.Q3, D6]
- Configurable retention classes (hot, warm, cold) with per-environment duration parameters. [Clarify: R2.Q2, D5]
- Clock synchronization requirements for all OSAC components to ensure accurate, auditable timestamps. [Clarify: R2.Q1]
- Query interface analysis: evaluation of extending the OSAC fulfillment API versus deploying a dedicated observability query API server, with a recommendation and documented trade-offs. [Clarify: R4.Q2, D10]
- Topology analysis: combined versus separate observability and auditing service models, with trade-offs and guidance for the Design phase. [Clarify: R1.Q3, D3]
- Scale target: tens of tenants and hundreds of OSAC-managed resources. [Clarify: R4.Q1, D9]
- All deliverables published as an Enhancement Proposal PRD followed by a Design document, using the OSAC Enhancement Proposal templates. [Clarify: R1.Q1, D1]

## Out of Scope

- Implementation of collection agents, storage backends, or query servers — delivered by subsequent Features under OSAC-3415.
- Alerting rules, dashboards, and visualization — OSAC-6984.
- Audit logging implementation — OSAC-63. This architecture defines the telemetry conventions and topology constraints that the audit service must conform to; it does not implement the audit service.
- Geographic data residency controls; data residency in this Feature means tenant-level data isolation only. [Clarify: R2.Q3, D6]
- Distributed traces as a first-class signal type — deferred to a future version; the conventions define an extensibility path. [Clarify: R1.Q2, D2]
- Compliance certification or regulatory audit.

## User Stories

### Cloud Provider Admin

- As a Cloud Provider Admin, I want a published telemetry conventions document specifying standard label names (tenant, owner, resource type, region, component) and metric-naming patterns so that I can verify all OSAC components emit consistent, queryable signals.
- As a Cloud Provider Admin, I want the tenancy model to enforce data isolation at the collection layer so that one tenant's metrics and logs cannot be accessed through any query interface by another tenant.
- As a Cloud Provider Admin, I want the retention classes to expose configurable duration parameters per tier (hot, warm, cold) so that I can set per-environment retention values to meet applicable regulatory requirements.

### Cloud Infrastructure Admin

- As a Cloud Infrastructure Admin, I want the GPU telemetry conventions to define standard metric names and labels for GPU utilization, temperature, power draw, and error counts — using a vendor-agnostic schema with NVIDIA as the primary reference — so that I can collect and correlate GPU health signals consistently across all managed clusters.
- As a Cloud Infrastructure Admin, I want the architecture to adopt Prometheus exposition format and OTLP as the collection protocols so that I can provision the observability collection pipeline to a well-specified interface.
- As a Cloud Infrastructure Admin, I want clock synchronization requirements defined for all OSAC components so that I can configure time sources and produce timestamps that are accurate and auditable across the fleet.

### Tenant Admin

- As a Tenant Admin, I want the observability architecture to define tenant-scoped label conventions so that my organization's metrics and logs are automatically segregated from other tenants' data and can be queried in isolation.
- As a Tenant Admin, I want the retention classes to support configurable durations so that I can request a retention configuration that meets my organization's data lifecycle requirements.

## Dependencies

- **OSAC-63 (Activity and Audit Log API):** The topology analysis (combined vs. separate service model) in this Feature's Design must align with OSAC-63's current design and assess whether a common storage backend is achievable. OSAC-63 design review is a required input before the OSAC-6978 Design document can be finalized.
- **KubeVirt, HyperShift, OpenShift AI (MaaS), BMC:** These components emit signals in their own formats. They are dependencies in that the adaptation strategy defined in this Feature must cover their signal schemas; the components themselves are not changed. [Clarify: D11]
- **Enhancement Proposals repository (`enhancement-proposals/`):** The PRD and Design deliverables for this Feature are published there following the Enhancement Proposal process.
- **Downstream observability Features (OSAC-6980, OSAC-6982, OSAC-6984, OSAC-6986, OSAC-6988, OSAC-6990, OSAC-6992):** All are blocked by this Feature. Their Design phases should not begin until this Feature's standards document is approved.

---

## Provenance

Committed: commit @ prd 0.11.5 - 7b2b8de, workspace prd/OSAC-6978 @ bb9e8f6

> Authoring phases not recorded this session (commit-time snapshot only).

<!-- ai-workflow-provenance:{"schema_version":1,"provenance_kind":"commit_only","workflow":"prd","workflow_version":"0.11.5","ai_workflows":"7b2b8de","source_repo":"bb9e8f6","source_repo_branch":"prd/OSAC-6978","commits_behind_main":0,"commits_ahead_main":0,"main_ref":"main","phases":["commit"],"authoring_modes":["skill"],"context_changed":false,"origin_untracked":false} -->
