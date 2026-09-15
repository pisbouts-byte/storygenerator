# Setup guide — Agile Backlog Builder as a Microsoft 365 Copilot agent

This turns the same rules used by the Claude Code skill (`.claude/skills/agile-backlog-builder/`)
and the Copilot-in-VS-Code version (`.github/copilot-instructions.md`) into a declarative agent
inside Microsoft 365 Copilot. Someone with a Copilot Studio maker role (or, if your tenant has it
enabled, the simpler in-Copilot-Chat "Create an agent" builder) needs to do this — it can't be done
by editing files in this repo, since M365 Copilot doesn't read GitHub repos.

Exact menu wording shifts between Microsoft releases — if a label below doesn't match exactly what
you see, look for the closest equivalent (e.g. "Knowledge" vs. "Add knowledge source").

## Which builder to use

- **In-Copilot-Chat Agent Builder** (open Copilot Chat → look for "Create an agent" / "Agent
  Builder") — simplest, usually needs no extra license beyond your existing Copilot seat, good
  enough for this use case since it's instructions + knowledge files with no custom actions/APIs.
- **Copilot Studio** (make.powerva.microsoft.com or via the Copilot Studio app) — more powerful
  (analytics, publishing controls, custom topics/actions), needs a Copilot Studio license/role.
  Use this if IT wants central governance over who can publish/update the agent.

Either one accepts the same three ingredients below.

## 1. Name and description

- **Name**: `Agile Backlog Builder`
- **Description**: `Converts requirements artifacts (Excel, meeting transcripts, process-flow
  diagrams, PDFs, Word docs) into a JIRA-importable agile backlog of epics, features, and user
  stories with full metadata.`

## 2. Instructions

Open `agent-instructions.md` in this folder and paste its content (everything below the `---`
line) into the agent's **Instructions** field.

## 3. Knowledge sources

Upload these three files from the `knowledge/` folder in this same directory as knowledge sources:
- `jira_csv_mapping.txt`
- `platform_patterns_pega.txt`
- `backlog_template.xlsx` (lets the agent describe the expected output format/columns, even though
  it can't fill in the live workbook the way Copilot *inside* an open Excel file can)

These are plain `.txt` rather than `.md` — Copilot Studio / Agent Builder's file-upload knowledge
sources don't accept raw Markdown. `.txt` keeps the same content and indexes fine even with
Markdown syntax (headers, tables) still visible in the text.

If your tenant only allows SharePoint/OneDrive knowledge sources rather than direct file upload,
put these three files in a shared SharePoint folder first and point the agent at that folder
instead.

## 4. Conversation starters

Add the four prompts from `conversation-starters.md` in this folder as suggested starter prompts.

## 5. Test before publishing

In the builder's test pane, try:
> "Build an agile backlog from this transcript. Platform is Pega Constellation/Infinity 24.1,
> on-prem. This is a new build." (attach or paste a short sample transcript)

Confirm it: asks for the platform/new-build/enhancement details if you didn't supply them, groups
output into epics → features → stories, keeps acceptance criteria to 1–6 per story, uses Fibonacci
sizing, and doesn't offer a JIRA export before you've said you approved the draft.

## 6. Publish and share

Publish the agent per your tenant's normal process, then share it with the team (via Teams, or
however your org distributes Copilot agents) so people don't need to build their own copy.

## Known limitation vs. Claude Code

The Claude Code skill can generate the actual formatted `.xlsx` workbook (tabs, dropdowns,
data validation) as a file. This M365 agent can only produce the backlog as chat text/tables —
if the user is working directly inside an open Excel file, Excel's own Copilot can help transcribe
the draft into `backlog_template.xlsx`'s tabs, but the agent itself won't generate the file.

## Keeping this in sync

If `.claude/skills/agile-backlog-builder/` or `.github/copilot-instructions.md` are updated later,
this folder's `agent-instructions.md` and the copied knowledge files need to be updated to match —
none of these three versions sync automatically.
