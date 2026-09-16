# Repository custom instructions

These instructions load automatically into every Copilot Chat request in this repo. The section
below applies **only** when the user is asking you to build, convert, or draft an agile backlog
from requirements artifacts (Excel requirements, meeting transcripts, process-flow diagrams, PDFs,
Word docs) — for any other coding task in this repo, ignore everything below and behave normally.

This is a Copilot-portable version of the `agile-backlog-builder` skill that lives at
`.claude/skills/agile-backlog-builder/` in this repo (built for Claude Code). Copilot doesn't read
that skill-package format or auto-invoke it, so this file inlines the same rules as plain
instructions. If the source skill files are ever updated, this file should be updated to match —
it will not pick up changes automatically.

---

## Agile Backlog Builder (Copilot version)

Converts requirements artifacts into a review-ready, JIRA-importable backlog of epics, features,
and user stories with full metadata.

Do not skip straight to drafting stories. This skill's output was once benchmarked against
manually-elaborated requirements and found to under-size large capabilities, template its
acceptance criteria, under-model integrations, and drop cross-cutting case controls. The Discovery
step and the metadata/sizing rules below exist specifically to close those gaps — don't shortcut
them under time pressure.

### Before drafting anything, ask the user for:

**Always required:**
1. **Development platform, version, and hosting mechanism** (e.g., "Pega Constellation/Infinity
   24.1, on-prem" vs "Camunda 8.9, cloud"). This changes what "design details" should look like per
   story. Pega-specific conventions are below — for any other platform, use general SDLC/
   architecture best practice for design details and say explicitly that no platform-specific
   pattern library exists yet for it.
2. **New build or enhancement to an existing application?** New builds need foundational/enabler
   stories (see below) that enhancements usually don't.
3. **Which source artifacts are actually provided**, and read all of them before drafting. If a
   process-flow diagram is an image/PDF, describe back the flow you extracted so the user can catch
   a misread early.

**Ask only if not already established for this project:**
4. **JIRA hierarchy model.** Default assumption: standard JIRA Software, where "Feature" is not a
   native issue type. Represent Feature as a **Label** (`feature:<name>`) and/or **Component** on
   each Story; the Story's real JIRA parent is the Epic (via Epic Link). If the user's JIRA has
   Advanced Roadmaps/Plans or Align, ask — Feature can then be a true issue type.
5. **Existing epic/feature taxonomy** to align new epics to, so this doesn't create duplicates.
6. **Mandatory NFRs/compliance standards** that should generate their own enabler stories or AC.
7. **JIRA field scheme**, if known (custom field names/IDs for Story Points, Priority values,
   Acceptance Criteria field). Default to the generic mapping below if none is provided, and say
   clearly it must be confirmed against the real project before import.
8. **Historical sizing actuals**, if the team has them (prior implementation hours/points for
   comparable work on this platform). If none exist, size from first principles and say so — don't
   invent calibration data.

If source artifacts conflict, or a requirement is too vague to size, don't guess silently — list it
as an open question and, if it's significant enough to change epic/feature grouping, ask the user
directly before continuing.

### Discovery — build these before drafting a single story

**Case-Type Matrix**: enumerate every case type implied by the source, including specialized/child
case types and secondary request types (e.g., an "assistance request" raised against a parent
case) — these are exactly what gets missed drafting straight from a narrative transcript. Columns:
Case Type, Parent Case Type, Trigger/Entry Point, Primary Actor, Key Stages, Special Handling
Notes. Every story should trace to a row here.

For **every** case type, explicitly evaluate against this cross-cutting case-control checklist
rather than only modeling the happy path — confirm with the user or infer from source which apply,
and write a story (or an explicit "not applicable" note) for each: **Withdraw, Hold, Resume, Skip,
Return/Go-back, Reassign, Merge, Duplicate detection, Dependency/relationship gating, Exception
processing.**

**Interface Inventory**: enumerate every system-to-system interface before drafting any integration
story. Columns: Interface ID, Source System, Target System, Direction, Protocol,
Interchange/Message Count, Data Objects Exchanged, Error Handling Approach, System of Record. Every
integration story must cite an Interface ID. Don't default an integration to a routine 8-point
story — size it on data-object count, sync vs. async, error/retry/reconciliation depth, and whether
it's a new pattern or reuse of an existing one.

**A capability is not automatically one story.** If something from the source (e.g., "draft
submissions and proposals") is clearly a major, multi-part capability, model it as its own
**Feature** from the start — don't draft one inflated story and rely on the size-gate to catch it
later.

### Epic / Feature / Story decomposition rules

Many-to-one grouping at every level — never 1:1:
- **Epic**: a large business capability/initiative, typically spanning a release. Normally contains
  **3+ features**. A single-feature epic is either too narrow (fold it into a sibling) or actually
  feature-sized.
- **Feature**: a deliverable, demoable slice of the epic, roughly one release/PI, usable end-to-end
  by a real actor. Normally contains **2–8 stories**. Represent as Label/Component per JIRA rule
  above.
- **Story**: satisfies INVEST (Independent, Negotiable, Valuable, Estimable, Small, Testable).
  Format: `As a <role>, I want <capability>, so that <benefit>`.

Process: extract every discrete requirement/capability/process step, cross-checked against the
Case-Type Matrix and Interface Inventory so nothing enumerated there is dropped → cluster top-down
into epics by business capability (not bottom-up from drafted stories) → cluster into features by
actor journey/deliverable slice → **enumerate variant dimensions explicitly** (standard/regulatory
type, product/segment, actor role, review/approval path, integration availability) and generate a
distinct story per meaningful variant rather than one generic story that hides the variation →
draft stories, splitting when too large or needing more than 6 AC (workflow-step, business-
rule/variation, CRUD-operation, interface/channel, happy-path-vs-exception splits).

### Story metadata (every story needs all of these)

| Field | Definition |
|---|---|
| Story ID | Stable short ID for dependency references, e.g. `EPC1-FT2-ST03` |
| Title | Short imperative summary |
| Epic | Parent epic name |
| Feature | Parent feature name (→ Label/Component) |
| Issue Type | `Story`, `Enabler` (infra/architecture work, no direct end-user value), or `Spike` (time-boxed research when unsizeable) |
| Category | `Functional` or `Foundation/Overhead` (environments, DevOps, testing, hardening, contingency) — kept separate from Issue Type so functional and overhead points don't get conflated in rollups |
| Summary | The As-a/I-want/so-that statement |
| Description | Fuller narrative, business rule detail; integration stories cite the Interface ID, case-control stories cite the Case-Type Matrix row |
| Acceptance Criteria | 1–6 criteria (see below); more than 6 means split the story instead |
| Sizing Drivers | The concrete facts behind the size: screen/view count, business rules/variants, data elements, integration touchpoints, exception paths — this is what makes a size defensible instead of a guess |
| Dependencies | Story IDs that must complete first; blank if none — don't invent dependencies |
| Priority | State the scale used (MoSCoW, P1–P4, etc.); derive from source-artifact urgency signals plus your judgment on what's load-bearing — explain the reasoning when it isn't obvious from source |
| Story Size | **Fibonacci only**: 1, 2, 3, 5, 8, 13. See sizing section below — 13 is a hard split-gate and 21 isn't a normal story size |
| Complexity | Low / Medium / High / Complex — independent of size (technical risk, not effort) |
| Design Details | Implementation approach, informed by the platform pattern section below |
| Actor/Persona | Primary role from the story sentence |
| Labels/Components | Feature label plus module/team labels |
| Requirement Origin | `Sourced` (traceable to a specific artifact location) or `Inferred` (filled in by judgment) — required on every story |
| Source Reference | Which artifact + location this traces to (transcript timestamp, requirement row, flow step, or matrix/inventory row ID); if Inferred, say what was inferred and why |
| Assumptions | Anything inferred rather than stated |
| Out of Scope | Explicit non-goals |
| Status | `Draft` until the user reviews it — never mark generated stories `Ready` |

### Acceptance criteria rules — and the anti-templating rule

- 1–6 per story; more means split the story, don't trim criteria to force-fit.
- Choose format per story: **Given/When/Then** for behavior/flow-driven stories (state
  transitions, multi-step interactions); **bulleted "the system shall..."** for simple
  condition/validation stories. Default to Given/When/Then when unsure.
- Every criterion must be independently testable and specific (no "handles errors appropriately" —
  name the error and expected behavior).
- **Don't reuse near-identical "created or updated / validated / audited" boilerplate across
  stories unless the behavior is genuinely identical.** Generic AC is a symptom of a generic story —
  if every story's AC reads the same, check whether Sizing Drivers is actually populated with real,
  story-specific facts. AC should make the specific rule/screen/variant/exception path legible to
  someone who's never seen the source.

### Sizing, complexity, and the split gate

Fibonacci size is relative effort, not days: 1–2 = well-understood single-component change; 3–5 =
multiple components or some unknowns; 8 = real unknowns/multi-system coordination; **13 is the
largest size a real story should carry, and it's a hard split-gate** — re-run the splitting patterns
before finalizing any 13, don't just size it and move on. **21 is not a normal story size** — reserve
it for a Spike; a Story/Enabler at 21 should be split. If a capability keeps coming out at 13+ no
matter how it's split, it probably wasn't story-sized to begin with — it's a Feature.

Complexity reflects technical risk (fragile integration, unclear requirements, new-to-team tech,
regulatory sensitivity), independent of size — state briefly why something is High/Complex when it
isn't obvious.

If the user has historical actuals for comparable work, use them as a sanity check on generated
sizes instead of a flat conversion factor — ask what they have before assuming none exists.

### New build vs. enhancement

- **New build**: include foundational/enabler stories the platform needs before feature work can
  start (environment setup, base data model, core integration scaffolding, security baseline,
  CI/CD if in scope). Group under a dedicated "Platform Foundation" epic unless one exists. Tag
  `Category: Foundation/Overhead`.
- **Enhancement**: assume the foundation exists. Only add foundational/enabler stories for the
  specific new capability. If unclear whether something foundational already exists, ask — don't
  assume either way.

### Output

Produce output as a **Markdown table** (or CSV text if the user wants to paste into Excel directly),
in this order: Case-Type Matrix, Interface Inventory, Epics, Features, Stories (all metadata
columns above), Traceability, Open Questions. Copilot Chat doesn't generate `.xlsx` files directly —
if the user wants the actual formatted workbook with tabs and dropdowns, point them to the template
already in this repo at `.claude/skills/agile-backlog-builder/templates/backlog_template.xlsx`, or to
running this same workflow through Claude Code, which can generate that file directly.

Don't generate a JIRA CSV export until the user has reviewed and approved the draft — it's a
mechanical transform of the approved content, not a separate creative pass. Generic JIRA CSV field
mapping:

| Backlog field | JIRA CSV column | Notes |
|---|---|---|
| Title | Summary | |
| Issue Type | Issue Type | `Enabler`/`Spike` → `Story` + matching label, unless the project has those as real issue types |
| Epic | Epic Link | Needs the target epic's real JIRA key (import epics first, in a separate pass) |
| Feature | Labels and/or Components | `feature:<name>` label — standard JIRA Software has no native Feature issue type |
| Summary + Description + Design Details + Source Reference + Assumptions + Out of Scope | Description | Each under its own heading within the Description field, unless the project has dedicated custom fields |
| Acceptance Criteria | Description ("Acceptance Criteria" heading) unless the project has a dedicated AC field (Xray/Zephyr) | |
| Dependencies | Linked Issues | Needs target keys to exist already — do this in a second pass |
| Priority | Priority | Confirm the project's actual Priority scheme |
| Story Size | Story point estimate / project's actual custom field | Field ID varies per JIRA instance — confirm before import |
| Complexity | Labels (`complexity:high`, etc.) unless a custom field exists | |
| Category | Labels (`category:foundation-overhead`, etc.) unless a custom field exists | |

Always tell the user this mapping is generic and must be checked against their real project's field
scheme before import.

### Pre-return quality gate

Before presenting the backlog, check:
- Case-Type Matrix and Interface Inventory were built before stories, and every story traces to a
  row in one of them (or is explicitly Foundation/Overhead)
- Every case type was evaluated against the case-control checklist, with a story or explicit
  "not applicable" note for each control
- Every epic has ≥2 features; every feature has ≥2 stories (or a stated reason it's an exception)
- No story has more than 6 acceptance criteria
- No story is sized at 21; every 13 was actually run through the split check, not left there by
  default
- Acceptance criteria aren't copy-pasted boilerplate across unrelated stories
- Sizing Drivers is populated with real counts, not left blank or vague
- Every story has Priority, Complexity, Category, Design Details, Requirement Origin, and Source
  Reference populated
- Dependencies reference only Story IDs that exist in this backlog
- Every assumption made during generation is captured as an open question, not just the obvious ones

### Platform patterns — Pega (Constellation/Infinity)

Use when the target platform is Pega. Confirm the specific version and hosting — the conventions
below assume Constellation UI on a reasonably current version (23.x+); flag explicitly when a
detail depends on a specific version/hosting model the user hasn't confirmed.

**Foundational/enabler stories for a new build:** application/case-type architecture (implementation
layer over framework/industry layer, ruleset and class structure), case lifecycle definition (stages,
processes, statuses) per primary case type, data model/data pages for core entities including
integration data pages, access group/role/privilege structure, DX API/Constellation UI shell setup
if a custom portal is needed, environment/pipeline setup if CI/CD is in scope, security baseline.

**Design Details conventions:**
- UI work → reference Constellation **DX components**, views, and the App Studio configuration
  (e.g., "DX list component bound to `.Items`"), not generic "build a UI screen"
- Process/flow work → reference **Case Types, Stages, Processes, Steps**, and whether logic belongs
  in a **Flow Action**, **Data Transform**, or **Activity** (activities only where DX/App
  Studio-native constructs genuinely can't do it — call this out as a deviation from best practice)
- Business rules → reference **Decision Tables/Trees** or **Constraints/Validations**
- Integrations → reference **Connectors** (REST/SOAP/etc.) and whether an integration wrapper class
  or data page is needed
- Reporting → reference **Report Definitions** or Constellation reporting components

**Case-control constructs** (map each Case-Type Matrix control to its Pega mechanism, don't leave it
generic): Withdraw → `Resolve-Withdrawn`; Hold/Resume → case-level Suspend/Resume with a defined
resume trigger; Skip → conditional stage/step navigation gated by a when-rule; Return/Go-back →
prior-stage routing, usually with a reason code; Reassign → worklist/workbasket move (manual or
rule-driven); Merge → custom logic, no OOTB support — usually at least Medium complexity; Duplicate
detection → Decision Table/Prospective Search at case creation; Dependency/relationship gating →
Case Relationships plus a when-rule guarding stage entry; Exception processing → work-queue routing
on connector/activity failure, `pxException` handling, or a dedicated exception case type.

**Correspondence/proposal generation is a Feature, not a story** — decompose into: template/content
inventory per variant, content rule/template design per variant, data-assembly Data Transforms,
generation trigger, review/approval routing (overlaps with case-control stories), and delivery
channel (its own integration story). Multiple product lines/standard types/review paths for
correspondence is a variant-dimension trigger, not a reason for one large story at the 13-point
ceiling.

**Integration sizing signals:** a new Connector/data page built from scratch, SOAP or file-based
protocols with custom parsing, multi-step orchestration across connectors, or reconciliation/retry
logic push a story above a routine 5–8. A same-pattern reuse can size smaller — say so explicitly.

**Complexity signals:** flag High/Complex for cross-application/cross-ruleset dependencies, changes
to inherited framework rules (vs. implementation-layer overrides), custom activities/Java steps
instead of out-of-the-box constructs, or integration with legacy/non-REST systems.

### Platform patterns — Camunda (Platform 8 / Zeebe)

Use when the target platform is Camunda. Confirm it's 8.x (Zeebe engine — Camunda 7's embedded
engine is architecturally different, these conventions don't transfer) and hosting (SaaS vs.
self-managed Kubernetes, which adds real foundational scope).

**Foundational/enabler stories for a new build:** cluster provisioning (SaaS cluster or self-managed
Kubernetes/Helm deployment of Zeebe, Operate, Tasklist, Identity, Connectors), identity/auth (OIDC,
Keycloak for self-managed), CI/CD for BPMN/DMN deployment separate from app code CI/CD, job worker
scaffolding, multi-tenancy config if applicable, Operate/Optimize monitoring integration.

**Design Details conventions:**
- Process/flow work → reference the process definition and specific element types (User Task,
  Service Task, Business Rule Task, gateway types, boundary events, call activity vs. embedded
  sub-process) — not generic "add a process step"
- Business rules → reference the **DMN decision table**/decision requirements diagram and hit policy
- Integrations → reference whether it's a **Job Worker** (external task pattern) or an out-of-box
  **Connector**, and whether new or reused
- UI/human tasks → reference **Camunda Forms** (form-js) vs. a custom front-end on the Tasklist/Zeebe
  REST APIs — state which
- Reporting → reference **Optimize** (Enterprise, don't assume licensed) or custom reporting off
  Operate

**Case-control constructs** (Camunda is an orchestration engine, not case management — most of these
need explicit modeling, don't assume "the engine handles it"): Withdraw → Cancel Process Instance
via an interrupting boundary event correlated to a message, not a bare ops-console cancellation;
Hold → an explicit modeled wait state (receive task/boundary event), no native hold button; Resume →
message correlation into that wait state; Skip → a conditional gateway fed by a process variable —
don't rely on Operate's "Modify Instance," that's an ops tool; Return/Go-back → an explicit rework
loop in the BPMN diagram, no native "go back"; Reassign → Tasklist claim/unclaim or candidate group
reassignment; Merge → no native support, build via message correlation on a shared business key,
flag at least Medium complexity; Duplicate detection → custom DMN/service-task check by business
key; Dependency/relationship gating → message correlation or call activities gating on a related
instance's state; Exception processing → Zeebe raises incidents automatically on job failures
(ops-level retry in Operate); business-facing exception handling needs explicit BPMN error/
escalation events.

**Correspondence generation is a Feature, not a story** — Camunda has no native document engine, so
this is always a Service Task/Job Worker calling an external templating service. Decompose into
template inventory per variant, the job worker populating templates, the BPMN trigger,
review/approval routing (often its own sub-process), and delivery-channel integration as a separate
story.

**Integration sizing signals:** a new job worker built from scratch, custom correlation logic across
multiple message events, DMN with several linked tables or a complex hit policy, or
reconciliation/retry beyond Zeebe's built-in job retry push above a routine 5–8.

**Complexity signals:** cross-process choreography via message correlation, self-managed
infrastructure changes rather than SaaS, complex DMN hit policies, high-volume async job handling
needing backpressure tuning, custom job worker development instead of an out-of-box Connector.

### Platform patterns — Appian

Use when the target platform is Appian. Confirm the version (frequent cloud releases change Records/
Sites/process-model behavior) and hosting (Appian Cloud is the norm; on-prem is uncommon — confirm).

**Foundational/enabler stories for a new build:** application object structure, base Record Type
design (external data store vs. native Appian store vs. process-backed), security Groups/role map,
Connected System(s) and credential/environment-variable setup, environment/ALM pipeline for
DEV/TEST/PROD promotion, base Site structure if new end-user-facing.

**Design Details conventions:**
- UI work → reference the **Interface** (SAIL), what Record/Related Action it's bound to, and
  whether it's a Site page or process Task form
- Process/flow work → reference **Process Model** nodes and whether logic belongs in a node, an
  **Expression Rule**, or an **Integration** object
- Business rules → reference **Decision** objects or Expression Rules
- Integrations → reference the **Connected System**/Integration object, new or reused
- Reporting → reference **Reports**/dashboards built on Record Types

**Case-control constructs** (Appian's Record + Process Model combo gives more native support than a
pure BPM engine, but still needs explicit modeling): Withdraw → a Related Action triggering an
explicit cancel/exit path; Hold → a wait node or a status field gating other Related Actions, with
an explicit resume trigger; Resume → the paused node's triggering event or status clearing; Skip →
a conditional gateway or a Related Action bypassing steps under a business rule; Return/Go-back →
an explicit rework loop node with a reason/comment capture; Reassign → built-in Task list Reassign
action or the Reassign Task smart service, manual or rule-driven; Merge → no native support, build
via a rule linking two Records, at least Medium complexity; Duplicate detection → an Expression Rule
or Record query at intake; Dependency/relationship gating → Related Records plus a rule/gateway
condition on a related Record's status; Exception processing → process model error nodes/subprocess
handling, or Process Admin Console for ops-level retry.

**Document generation is a Feature, not a story** — Appian's **Generate Document** smart service
makes this common but still multi-part: template inventory per variant, template design per
variant, data-mapping/merge-field assembly, the triggering process node, review/approval routing
(overlaps with case-control stories), and delivery channel as its own integration story.

**Integration sizing signals:** a new Connected System from scratch, SOAP/on-premises systems
requiring Appian's on-prem connectivity rather than direct Cloud-to-Cloud REST, multi-step
orchestration across Integration objects, or custom error-handling/retry push above a routine 5–8.

**Complexity signals:** custom plugins/Java components (rare but a strong signal), heavy
conditional/dynamic SAIL interfaces, cross-application Record relationships, legacy/on-premises
integration requiring a gateway.

### Platform patterns — Unqork

Use when the target platform is Unqork. Confirm the version/release train and which core components
are actually in use. **Important:** Unqork has no native business-process/case-management engine, no
native work-queue/task-assignment engine, and no native wizard back-stack — every case-control and
orchestration capability is manually built from Modules, Data Tables, and Workflow components.
Don't size Unqork case-control stories by analogy to Pega/Appian/Camunda — they're almost always
more custom, and therefore larger, on this platform.

**Foundational/enabler stories for a new build:** environment/tenant setup + Deployment Manager
pipeline, base Data Table schema for core entities, authentication/SSO + entitlement config, core
Reference Data Tables, base module template/theming, plugin/integration framework setup (decide
early whether marketplace plugins cover the need or custom plugin development is required).

**Design Details conventions:**
- UI work → reference the specific **Components** used and which **Module** they live in
- Process/flow work → reference the **Workflow** component's logic nodes and the specific
  module-to-module navigation/routing logic, naming modules and transition conditions
- Business rules → reference Workflow conditional logic or a Reference Data Table-driven lookup
- Integrations → reference the specific **Plugin** or RESTv2/Integration component, new or reused
- Reporting → limited native reporting vs. Pega/Appian — reference Unqork's own export capability or
  a downstream BI tool; don't assume rich native reporting

**Case-control constructs** (all custom-built — call this out explicitly rather than implying
platform support): Withdraw → a status field update gated by conditional logic; Hold → a status flag
plus conditional logic blocking navigation, often with a dedicated Hold Reason table; Resume → a
status transition triggered by user action or scheduled job; Skip → custom Workflow/navigation logic
evaluating a data condition; Return/Go-back → custom navigation logic — no built-in wizard
back-stack, so journey-state tracking is often its own sub-scope; Reassign → an owner/assigned-to
field plus a module or admin screen — no built-in work-queue engine, so this is commonly
under-scoped, size it accordingly; Merge → fully custom, no native support, at least Medium-High
complexity; Duplicate detection → a plugin call or Data Table query at intake; Dependency/
relationship gating → a relationship field plus conditional logic on a related record's status;
Exception processing → error-branch handling within Workflow components, or a dedicated
error-logging table plus admin remediation module.

**Document generation is a Feature, not a story** — built via a document-generation plugin or
external document service call. Decompose into template inventory per variant, the plugin call
populating each variant, the module/Workflow trigger, review/approval routing (overlaps with
case-control stories), and delivery channel as its own integration story.

**Integration sizing signals:** custom plugin development from scratch (no marketplace plugin
covers it), multi-step orchestration across several plugins/RESTv2 calls, or hand-built
retry/resilience logic (no native support) push above a routine 5–8.

**Complexity signals:** flag High/Complex by default for any case-control or workflow-orchestration
capability (these skew higher than case-management-native platforms), heavy custom JavaScript in
scripting components, multi-module journeys with complex branching, and any custom-built (vs.
marketplace) plugin.

### Platform patterns — Microsoft Power Apps (Power Platform)

Use when the target platform is Power Apps. Confirm the app model — **Canvas App** (Power Fx
formula-driven) vs. **Model-Driven App** (Dataverse, form/view/BPF-driven) — these are
architecturally different and Design Details should always say which. Also confirm whether this
sits on a Dynamics 365 base (e.g., Customer Service's Case table) or a fully custom Dataverse
schema.

**Foundational/enabler stories for a new build:** environment strategy (Dev/Test/Prod, Sandbox) with
Dataverse database allocation, Dataverse table/schema design (extends Dynamics 365 base vs. fully
custom), security role/Business Unit/Team design, solution architecture (managed solution + ALM
pipeline via Power Platform Pipelines or Azure DevOps, environment variables/connection references),
connector/custom connector setup, base Business Process Flow for the primary case/record type if
case-management-shaped.

**Design Details conventions:**
- UI work → Canvas App: the screen and Power Fx formulas/controls; Model-Driven App: the Form, View,
  or BPF stage — always state which app model
- Process/flow work → reference **Business Process Flow** stages or **Power Automate** cloud flow
  triggers/actions, naming the Dataverse table(s)
- Business rules → **Dataverse Business Rules** for simple field-level logic vs. a flow
  condition/**Dataverse plugin** for anything more complex — state which tier, a plugin is
  materially larger than a Business Rule
- Integrations → reference the **Connector** (standard/premium/custom via OpenAPI), new or reused
- Reporting → reference **Power BI** dashboards or native Model-Driven App views/charts

**Case-control constructs** (Dataverse/BPF give genuinely native support for several of these — say
so when it applies, it should size smaller than a custom build): Withdraw → native Status/Status
Reason field transition via a command-bar button or bound flow — a strong out-of-box fit; Hold → a
custom "On Hold" Status Reason plus a Business Rule/flow blocking progression, also a good native
fit; Resume → a flow/user action clearing the Hold status reason; Skip → BPFs support native
branching logic in current Dataverse versions — confirm before assuming custom build is needed;
Return/Go-back → BPFs natively support returning to a previous stage; Canvas Apps need custom
screen-navigation logic instead; Reassign → native Dataverse ownership reassignment (Assign action)
or queue-based assignment, another strong native fit; Merge → native Merge exists for certain base
tables (Account/Contact) but custom tables need a custom flow/plugin — check table type; Duplicate
detection → native Duplicate Detection Rules, a strong out-of-box fit, size smaller when used;
Dependency/relationship gating → table relationships plus a Business Rule/flow condition; Exception
processing → Power Automate error handling (`Configure Run After`, Scope/Catch) or Dataverse plugin
exceptions, surfaced in flow run history for ops-level retry.

**Document generation is a Feature, not a story** — Power Platform has native paths (Dataverse Word
Templates, or a flow's "Populate a Microsoft Word template" action) that can make this smaller than
platforms with no native generation, but it's still rarely one story with multiple variants.
Decompose into template inventory per variant, template design per variant, the trigger,
review/approval routing (a BPF stage or Power Automate approval, overlaps with case-control
stories), and delivery channel as its own integration story if beyond a simple email action.

**Integration sizing signals:** a new custom connector built from an OpenAPI spec vs. a
standard/premium prebuilt connector, a multi-step flow with explicit error handling vs. a simple
trigger-action flow, or Dataverse plugin (C#) development — a stronger signal than a declarative
Business Rule/flow — push above a routine 5–8.

**Complexity signals:** custom Dataverse plugins (C#) instead of declarative Business Rules/flows,
PCF custom component development, on-premises integration requiring the On-Premises Data Gateway,
solution-layering/ALM complexity across environments, heavy Power Fx formula complexity.

No pattern library yet exists here for Salesforce or other custom-code builds — for those, use
general SDLC/architecture best practice for Design Details and say so explicitly rather than
inventing platform-specific claims.
