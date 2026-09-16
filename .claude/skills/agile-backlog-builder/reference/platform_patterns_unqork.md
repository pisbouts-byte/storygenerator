# Platform Patterns — Unqork

Loaded when intake identifies the target platform as Unqork. Confirm the **specific version/release
train** the org is on and **which core components are actually in use** (Deployment Manager,
Orchestrate/scheduled jobs, specific plugin set) — Unqork is a no-code canvas platform with a
smaller, more implementation-specific standard toolkit than Pega/Appian/Camunda, so more here than
on other platforms genuinely varies by how a given team has built things. Flag assumptions
explicitly when the source material doesn't confirm a pattern.

**Important framing for this platform:** Unqork has no native business-process/case-management
engine, no native task/work-queue assignment engine, and no native multi-stage wizard/back-stack
state management. Every one of those is manually built from Modules, Data Tables, and Workflow
components. Case-control and orchestration stories on Unqork are almost always more custom — and
therefore larger and more complex — than the equivalent capability on Pega, Appian, or Camunda.
Don't size Unqork case-control stories by analogy to those platforms.

## Foundational / enabler stories for a **new build** on Unqork

Don't generate feature stories before these exist (or are explicitly confirmed as already in place):
- Environment/tenant setup across Dev/QA/UAT/Prod and Deployment Manager pipeline configuration for
  promoting Modules between them
- Base **Data Table** schema/data model design for core entities (the record store underlying the
  application, since Unqork has no separate case-management data layer)
- Authentication/SSO setup (SAML/OIDC integration) and entitlement/permission configuration
- Core **Reference Data Tables** (RDTs) needed for lookups across the application
- Base module template/theming setup (styling/branding components reused across modules)
- Plugin/integration framework setup for the external systems this build will call — decide early
  whether existing marketplace plugins cover them or custom plugin development is needed

## Design-detail conventions for Story rows

When writing the **Design Details** field for an Unqork story, name the actual component/module:
- UI work → reference the specific **Components** used (Input, Data Table, Panel, etc.) and which
  **Module** they live in, not generic "build a screen"
- Process/flow work → reference the **Workflow** component's logic nodes and the specific
  module-to-module navigation/routing logic — name the modules and the transition conditions, since
  this is hand-built rather than a platform-native process engine
- Business rules → reference **Workflow conditional logic** or a **Reference Data Table**-driven
  lookup, not generic "validate the field"
- Integrations → reference the specific **Plugin** (marketplace or custom-built) or the
  **RESTv2/Integration** component used, and whether it's a new custom plugin or reuse of an
  existing one
- Reporting → Unqork's native reporting is limited compared to Pega/Appian — reference whichever
  applies: Unqork's own export/reporting capability, or a downstream BI tool fed by Data Table
  exports; don't assume rich native reporting exists without confirming

## Case-control constructs (map to the checklist in SKILL.md §2a)

All of these are custom-built on Unqork — call this out explicitly in each story rather than
implying platform support:
- **Withdraw** → a status field update (e.g., "Withdrawn") on the Data Table record, driven by a
  user action in a module, gated by conditional logic restricting which statuses allow it
- **Hold** → a status flag plus conditional logic blocking further module navigation until cleared;
  commonly paired with a dedicated "Hold Reason" Data Table
- **Resume** → a status transition clearing the hold flag, triggered by a user action or a scheduled
  job (Unqork supports scheduled module runs / external API triggers)
- **Skip** → custom Workflow/navigation logic evaluating a data condition to bypass a module in the
  journey — specify the condition explicitly
- **Return / Go-back** → custom navigation logic routing to a prior module; Unqork has no built-in
  wizard back-stack, so journey-state tracking (which module the user was on, what to restore) is
  often its own sub-scope worth calling out, not an assumed detail
- **Reassign** → an "owner"/"assigned to" field on the Data Table plus a module or admin screen for
  reassignment — Unqork has no built-in work-queue/task-assignment engine, so this is one of the
  most commonly under-scoped areas on this platform; size it accordingly rather than treating it as
  a small config change
- **Merge** → fully custom logic combining two Data Table records; no native support — at least
  Medium-High complexity as a starting assumption
- **Duplicate detection** → a plugin call or a Data Table query component checking for an existing
  matching record at intake
- **Dependency / relationship gating** → a relationship field (foreign-key-style reference between
  Data Table records) plus conditional logic checking a related record's status before allowing
  progression
- **Exception processing** → error-branch handling within Workflow components for plugin/integration
  failures, or a dedicated error-logging Data Table plus an admin remediation module — specify which

## Document/correspondence generation is a Feature, not a story

Unqork implementations (frequently insurance/govtech) commonly need generated correspondence, built
via a document-generation plugin or an external document service call. Treat this as its own
**Feature**: template inventory by product/segment/standard type, the plugin/integration call
populating each template variant, the module/Workflow trigger point, review/approval routing
(overlapping with case-control stories, not invented separately), and the delivery channel as its
own integration story. Multiple templates or review paths is a variant-dimension trigger, not a
reason for one large story at the 13-point ceiling.

## Integration sizing signals specific to Unqork

Drivers that push a story above a routine 5–8: custom plugin development from scratch (no
marketplace plugin covers the need) vs. reuse of an existing plugin, multi-step orchestration across
several plugins/RESTv2 calls within one Workflow, or the lack of native retry/error-handling
requiring hand-built resilience logic (Unqork doesn't provide this out of the box the way a BPM
engine does). A same-pattern reuse of an already-built plugin can size smaller — say so explicitly.

## Complexity signals specific to Unqork

Flag **High/Complex** by default for: any case-control or workflow-orchestration capability (per the
framing note above, these skew higher than equivalent work on case-management-native platforms),
heavy custom JavaScript within scripting-capable components (a strong High/Complex signal on this
platform), multi-module journeys with complex branching logic (harder to maintain and test than a
platform-native process model), and any capability relying on a custom-built plugin rather than a
marketplace one.
