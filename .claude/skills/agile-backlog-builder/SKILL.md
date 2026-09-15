---
name: agile-backlog-builder
description: Converts mixed source artifacts (Excel requirements, meeting transcripts, process-flow diagrams, PDFs, Word docs) into a JIRA-importable agile backlog of epics, features, and user stories with full metadata. Use when asked to draft a backlog, write user stories, decompose requirements/transcripts/process flows into epics and stories, or prepare a backlog for JIRA import.
---

# Agile Backlog Builder

Turns raw requirements artifacts into a review-ready, JIRA-importable backlog. Optimized for
consistency across a team working on multiple delivery platforms (Pega, Camunda, custom code, etc.).

This skill has two halves:
1. **Intake** — ask the questions that change the shape of the output, before drafting anything.
2. **Generation** — apply epic/feature/story decomposition rules, populate full metadata, validate
   against hard constraints, and emit both a review workbook and a JIRA import file.

Do not skip straight to drafting stories. A backlog built before intake questions are answered will
have wrong sizing assumptions, missing foundational stories, or a JIRA export that doesn't match the
target project's schema.

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
   /enabler stories (see §4) that enhancements typically don't need — don't generate foundational
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

**Mid-generation clarifying questions:** if source artifacts conflict, a process-flow step has no
corresponding requirement text, or a requirement is too vague to size, do not guess silently — list
these as flagged items in the **Open Questions** tab (see §6) AND, if the ambiguity is large enough to
change epic/feature grouping, ask the user directly before continuing.

---

## 2. Epic / Feature / Story decomposition rules

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

**Grouping process:**
1. Extract every discrete requirement/capability/process step from all provided artifacts.
2. Cluster by business capability into candidate epics first (top-down), not by drafting stories and
   grouping upward — bottom-up grouping produces arbitrary, inconsistent epics.
3. Within each epic, cluster into features by actor journey or deliverable slice.
4. Draft stories within each feature. Apply story-splitting patterns when a candidate story is too
   large or its AC would exceed 6 (see §3): workflow-step splits, business-rule/variation splits,
   CRUD-operation splits, interface/channel splits (web vs mobile vs API), happy-path-vs-exception
   splits.
5. Sanity-check the finished tree: every epic has ≥2 features (flag single-feature epics), every
   feature has ≥2 stories (flag single-story features), no epic or feature title duplicates one in
   the existing taxonomy from intake Q5.

---

## 3. Story metadata schema

Every story gets all of these fields. Field list is deliberately broader than "story number, title,
summary, description, AC, dependency, priority, design details, size, complexity" per the original
brief — the additions below are standard agile metadata commonly missing from ad-hoc backlogs and are
needed for a clean JIRA import and later grooming:

| Field | Definition |
|---|---|
| Story ID | Stable short ID, e.g. `EPC1-FT2-ST03`, used for dependency references before JIRA keys exist |
| Title | Short imperative summary, not the full user-story sentence |
| Epic | Parent epic name |
| Feature | Parent feature name (→ Label/Component on export) |
| Issue Type | `Story`, `Enabler` (infra/architecture/technical work with no direct end-user value — SAFe term), or `Spike` (time-boxed research when a story can't be sized yet) |
| Summary | The `As a / I want / so that` statement |
| Description | Fuller narrative: context, business rule detail, links to source artifact section |
| Acceptance Criteria | 1–6 criteria (see §4 for format rules); if a draft needs more than 6, split the story instead |
| Dependencies | Story IDs that must complete first; leave blank if none — don't invent dependencies for schedule convenience |
| Priority | Team's priority scale (state which you're using, e.g. MoSCoW or P1–P4); derived from source-artifact urgency signals plus AI judgment on what's load-bearing for the epic — state the reasoning briefly when priority isn't obvious from the source |
| Story Size | Fibonacci only: 1, 2, 3, 5, 8, 13, 21. If a story feels bigger than 13, it should almost always be split — flag it rather than sizing it |
| Complexity | Low / Medium / High / Complex — independent of size; a small story can still be High complexity (e.g., a 2-point story touching a fragile legacy integration) |
| Design Details | How to implement, informed by the platform pattern file from intake Q1 |
| Actor / Persona | Primary role from the user-story sentence |
| Labels / Components | Feature label plus any module/team labels |
| Source Reference | Which artifact + location this story traces to (transcript timestamp, requirement row, process-flow step) |
| Assumptions | Anything inferred rather than stated in source material |
| Out of Scope | Explicit non-goals, used to keep AC count down and prevent scope creep during grooming |
| Status | `Draft` until the user reviews it — never mark a generated story `Ready` |

---

## 4. Acceptance criteria rules

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

---

## 5. Sizing and complexity guidance

- **Fibonacci sizing** (1, 2, 3, 5, 8, 13, 21) is *relative effort*, not days. Anchor it: a 1–2 is a
  well-understood, single-component change; 3–5 touches multiple components or has some unknowns; 8
  has real unknowns or multi-system coordination; 13+ is a signal to split before grooming.
- **Complexity** (Low/Medium/High/Complex) is orthogonal — it should reflect technical risk (fragile
  integration, unclear requirements, new-to-team tech, regulatory sensitivity), not just size. State
  briefly why a story is High/Complex when it isn't obvious, so the reviewer isn't left guessing.

---

## 6. Output format

Produce **two artifacts**, not one:

### A. Review workbook (Excel, primary editable format)
Tabs: **Epics**, **Features**, **Stories** (all metadata from §3 as columns, with data-validation
dropdowns on Issue Type / Priority / Story Size / Complexity / Status), **Traceability** (story →
source artifact mapping), **Open Questions** (anything flagged during generation, with a Status column
the team updates as they resolve items).

### B. JIRA import file (CSV), generated from the approved workbook
Follow [reference/jira_csv_mapping.md](reference/jira_csv_mapping.md) for the generic field mapping.
Always tell the user this mapping uses standard/generic JIRA field names and must be checked against
their actual project's field scheme (custom field IDs in particular) before a real import — don't
imply it's guaranteed to import cleanly without that check.

Do not generate the JIRA CSV until the user has reviewed and approved the Excel workbook — the CSV is
a mechanical transform of the approved draft, not a separate creative pass.

---

## 7. New build vs. enhancement — foundational stories

- **New build**: include foundational/enabler stories for things the platform needs before feature
  work can start (environment/instance setup, base data model, core integration scaffolding, security
  baseline, CI/CD pipeline if in scope). Group these under a dedicated "Platform Foundation" epic
  unless the user has an existing epic for this.
- **Enhancement**: assume the foundation exists. Only add foundational/enabler stories for the
  *specific* new capability being added (e.g., a new integration endpoint), never for things the base
  platform already provides. If it's unclear whether a foundational piece already exists, ask rather
  than assuming either way — this is one of the more expensive wrong guesses to make.

---

## 8. Pre-return quality gate

Before presenting the backlog, self-check:
- [ ] Every epic has ≥2 features; every feature has ≥2 stories (or a stated reason it's an exception)
- [ ] No story has >6 AC
- [ ] Every Story Size is a Fibonacci number
- [ ] Every story has Priority, Complexity, Design Details, and Source Reference populated (not left
      blank)
- [ ] Dependencies reference only Story IDs that exist in this backlog
- [ ] Open Questions tab captures every assumption made during generation, not just the ones the user
      is likely to notice
- [ ] JIRA CSV (if generated) matches the approved Excel content exactly — no new stories introduced
      during export
