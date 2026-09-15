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

If source artifacts conflict, or a requirement is too vague to size, don't guess silently — list it
as an open question and, if it's significant enough to change epic/feature grouping, ask the user
directly before continuing.

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

Process: extract every discrete requirement/capability/process step from the artifacts → cluster
top-down into epics by business capability (not bottom-up from drafted stories) → cluster into
features by actor journey/deliverable slice → draft stories, splitting when a story is too large or
would need more than 6 AC (workflow-step splits, business-rule/variation splits, CRUD-operation
splits, interface/channel splits, happy-path-vs-exception splits).

### Story metadata (every story needs all of these)

| Field | Definition |
|---|---|
| Story ID | Stable short ID for dependency references, e.g. `EPC1-FT2-ST03` |
| Title | Short imperative summary |
| Epic | Parent epic name |
| Feature | Parent feature name (→ Label/Component) |
| Issue Type | `Story`, `Enabler` (infra/architecture work, no direct end-user value), or `Spike` (time-boxed research when unsizeable) |
| Summary | The As-a/I-want/so-that statement |
| Description | Fuller narrative, business rule detail, source-artifact link |
| Acceptance Criteria | 1–6 criteria (see below); more than 6 means split the story instead |
| Dependencies | Story IDs that must complete first; blank if none — don't invent dependencies |
| Priority | State the scale used (MoSCoW, P1–P4, etc.); derive from source-artifact urgency signals plus your judgment on what's load-bearing — explain the reasoning when it isn't obvious from source |
| Story Size | **Fibonacci only**: 1, 2, 3, 5, 8, 13, 21. Anything feeling bigger than 13 should almost always be split, not sized |
| Complexity | Low / Medium / High / Complex — independent of size (technical risk, not effort) |
| Design Details | Implementation approach, informed by the platform pattern section below |
| Actor/Persona | Primary role from the story sentence |
| Labels/Components | Feature label plus module/team labels |
| Source Reference | Which artifact + location this traces to (transcript timestamp, requirement row, flow step) |
| Assumptions | Anything inferred rather than stated |
| Out of Scope | Explicit non-goals |
| Status | `Draft` until the user reviews it — never mark generated stories `Ready` |

### Acceptance criteria rules

- 1–6 per story; more means split the story, don't trim criteria to force-fit.
- Choose format per story: **Given/When/Then** for behavior/flow-driven stories (state
  transitions, multi-step interactions); **bulleted "the system shall..."** for simple
  condition/validation stories. Default to Given/When/Then when unsure.
- Every criterion must be independently testable and specific (no "handles errors appropriately" —
  name the error and expected behavior).

### Sizing and complexity

Fibonacci size is relative effort, not days: 1–2 = well-understood single-component change; 3–5 =
multiple components or some unknowns; 8 = real unknowns/multi-system coordination; 13+ = split
before grooming. Complexity reflects technical risk (fragile integration, unclear requirements,
new-to-team tech, regulatory sensitivity) — state briefly why something is High/Complex when it
isn't obvious.

### New build vs. enhancement

- **New build**: include foundational/enabler stories the platform needs before feature work can
  start (environment setup, base data model, core integration scaffolding, security baseline,
  CI/CD if in scope). Group under a dedicated "Platform Foundation" epic unless one exists.
- **Enhancement**: assume the foundation exists. Only add foundational/enabler stories for the
  specific new capability. If unclear whether something foundational already exists, ask — don't
  assume either way.

### Output

Produce output as a **Markdown table** (or CSV text if the user wants to paste into Excel directly)
with columns matching the metadata schema above, split into Epics / Features / Stories sections.
Copilot Chat doesn't generate `.xlsx` files directly — if the user wants the actual formatted
workbook with tabs and dropdowns (Epics/Features/Stories/Traceability/Open Questions), point them to
the template already in this repo at `.claude/skills/agile-backlog-builder/templates/backlog_template.xlsx`,
or to running this same workflow through Claude Code, which can generate that file directly.

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

Always tell the user this mapping is generic and must be checked against their real project's field
scheme before import.

### Pre-return quality gate

Before presenting the backlog, check:
- Every epic has ≥2 features; every feature has ≥2 stories (or a stated reason it's an exception)
- No story has more than 6 acceptance criteria
- Every Story Size is a Fibonacci number
- Every story has Priority, Complexity, Design Details, and Source Reference populated
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

**Complexity signals:** flag High/Complex for cross-application/cross-ruleset dependencies, changes
to inherited framework rules (vs. implementation-layer overrides), custom activities/Java steps
instead of out-of-the-box constructs, or integration with legacy/non-REST systems.

No pattern library yet exists here for Camunda, Salesforce, or custom-code builds — for those, use
general SDLC/architecture best practice for Design Details and say so explicitly rather than
inventing platform-specific claims.
