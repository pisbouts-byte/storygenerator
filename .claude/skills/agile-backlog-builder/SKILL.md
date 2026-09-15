---
name: agile-backlog-builder
description: Converts mixed source artifacts (Excel requirements, meeting transcripts, process-flow diagrams, PDFs, Word docs) into a JIRA-importable agile backlog of epics, features, and user stories with full metadata. Use when asked to draft a backlog, write user stories, decompose requirements/transcripts/process flows into epics and stories, or prepare a backlog for JIRA import.
---

# Agile Backlog Builder

Turns raw requirements artifacts into a review-ready, JIRA-importable backlog. Optimized for
consistency across a team working on multiple delivery platforms (Pega, Camunda, custom code, etc.).

This skill has three halves:
1. **Intake** — ask the questions that change the shape of the output, before drafting anything.
2. **Discovery** — build the Case-Type Matrix and Interface Inventory *before* drafting a single
   story, so large/cross-cutting capabilities are sized as what they actually are instead of getting
   compressed into one convenient story.
3. **Generation** — apply epic/feature/story decomposition rules, populate full metadata, validate
   against hard constraints, and emit both a review workbook and a JIRA import file.

Do not skip straight to drafting stories. A backlog built before intake questions and discovery
artifacts are done will systematically under-size large capabilities, miss cross-cutting case
controls, under-model integrations, and produce acceptance criteria that look consistent but hide
what actually drove each story's size — this is a real failure mode observed when this skill's
output was benchmarked against manually-elaborated requirements; see the gap list §9 addresses.

---

## 1. Intake — always ask before generating

Ask these as a short batch of clarifying questions (not one at a time) before producing any backlog
content. If the user has already answered one in the conversation, don't re-ask it.

**Always required:**
1. **Development platform, version, and hosting mechanism.** E.g., "Pega Constellation/Infinity 24.1,
   on-prem" vs "Camunda 8.9, cloud (SaaS)" vs "custom Java/Spring, AWS." This determines which
   platform pattern file to load (see [reference/platform_patterns_pega.md](reference/platform_patterns_pega.md)
   for the first supported platform) and what "design details" look like for each story. If no
   pattern file exists yet for the named platform, say so explicitly and draft design details from
   general SDLC/architecture best practice instead of inventing platform-specific claims.
2. **New build or enhancement to an existing application?** New builds imply a set of foundational
   /enabler stories (see §7) that enhancements typically don't need — don't generate foundational
   setup stories for an enhancement unless the user says the foundation doesn't already exist.
3. **Source artifacts available.** Confirm what's actually provided (Excel requirements, transcript,
   process-flow diagram, PDF, Word doc) and read all of them before drafting. If a process-flow
   diagram is provided as an image/PDF, describe the flow you extracted back to the user briefly so
   they can catch a misread before it propagates into ten stories.

**Ask only if not already established for this team/project (reuse the answer across a session):**
4. **JIRA hierarchy model.** Default assumption for this skill: standard JIRA Software, where "Feature"
   is not a native issue type. Feature is represented as a **Label** (`feature:<name>`) and/or
   **Component** on each Story, while Story's real JIRA parent is the Epic (via Epic Link). If the
   user's JIRA has Advanced Roadmaps/Plans or Align, ask — Feature can then be a true issue type and
   the export mapping changes accordingly.
5. **Existing epic/feature taxonomy** to align new epics to, so this run doesn't create a duplicate
   "Claims Intake" epic that already exists under a different name.
6. **Mandatory NFRs / compliance standards** (security, accessibility, data residency, audit) that
   should generate their own enabler stories or acceptance criteria.
7. **JIRA field scheme**, if the user has one (custom field names/IDs for Story Points, Priority
   values, Acceptance Criteria field). Default to the generic mapping in
   [reference/jira_csv_mapping.md](reference/jira_csv_mapping.md) if none is provided, and say clearly
   that it's a generic template the user must confirm against their real project before import.
