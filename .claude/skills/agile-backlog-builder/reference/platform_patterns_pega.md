# Platform Patterns — Pega (Constellation / Infinity)

Loaded when intake identifies the target platform as Pega. Always confirm the **specific version**
(e.g., 24.1) and **hosting** (on-prem / Pega Cloud) — design details below assume Constellation UI on
a reasonably current version (23.x+); call out explicitly when a design detail depends on a specific
version or hosting model and the user hasn't confirmed it.

## Foundational / enabler stories for a **new build** on Pega

Don't generate feature stories before these exist (or are explicitly confirmed as already in place):
- Application/case-type architecture setup (implementation layer on top of framework/industry layer,
  ruleset and class structure)
- Case lifecycle definition (stages, processes, statuses) for each primary case type
- Data model / data pages for core entities, including integration data pages if external systems are
  in scope
- Access group / role / privilege structure
- DX API / Constellation UI shell setup if a custom portal or embedded experience is needed
- Environment/pipeline setup (Deployment Manager or equivalent) if CI/CD is in scope
- Security baseline (authentication profile, if on-prem: app server/security config)

## Design-detail conventions for Story rows

When writing the **Design Details** field for a Pega story, prefer platform-native constructs over
generic descriptions:
- UI work → reference Constellation **DX components**, views, and the App Studio configuration
  involved (e.g., "Add a new field on the Case View using a DX list component bound to `.Items`") —
  not generic "build a UI screen"
- Process/flow work → reference **Case Types, Stages, Processes, Steps**, and whether logic belongs in
  a **Flow Action**, **Data Transform**, or **Activity** (activities only where DX/App Studio-native
  constructs genuinely can't do it — call this out as a deviation from Pega best practice)
- Business rules → reference **Decision Tables/Trees**, or **Constraints/Validations**, not generic
  "validate the field"
- Integrations → reference **Connectors** (REST/SOAP/etc.) and whether an integration wrapper class or
  data page is needed
- Reporting → reference **Report Definitions** or the Constellation reporting components in scope

## Complexity signals specific to Pega

Flag **High/Complex** when a story involves: cross-application/cross-ruleset dependencies, changes to
inherited framework rules (vs. implementation-layer overrides), custom activities/Java steps instead
of out-of-the-box constructs, or integration with legacy/non-REST systems requiring custom connectors.

## Case-control constructs (map to the checklist in SKILL.md §2a)

When building the Case-Type Matrix and its case-control stories, map each control to its Pega-native
mechanism rather than a generic description — this is what keeps case-control stories from being
silently dropped or hand-waved as "the framework handles it":
- **Withdraw** → `Resolve-Withdrawn` (or a custom withdraw flow action); confirm whether withdrawal
  is allowed only from specific stages
- **Hold / Resume** → case-level Suspend/Resume (via `pyDefault` case actions or a custom
  Hold/Resume flow action with a defined resume trigger — manual, SLA-based, or external event); a
  story here should specify what actually triggers resume, not just "the case can be held"
- **Skip** → conditional stage/step navigation (`Change Stage`, or a skip flow action gated by a
  when-rule); specify the condition under which skip is allowed
- **Return / Go-back** → prior-stage routing (custom flow action or `Change Stage` backward), usually
  paired with a reason-code capture
- **Reassign** → standard case assignment reassignment (worklist/workbasket move); specify whether
  reassignment is manual (user-initiated) or rule-driven (routing/SLA-triggered)
- **Merge** → custom logic; Pega has no fully out-of-the-box case-merge — a merge story is almost
  always at least Medium complexity and should say so, covering what happens to each case's
  in-flight work and history
- **Duplicate detection** → typically a Decision Table/search-based check at case creation
  (`Prospective Search`, a report-definition-backed lookup, or a duplicate-check data transform);
  specify the matching criteria
- **Dependency / relationship gating** → Case Relationships (page list of related case IDs) plus a
  when-rule/guard gating stage entry on a related case's status; specify which relationship and
  which status
- **Exception processing** → generalized error handling: work queue routing on connector/activity
  failure, `pxException` handling, or a dedicated exception case type — specify which pattern applies
  rather than leaving "handles exceptions" generic

## Correspondence and document/proposal generation is a Feature, not a story

Drafting submissions, proposals, or other generated correspondence in Pega is rarely a single story —
treat it as its own **Feature** per SKILL.md §3's "is this actually a Feature" check, decomposed at
minimum into:
- Template/content inventory (what documents exist, by product/segment/standard type — this feeds
  the variant-dimension step in SKILL.md §3 step 4)
- Content rule and template design per variant (Correspondence rules, Content rules, Paragraph rules
  or Word/HTML templates), sized per variant rather than lumped into one story
- Data assembly feeding the template (Data Transforms/clipboard pages sourcing case, party, and
  product data into the document)
- Generation trigger (flow action, activity, or batch job that produces the document)
- Review/approval routing before send, if applicable — this overlaps with case-control stories in
  the Case-Type Matrix, not a separate concern to invent from scratch
- Delivery channel (email, portal, print/mail vendor integration) — an integration story in its own
  right per the Interface Inventory, not folded into the document-generation story

Sizing signal: if the source material describes multiple product lines, standard types, or review
paths for correspondence, that is a variant-dimension trigger (SKILL.md §3 step 4), not a reason to
write one large story and size it at the 13-point ceiling.

## Integration sizing signals specific to Pega

When sizing a story tied to an Interface Inventory row (SKILL.md §2b), Pega-specific drivers that
push a story above a routine 5–8: a new Connector class/data page built from scratch (vs. reusing an
existing integration pattern in the application), SOAP or file-based protocols requiring custom
parsing, multi-step orchestration across more than one connector call, or reconciliation/retry logic
beyond a simple synchronous request-response. A same-pattern reuse (new REST connector matching an
existing wrapper class's shape) can legitimately size smaller — say so explicitly rather than
defaulting every integration to the same size.

## Note on other platforms

No pattern file yet exists for Camunda, Salesforce, or custom-code builds. When one of those is named
as the target platform, say so explicitly and fall back to general SDLC/architecture best practice for
Design Details rather than inventing Pega-specific claims for a non-Pega platform. Add a new
`platform_patterns_<name>.md` file here (same structure as this one) the first time the team runs this
skill against a new platform, so it accumulates over time.
