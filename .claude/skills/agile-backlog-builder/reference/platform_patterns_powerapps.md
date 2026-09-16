# Platform Patterns — Microsoft Power Apps (Power Platform)

Loaded when intake identifies the target platform as Power Apps. Confirm which app model is in
scope — **Canvas App** (pixel-precise, Power Fx formula-driven) vs. **Model-Driven App** (built on
Dataverse, form/view/Business Process Flow-driven) — these are architecturally different and design
details should always say which. Also confirm whether this build sits on top of a **Dynamics 365**
base application (e.g., Customer Service's Case table) or a purely custom Dataverse schema, since
that changes how much is already available vs. needs building.

## Foundational / enabler stories for a **new build** on Power Platform

Don't generate feature stories before these exist (or are explicitly confirmed as already in place):
- Environment strategy — Dev/Test/Prod (and Sandbox if used), with Power Platform environment
  provisioning and Dataverse database allocation
- **Dataverse table/schema design** for core entities, including relationships (lookup/N:N) and
  whether this extends a Dynamics 365 base table or is fully custom
- Security role / Business Unit / Team design in Dataverse
- **Solution architecture** — managed solution packaging and ALM pipeline setup (Power Platform
  Pipelines, or Azure DevOps if the team uses that instead), including environment variables and
  connection references for solution portability across environments
- Connector/custom connector setup for the external systems this build will integrate with
- A base **Business Process Flow** definition for the primary case/record type, if the app is
  case-management-shaped

## Design-detail conventions for Story rows

When writing the **Design Details** field for a Power Platform story, name the actual construct and
which app model it belongs to:
- UI work → for a **Canvas App**, reference the screen and relevant Power Fx formulas/controls; for
  a **Model-Driven App**, reference the Form, View, or Business Process Flow stage — always state
  which app model, since the two are built and sized differently
- Process/flow work → reference **Business Process Flow** stages/steps, or **Power Automate** cloud
  flow triggers/actions, and name the Dataverse table(s) involved
- Business rules → reference **Dataverse Business Rules** for simple declarative field-level logic,
  vs. a **Power Automate flow condition** or a **Dataverse plugin** for anything more complex or
  server-side — state which tier, since a plugin is a materially larger/higher-complexity story than
  a Business Rule
- Integrations → reference the specific **Connector** (standard, premium, or a custom connector
  built from an OpenAPI spec) used in the Power Automate flow, and whether it's new or reused
- Reporting → reference **Power BI** dashboards/reports built on Dataverse data, or native
  Model-Driven App views/charts — specify which

## Case-control constructs (map to the checklist in SKILL.md §2a)

Dataverse and Business Process Flows give genuinely native support for several of these — say so
when it applies, since it should size smaller than a fully custom build:
- **Withdraw** → Dataverse tables natively have **Status/Status Reason** fields; a withdraw action
  is typically a status transition (deactivation) triggered by a command-bar button or bound
  Power Automate flow — this is a strong out-of-box fit
- **Hold** → a custom "On Hold" Status Reason plus a Business Rule or Power Automate flow blocking
  further stage progression while held — also a good native fit given Status Reason support
- **Resume** → a flow or user action clearing the Hold status reason, resuming BPF stage progression
- **Skip** → Business Process Flows support **native branching logic** in current Dataverse
  versions — confirm whether that's sufficient, or whether a Power Automate flow/PCF component is
  needed for the specific skip condition; don't assume custom build is required without checking
- **Return / Go-back** → Business Process Flows natively support moving to a **previous stage**; for
  a Canvas App (no BPF), this needs custom screen-navigation/state logic instead — state which
  applies
- **Reassign** → Dataverse natively supports record **ownership reassignment** (the out-of-box
  Assign action, respecting security roles), or queue-based assignment if Dataverse queues are in
  use — another strong native fit
- **Merge** → Dataverse has a **native Merge** capability for certain base tables (e.g., Account/
  Contact in a Dynamics 365 base), but custom tables need a custom merge flow or plugin — specify
  which table type applies before assuming native support
- **Duplicate detection** → Dataverse has **native Duplicate Detection Rules** (configurable matching
  criteria run on create/update) — a strong out-of-box fit; size smaller when this native feature is
  used rather than building custom duplicate-check logic
- **Dependency / relationship gating** → Dataverse table relationships plus a Business Rule or
  Power Automate flow condition checking a related record's status before allowing progression
- **Exception processing** → Power Automate flow error handling (`Configure Run After`, Scope +
  Catch-style branches), or Dataverse plugin exception handling; failed flows surface in run history
  for ops-level retry — specify whether business-facing exception handling or just ops-level flow
  retry is in scope

## Document/correspondence generation is a Feature, not a story

Power Platform has more than one native path for this — **Dataverse Word Templates** (model-driven,
no separate integration needed) or a Power Automate flow using **"Populate a Microsoft Word
template"**/similar actions — which genuinely can make this smaller than on platforms with no native
document generation, but it's still rarely one story once multiple variants exist. Treat it as its
own **Feature**: template inventory by product/segment/standard type, the template design per
variant (Word template mapped to Dataverse fields), the trigger (a flow or a Dataverse command),
review/approval routing (a BPF stage or Power Automate approval — overlapping with case-control
stories, not invented separately), and the delivery channel (email via flow, portal download,
external send) as its own integration story if it goes beyond a simple email action.

## Integration sizing signals specific to Power Platform

Drivers that push a story above a routine 5–8: a **new custom connector** built from scratch
(defining an OpenAPI spec against a bespoke API) vs. reuse of a standard/premium prebuilt connector,
a Power Automate flow with multi-step orchestration and explicit error handling (`Scope` +
`Configure Run After`) vs. a simple trigger-action flow, or **Dataverse plugin** (C#) development,
which is a stronger complexity/size signal than a declarative Business Rule or flow. A standard
connector used in a simple flow can legitimately size small — say so explicitly.

## Complexity signals specific to Power Platform

Flag **High/Complex** when a story involves: custom **Dataverse plugins** (C#) instead of
declarative Business Rules/Power Automate flows, **PCF** (Power Apps Component Framework) custom
component development, integration with on-premises systems requiring the **On-Premises Data
Gateway**, solution-layering/ALM complexity across multiple environments, or heavy Power Fx formula
complexity in a Canvas App.
