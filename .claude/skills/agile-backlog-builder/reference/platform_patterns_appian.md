# Platform Patterns — Appian

Loaded when intake identifies the target platform as Appian. Confirm the **specific version**
(Appian ships frequent cloud releases; behavior around Records, Sites, and process modeling has
evolved release over release) and **hosting** (Appian Cloud is the norm; on-premises/self-managed
Appian exists but is uncommon — confirm rather than assume Cloud).

## Foundational / enabler stories for a **new build** on Appian

Don't generate feature stories before these exist (or are explicitly confirmed as already in place):
- Application object structure — the Appian **Application** grouping objects (process models,
  interfaces, Record Types, rules, integrations) that will contain this work
- Base **Record Type** design for core entities, including whether Records sync from an external
  data store, a Dataverse-style native Appian data store, or are process-backed
- Security **Groups**/role map design (who can see/do what across Sites, Records, and process tasks)
- **Connected System(s)** and credential/environment-variable setup for external integrations
- Environment/ALM pipeline — Appian application packaging and promotion across DEV/TEST/PROD via
  Appian Deployments (or the Appian DevOps API/Azure DevOps integration if the team uses one)
- Base **Site** structure if this is a new end-user-facing application shell

## Design-detail conventions for Story rows

When writing the **Design Details** field for an Appian story, name the actual object type:
- UI work → reference the **Interface** (SAIL) involved, what **Record** or **Related Action** it's
  bound to, and whether it's embedded in a Site page or a process Task form — not generic "build a
  screen"
- Process/flow work → reference **Process Model** nodes (User Input Task, Script Task, Subprocess,
  Gateway) and whether logic belongs in a process node, an **Expression Rule**, or an **Integration**
  object
- Business rules → reference **Decision** objects or **Expression Rules**, not generic "business
  logic" — Appian Decisions are the closer analog to a decision table when the logic is tabular
- Integrations → reference the **Connected System** and **Integration** object used, and whether a
  new Connected System needs configuring or an existing one is reused
- Reporting → reference **Reports**/dashboards built on Record Types, or summary Interfaces —
  specify which

## Case-control constructs (map to the checklist in SKILL.md §2a)

Appian's Record + Process Model combination gives more native support for these than a pure BPM
engine, but several still need explicit modeling — don't assume "the framework handles it":
- **Withdraw** → a **Related Action** on the Record triggering an explicit cancel/exit path in the
  process model (an interrupting node or a "Cancel" smart service) — specify which stages allow it
- **Hold** → a wait node in the process model (paused pending a signal), or a status field on the
  Record gating other Related Actions until cleared — specify the resume trigger explicitly, not
  just "the record can be held"
- **Resume** → the paused node's triggering event (a Related Action, a Process Message, or an API
  call), or the status field clearing
- **Skip** → a conditional gateway in the process model, or a Related Action that advances Record
  status bypassing standard steps, gated by role/business rule — specify the condition
- **Return / Go-back** → an explicit rework loop node routing back to a prior user input task,
  typically paired with a reason/comment field capture
- **Reassign** → Appian tasks support built-in reassignment (Task list "Reassign" action, or the
  Reassign Task smart service), plus group-based queue routing — specify manual vs. rule-driven
- **Merge** → no native Record/case merge; build via a rule linking two Records and
  archiving/redirecting one — flag as higher complexity, at least Medium
- **Duplicate detection** → an Expression Rule or Record query (`a!queryRecordType`) checked at
  intake before creating a new Record/process instance
- **Dependency / relationship gating** → Related Records (Appian's record relationships) plus a rule
  or gateway condition checking a related Record's status field before proceeding
- **Exception processing** → process model error nodes / subprocess exception handling, or a
  dedicated exception-handling subprocess; node failures also surface in the Process Admin Console
  for ops-level retry — specify which pattern (business-facing vs. ops-only) applies

## Document/correspondence generation is a Feature, not a story

Appian's built-in **Generate Document** smart service (producing Word/PDF from a template) makes
document generation common but still multi-part. Treat it as its own **Feature**, decomposed into:
template inventory by product/segment/standard type (feeding the variant-dimension step), the
document template design per variant, the data-mapping/merge-field assembly feeding the template,
the process node triggering generation, review/approval routing (a Related Action or user task,
overlapping with case-control stories rather than invented separately), and the delivery channel
(email, Site download, external delivery integration) as its own integration story. Multiple
product lines or review paths for correspondence is a variant-dimension trigger, not a reason to
size one large story at the 13-point ceiling.

## Integration sizing signals specific to Appian

Drivers that push a story above a routine 5–8: a new Connected System built from scratch (vs. reuse
of an existing one), SOAP integrations or on-premises systems requiring Appian's on-prem
connectivity (site-to-site VPN or a data gateway) rather than direct Cloud-to-Cloud REST,
multi-step orchestration across several Integration objects within one process node, or
custom error-handling/retry logic beyond a simple synchronous call. A same-pattern reuse of an
existing Connected System can legitimately size smaller.

## Complexity signals specific to Appian

Flag **High/Complex** when a story involves: custom plugins or Java components (rare in Appian, but
a strong complexity signal when present), SAIL interfaces with heavy conditional/dynamic component
logic, cross-application Record relationships spanning multiple Appian applications, or integration
with legacy/on-premises systems requiring a gateway rather than direct connectivity.