8. **Historical sizing actuals**, if the team has them (prior implementation hours/points for
   comparable stories on this platform). If a calibration reference exists — see
   [reference/sizing_calibration_template.md](reference/sizing_calibration_template.md) — load it and
   use it per §5. If none exists, say so and size from first principles, and suggest the team build
   one as they groom this backlog (actuals from this project become next project's calibration data).

**Mid-generation clarifying questions:** if source artifacts conflict, a process-flow step has no
corresponding requirement text, or a requirement is too vague to size, do not guess silently — list
these as flagged items in the **Open Questions** tab (see §6) AND, if the ambiguity is large enough to
change epic/feature grouping, ask the user directly before continuing.

---

## 2. Discovery — build these before drafting a single story

Two artifacts must exist before story drafting starts. Skipping this step is the single biggest
cause of under-sized backlogs: large capabilities get compressed into one convenient story, and
cross-cutting controls get silently dropped because nothing forced them onto a list first.

### 2a. Case-Type Matrix

Enumerate **every** case type implied by the source artifacts, including specialized/child case
types and secondary request types (e.g., an "assistance request" or "exception request" raised
against a parent case) — these are exactly the kind of thing that gets missed when stories are
drafted directly from a narrative transcript. Columns: Case Type, Parent Case Type (blank if
top-level), Trigger/Entry Point, Primary Actor, Key Stages, Special Handling Notes. Build this as
its own workbook tab (§6). Every story in the backlog should trace to a row in this matrix — if a
story doesn't, that's a sign either the matrix is incomplete or the story is scope creep.

For **every** case type in the matrix, explicitly evaluate against the cross-cutting case-control
checklist below rather than only modeling the happy-path flow. Confirm with the user or infer from
source which controls apply, and generate case-control stories accordingly — don't assume a control
is "implied" by the case type and skip writing a story for it:

- **Withdraw** — actor cancels/withdraws the case
- **Hold** — case is paused pending an external condition, with a defined resume trigger
- **Resume** — case returns to active processing from Hold
- **Skip** — actor bypasses a stage/step under defined conditions
- **Return / Go-back** — case is sent back to a prior stage/actor (e.g., rework, additional info
  needed)
- **Reassign** — case ownership moves to a different actor/queue
- **Merge** — two or more cases are combined
- **Duplicate detection** — system identifies/handles a likely-duplicate case
- **Dependency / relationship gating** — a case cannot proceed until a related case reaches a
  defined state
- **Exception processing** — generalized error/exception handling that isn't specific to one
  business rule (timeout, external-system failure, data-quality failure, manual-intervention
  routing)

Not every control applies to every case type — but the matrix forces an explicit yes/no per case
type instead of a silent default to "no." Record the "not applicable" ones as a one-line note on
the matrix row, not just an absence.

### 2b. Interface Inventory

Enumerate every system-to-system interface implied by the source artifacts as its own workbook tab,
**before** drafting any integration story. Columns: Interface ID, Source System, Target System,
Direction (inbound / outbound / bidirectional), Protocol (REST, SOAP, file, MQ, database, etc.),
Interchange/Message Count (how many distinct message types or call patterns this interface covers —
not "1" by default), Data Objects Exchanged, Error Handling Approach, System of Record. Every
integration-related story must reference an Interface Inventory row by Interface ID in its
Description — an integration story with no matching inventory row is a sign the inventory is
incomplete.

Do not size an integration as a routine story by default. Sizing must account for: number of data
objects and their complexity, synchronous vs. asynchronous, depth of error/retry/reconciliation
handling required, and whether this is a new connector/pattern or reuse of an existing one. A
same-system reuse of an already-built pattern can legitimately be small; a new external
integration with multiple data objects and custom error handling rarely is — don't default to 8
points because that's what integrations "usually" get sized at.

---

## 3. Epic / Feature / Story decomposition rules

Standard SDLC/SAFe-aligned hierarchy, applied so grouping is many-to-one at every level (never 1:1):

- **Epic** — a large business capability or initiative, typically spanning a release or program
  increment, delivering a coherent outcome a stakeholder would recognize by name (e.g., "Claims
  Intake Modernization"). An epic normally contains **3+ features**. If your draft produces an epic
  with only one feature, either the epic is too narrow (fold it into a sibling) or the feature is
  actually epic-sized (split it).
- **Feature** — a deliverable, demoable slice of the epic, sized to roughly one release/PI, usable
  end-to-end by a real actor (e.g., "Submit Claim via Web Portal"). A feature normally contains
  **2–8 stories**. Represent it as a Label/Component per §1.4 unless told otherwise.
- **Story** — a single unit of work satisfying **INVEST** (Independent, Negotiable, Valuable,
  Estimable, Small, Testable). Written as `As a <role>, I want <capability>, so that <benefit>`.

**A capability is not automatically one story.** If a capability from the source artifacts (e.g.,
"draft submissions and proposals") clearly represents a major, multi-part product capability rather
than a single unit of work, model it as its own **Feature** decomposed into multiple stories from
the start — don't draft one inflated story and rely on the size-gate in §5 to catch it later.
Treat "does this look like a whole feature, not a story?" as a question to ask *before* drafting,
not just a post-hoc split trigger.

**Grouping process:**
1. Extract every discrete requirement/capability/process step from all provided artifacts, cross-
   checked against the Case-Type Matrix and Interface Inventory from §2 so nothing enumerated there
   is silently dropped.
2. Cluster by business capability into candidate epics first (top-down), not by drafting stories and
   grouping upward — bottom-up grouping produces arbitrary, inconsistent epics.
3. Within each epic, cluster into features by actor journey or deliverable slice. Case-control
   capabilities (§2a) for a case type typically form their own feature (e.g., "Claim Case Lifecycle
   Controls") rather than being scattered as afterthoughts across unrelated features.
4. **Enumerate variant dimensions explicitly** before drafting stories within a feature: does this
   capability vary by standard/regulatory type, product/segment, actor role, review/approval path,
   or integration availability (e.g., behavior differs when an external system is unavailable)? If
   a meaningful variant exists along any of these dimensions, generate a distinct story (or an
   explicitly named story variant) per combination rather than one generic story that hides the
   variation — a generic story here is exactly what produces templated, uninformative acceptance
   criteria (§4).
5. Draft stories within each feature. Apply story-splitting patterns when a candidate story is too
   large or its AC would exceed 6 (see §4): workflow-step splits, business-rule/variation splits,
   CRUD-operation splits, interface/channel splits (web vs mobile vs API), happy-path-vs-exception
   splits.
6. Sanity-check the finished tree: every epic has ≥2 features (flag single-feature epics), every
   feature has ≥2 stories (flag single-story features), no epic or feature title duplicates one in
   the existing taxonomy from intake Q5, and every Case-Type Matrix / Interface Inventory row maps
   to at least one story.

---

## 4. Story metadata schema

Every story gets all of these fields. Field list is deliberately broader than "story number, title,
summary, description, AC, dependency, priority, design details, size, complexity" per the original
brief — the additions below are standard agile metadata commonly missing from ad-hoc backlogs, plus
fields added specifically to keep sizing and provenance auditable rather than templated (§9):

| Field | Definition |
|---|---|
| Story ID | Stable short ID, e.g. `EPC1-FT2-ST03`, used for dependency references before JIRA keys exist |
| Title | Short imperative summary, not the full user-story sentence |
| Epic | Parent epic name |
| Feature | Parent feature name (→ Label/Component on export) |
| Issue Type | `Story`, `Enabler` (infra/architecture/technical work with no direct end-user value — SAFe term), or `Spike` (time-boxed research when a story can't be sized yet) |
| Category | `Functional` (delivers a business capability) or `Foundation/Overhead` (environments, DevOps, testing infrastructure, hardening, contingency/buffer) — kept distinct from Issue Type so functional-value points and delivery-overhead points can be rolled up separately rather than conflated in one total |
| Summary | The `As a / I want / so that` statement |
| Description | Fuller narrative: context, business rule detail, links to source artifact section; integration stories must cite the relevant Interface Inventory ID, case-control stories must cite the relevant Case-Type Matrix row |
| Acceptance Criteria | 1–6 criteria (see §5 for format rules); if a draft needs more than 6, split the story instead. Must name the specific rule/screen/variant driving this story — see §5 for the anti-templating rule |
| Sizing Drivers | The concrete facts that produced this story's size: count of screens/views, business rules/decision variants, data elements/fields, integration touchpoints, and exception/alternate paths in scope. This is what makes a size defensible in grooming instead of a guess |
| Dependencies | Story IDs that must complete first; leave blank if none — don't invent dependencies for schedule convenience |
| Priority | Team's priority scale (state which you're using, e.g. MoSCoW or P1–P4); derived from source-artifact urgency signals plus AI judgment on what's load-bearing for the epic — state the reasoning briefly when priority isn't obvious from the source |
| Story Size | Fibonacci only: 1, 2, 3, 5, 8, 13. See §5 for the hard split-gate at 13 and why 21 is not a normal story size |
| Complexity | Low / Medium / High / Complex — independent of size; a small story can still be High complexity (e.g., a 2-point story touching a fragile legacy integration) |
| Design Details | How to implement, informed by the platform pattern file from intake Q1 |
| Actor / Persona | Primary role from the user-story sentence |
| Labels / Components | Feature label plus any module/team labels |
| Requirement Origin | `Sourced` (directly traceable to a specific location in a provided artifact) or `Inferred` (filled in by AI judgment because the source didn't specify it). Required on every story — see §6 |
| Source Reference | Which artifact + location this story traces to (transcript timestamp, requirement row, process-flow step, or Case-Type Matrix / Interface Inventory row ID). If Requirement Origin is `Inferred`, state what was inferred and why here instead of leaving this blank |
| Assumptions | Anything inferred rather than stated in source material |
| Out of Scope | Explicit non-goals, used to keep AC count down and prevent scope creep during grooming |
| Status | `Draft` until the user reviews it — never mark a generated story `Ready` |

---

## 5. Acceptance criteria rules — and the anti-templating rule

- 1–6 criteria per story. More than 6 means the story is doing too much — split it and re-derive AC
  per resulting story rather than trimming criteria to force-fit the limit.
- Choose format per story, not globally:
  - **Given/When/Then (Gherkin)** for behavior- or flow-driven stories (state transitions, multi-step
    interactions, anything a QA would want to automate).
  - **Bulleted "the system shall..." statements** for simple condition/validation stories (field
    validation, static display rules, simple CRUD).
  - Default to Gherkin when in doubt — it forces explicit preconditions, which catches ambiguity
    earlier.
- Every AC must be independently testable and unambiguous (no "the system should handle errors
  appropriately" — name the error and the expected behavior).
- **Do not reuse near-identical "created or updated / validated / audited" boilerplate across
  stories unless the underlying behavior is genuinely identical.** Generic AC is a symptom of a
  generic story — if every story's AC reads the same, go back and check whether the Sizing Drivers
  field is actually populated with real, story-specific facts (see §4). AC should make the specific
  rule, screen, variant, or exception path that this story covers legible to a reviewer who has
  never seen the source artifact.

---

## 6. Sizing, complexity, and the split gate

- **Fibonacci sizing** (1, 2, 3, 5, 8, 13) is *relative effort*, not days. Anchor it: a 1–2 is a
  well-understood, single-component change; 3–5 touches multiple components or has some unknowns; 8
  has real unknowns or multi-system coordination; 13 is the largest size a real story should carry.
- **13 is a hard split-gate, not a soft signal.** Any candidate story that would size at 13 must be
  re-evaluated against the story-splitting patterns in §3 step 5 *before* it is finalized — don't
  present a 13-point story without first attempting to split it and confirming the split isn't
  viable (e.g., truly atomic but broad work). If a capability keeps coming out at 13+ no matter how
  it's split, that's usually a sign it was never story-sized to begin with — go back to §3's
  "is this actually a Feature" check.
- **21 is not a normal story size.** Reserve it only for a `Spike` (time-boxed research) where the
  work is explicitly not yet estimable as delivery work. A `Story` or `Enabler` sized at 21 should
  always be split instead.
- **Complexity** (Low/Medium/High/Complex) is orthogonal to size — it should reflect technical risk
  (fragile integration, unclear requirements, new-to-team tech, regulatory sensitivity), not just
  effort. State briefly why a story is High/Complex when it isn't obvious, so the reviewer isn't
  left guessing.
- **Calibrate against historical actuals when available**, per intake Q8. If
  [reference/sizing_calibration_template.md](reference/sizing_calibration_template.md) (or a
  project-specific filled-in version of it) exists, match each story's Sizing Drivers against the
  closest historical archetype row and use its actual hours/points as a sanity check on the
  generated size — don't apply a flat, generic points-to-hours conversion factor across every story
  regardless of what it actually involves. If no calibration reference exists, size from first
  principles and say so.

---

## 7. Output format

Produce **two artifacts**, not one:

### A. Review workbook (Excel, primary editable format)
Tabs: **Case-Type Matrix**, **Interface Inventory** (both from §2, built before stories), **Epics**,
**Features**, **Stories** (all metadata from §4 as columns, with data-validation dropdowns on Issue
Type / Category / Priority / Story Size / Complexity / Requirement Origin / Status), **Traceability**
(story → source artifact mapping), **Open Questions** (anything flagged during generation, with a
Status column the team updates as they resolve items — inferred/assumed items belong here even if
they're also noted on the story itself).

### B. JIRA import file (CSV), generated from the approved workbook
Follow [reference/jira_csv_mapping.md](reference/jira_csv_mapping.md) for the generic field mapping.
Always tell the user this mapping uses standard/generic JIRA field names and must be checked against
their actual project's field scheme (custom field IDs in particular) before a real import — don't
imply it's guaranteed to import cleanly without that check.

Do not generate the JIRA CSV until the user has reviewed and approved the Excel workbook — the CSV is
a mechanical transform of the approved draft, not a separate creative pass.

---

## 8. New build vs. enhancement — foundational stories

- **New build**: include foundational/enabler stories for things the platform needs before feature
  work can start (environment/instance setup, base data model, core integration scaffolding, security
  baseline, CI/CD pipeline if in scope). Group these under a dedicated "Platform Foundation" epic
  unless the user has an existing epic for this. Tag these `Category: Foundation/Overhead`.
- **Enhancement**: assume the foundation exists. Only add foundational/enabler stories for the
  *specific* new capability being added (e.g., a new integration endpoint), never for things the base
  platform already provides. If it's unclear whether a foundational piece already exists, ask rather
  than assuming either way — this is one of the more expensive wrong guesses to make.
- Delivery overhead that isn't platform-foundational but still isn't business-functional — dedicated
  testing/hardening passes, environment promotion, contingency/buffer work — should also be tagged
  `Category: Foundation/Overhead` so it doesn't get silently counted as functional capacity in
  rollups.

---

## 9. Pre-return quality gate

Before presenting the backlog, self-check:
- [ ] Case-Type Matrix and Interface Inventory were built before stories were drafted, and every
      story traces to a row in one of them (or is explicitly Foundation/Overhead)
- [ ] Every case type in the matrix was evaluated against the case-control checklist in §2a, with a
      story or an explicit "not applicable" note for each control
- [ ] Every epic has ≥2 features; every feature has ≥2 stories (or a stated reason it's an exception)
- [ ] No story has >6 AC
- [ ] No story is sized at 21; every story sized at 13 was actually run through the split check in
      §6, not just left there by default
- [ ] Acceptance criteria are not copy-pasted boilerplate across unrelated stories — spot-check a
      handful for story-specific content
- [ ] Sizing Drivers is populated with real counts (screens, rules, data elements, integrations,
      exception paths), not left blank or vague
- [ ] Every story has Priority, Complexity, Category, Design Details, Requirement Origin, and Source
      Reference populated (not left blank)
- [ ] Dependencies reference only Story IDs that exist in this backlog
- [ ] Open Questions tab captures every assumption made during generation, not just the ones the user
      is likely to notice
- [ ] JIRA CSV (if generated) matches the approved Excel content exactly — no new stories introduced
      during export

This gate exists because this skill's output was benchmarked once against manually-elaborated
requirements and found to under-size large capabilities, template its acceptance criteria, under-
model integrations, and drop cross-cutting case controls. §2, §4's Sizing Drivers/Requirement Origin
fields, §5's anti-templating rule, and §6's hard split-gate exist specifically to close those gaps —
don't let generation drift back to skipping them under time pressure.
