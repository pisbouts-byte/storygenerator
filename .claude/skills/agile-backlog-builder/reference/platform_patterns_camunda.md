# Platform Patterns — Camunda (Platform 8 / Zeebe)

Loaded when intake identifies the target platform as Camunda. Confirm the **specific version**
(8.x — the engine is Zeebe; Camunda 7's embedded-engine model is architecturally different and this
file's conventions don't transfer) and **hosting** (Camunda SaaS vs. self-managed on Kubernetes) —
self-managed hosting adds real foundational scope (Helm chart deployment, cluster operations) that
SaaS hosting removes; don't assume one without confirming.

## Foundational / enabler stories for a **new build** on Camunda 8

Don't generate feature stories before these exist (or are explicitly confirmed as already in place):
- Cluster provisioning: a Camunda SaaS cluster, or a self-managed Kubernetes/Helm deployment
  (Zeebe, Operate, Tasklist, Identity, and Connectors runtime components)
- Identity/auth setup — Camunda 8 uses an Identity component backed by OIDC (Keycloak for
  self-managed, or an external IdP); confirm SSO integration scope
- CI/CD for process artifacts — BPMN/DMN deployment pipeline (via `zbctl`, the Zeebe REST/gRPC
  deploy API, or Camunda's GitHub Actions), separate from application code CI/CD
- Job worker scaffolding — the base microservice pattern/SDK external workers will be built on
  (Camunda's Spring Zeebe client, Java/Node/Python client, or Connector-based workers)
- Multi-tenancy configuration, if applicable — Camunda 8 supports tenant-scoped process definitions
- Monitoring/alerting integration with Operate (operational visibility) and, if licensed, Optimize
  (process analytics)

## Design-detail conventions for Story rows

When writing the **Design Details** field for a Camunda story, name the actual BPMN/DMN construct:
- Process/flow work → reference the **process definition** and specific **element types** (User
  Task, Service Task, Business Rule Task, Script Task, Send/Receive Task, Timer/Message/Error/
  Escalation events, exclusive/parallel/inclusive/event-based gateways, embedded sub-process vs.
  call activity, multi-instance markers) — not generic "add a process step"
- Business rules → reference the **DMN decision table** (or decision requirements diagram if
  multiple tables feed one decision) and its hit policy, not generic "apply business logic"
- Integrations → reference whether the call is a **Job Worker** (external task pattern — a
  custom microservice polling Zeebe for jobs) or an out-of-box **Connector** (Camunda's prebuilt
  REST/etc. connector task), and whether it's new or reuses an existing worker/connector
- UI/human tasks → reference **Camunda Forms** (form-js, embedded or linked) for simple task UIs
  vs. a custom front-end calling the Tasklist/Zeebe REST APIs for anything beyond a basic form —
  state which, since the design and sizing differ substantially
- Reporting → reference **Optimize** dashboards/reports (Enterprise feature) or custom reporting
  built off Operate/exported process data — don't assume Optimize is licensed without confirming

## Case-control constructs (map to the checklist in SKILL.md §2a)

Camunda 8 is a process orchestration engine, not a case-management platform — most of these
controls have no single built-in button and must be explicitly modeled in the BPMN diagram or built
as a custom capability. Don't under-size these by assuming "the engine handles it":
- **Withdraw** → Cancel Process Instance (a Zeebe command), but exposing this safely to business
  users needs an explicit BPMN element — typically an interrupting boundary event correlated to a
  "Cancel" message, not a bare ops-console cancellation
- **Hold** → no native hold button; model an explicit wait state (a receive task or boundary event
  catching an external trigger) — specify what actually resumes it
- **Resume** → message correlation into the waiting receive task/boundary event from §Hold
- **Skip** → a conditional sequence flow (gateway) bypassing a task under a business condition fed
  by a process variable — don't rely on Operate's "Modify Instance" capability for a business-facing
  skip feature; that's an operations/support tool, not meant for end users
- **Return / Go-back** → requires an explicit rework loop modeled in the BPMN diagram (gateway
  routing back to an earlier task); Camunda has no native "go back" action outside the modeled flow
- **Reassign** → Tasklist supports task reassignment via its API/UI (claim/unclaim, candidate group
  reassignment) — specify whether reassignment is user-initiated or rule-driven
- **Merge** → no native process-instance merge; build via message correlation on a shared business
  key across instances — flag as custom, at least Medium complexity
- **Duplicate detection** → custom: a DMN table or service task querying for an existing instance by
  business key before starting a new one
- **Dependency / relationship gating** → message correlation or call activities linking parent/child
  process instances, gating progress on a related instance's state via message events
- **Exception processing** → Zeebe automatically raises **incidents** on job/task failures (visible
  in Operate for ops-level retry); business-level exception handling needs explicit BPMN error
  boundary events or escalation events — specify which pattern applies rather than leaving
  "handles exceptions" generic

## Document/correspondence generation is a Feature, not a story

Camunda has no native document-generation engine — this is always integration work, typically a
Service Task/Job Worker calling an external templating service or library (e.g., a document
service, DOCX/PDF template engine, or a connector to a document platform). Treat multi-variant
correspondence as its own **Feature**, decomposed into: template inventory (by product/segment/
standard type), the job worker/service handling template population, the BPMN trigger point(s),
review/approval routing (often its own sub-process with a user task), and the delivery-channel
integration (email/portal/print vendor) as a separate integration story. Multiple templates or
review paths is a variant-dimension trigger, not a reason for one large story.

## Integration sizing signals specific to Camunda

Drivers that push a story above a routine 5–8: a new job worker microservice built from scratch
(vs. reuse of an existing worker pattern), custom correlation logic across multiple message events,
DMN decision requirements with several linked tables or a complex hit policy, or
reconciliation/retry logic beyond Zeebe's built-in job retry mechanism. A same-pattern reuse (new
REST connector matching an existing worker's shape) can legitimately size smaller — say so
explicitly rather than defaulting every integration to the same size.

## Complexity signals specific to Camunda

Flag **High/Complex** when a story involves: cross-process choreography via message correlation
(multiple independently-deployed process definitions coordinating), self-managed infrastructure
changes (Kubernetes/Helm) rather than SaaS-hosted, DMN with many decision tables or a `COLLECT`/
`RULE ORDER` hit policy, high-volume async job handling requiring backpressure/throughput tuning, or
custom job worker development instead of an out-of-box Connector.
