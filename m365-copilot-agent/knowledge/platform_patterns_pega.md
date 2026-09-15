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

## Note on other platforms

No pattern file yet exists for Camunda, Salesforce, or custom-code builds. When one of those is named
as the target platform, say so explicitly and fall back to general SDLC/architecture best practice for
Design Details rather than inventing Pega-specific claims for a non-Pega platform. Add a new
`platform_patterns_<name>.md` file here (same structure as this one) the first time the team runs this
skill against a new platform, so it accumulates over time.
