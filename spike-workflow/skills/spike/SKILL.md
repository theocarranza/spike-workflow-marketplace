---
name: spike
description: >
  Run a structured Agile/XP spike investigation from start to finish. Use this skill whenever
  the user provides a spike question or asks Claude to "do a spike on X", "investigate X",
  "explore whether X is feasible", or "run a spike". Also trigger when the user mentions
  spike types (technical spike, functional spike, risk spike) or says things like
  "we need to validate before committing", "time-boxed research", or "proof of concept for X".
  The skill produces all required spike deliverables: findings document, recommendation,
  refined estimates, and a functional archived prototype where applicable.
  Sub-commands: spike run, spike resume, spike rewrite, spike wikigen, spike add-context, spike publish.
disable-model-invocation: true
allowed-tools: >
  Read Write Edit Glob
  Agent Task
  WebFetch WebSearch
  Bash(git init *) Bash(git add *) Bash(git commit *) Bash(git remote add *)
  Bash(git push *) Bash(git branch *) Bash(git log *) Bash(git show *)
  Bash(git rev-parse *) Bash(gh repo create *) Bash(gh repo view *)
  Bash(flutter test *) Bash(fvm flutter test *) Bash(flutter pub get *)
  Bash(dart analyze *) Bash(npm test *) Bash(npm install *)
  Bash(pytest *) Bash(pip install *)
  Bash(mkdir *) Bash(date *) Bash(maestro *) Bash(adb *)
---

# Spike Skill — v0.2

Executes a full spike investigation and produces all required deliverables in compliance
with the accepted Agile/XP definition of a spike. Each phase writes a checkpoint to
`.spike-state.json` before advancing so interrupted sessions can resume without loss.

---

## Sub-commands

If the invocation begins with one of these keywords, jump directly to that section:

| Invocation | Section |
|---|---|
| `spike resume <path>` | [§ Resume](#resume) |
| `spike run <path>` | [§ Run](#run) |
| `spike rewrite <path>` | [§ Rewrite](#rewrite) |
| `spike wikigen <path>` | [§ Wikigen](#wikigen) |
| `spike add-context <path> <input>` | [§ Add Context](#add-context) |
| `spike publish <path>` | [§ Publish](#publish) |

If no sub-command is given, run the full spike workflow starting at Phase 0.

---

## Inputs (full spike)

Collect before starting. If any required input is missing, ask before proceeding.

| Input | Required | Description |
|---|---|---|
| `question` | ✅ | The single question the spike must answer |
| `type` | ✅ | `technical` \| `functional` \| `risk` |
| `timebox` | ✅ | Duration agreed by the team (e.g. "2 days", "1 sprint") |
| `context` | ✅ | Project, stack, and constraints relevant to the investigation |
| `code_context` | Optional | Relevant code snippets: existing implementations, interfaces, state shapes, or patterns the spike must be compatible with |

If `type` is not given, infer it from the question and confirm with the user.
If `timebox` is not given, default to **1 day** and state this assumption explicitly.

**When `code_context` is provided:**
- The prototype must be structurally compatible with the patterns shown.
- Analyse it before investigating externally — identify patterns, constraints, and conventions.
- Reference it explicitly in the findings document under "Approach."

---

## Output Destinations

| Deliverable | Destination |
|---|---|
| Findings document | `FINDINGS.md` in the spike folder (local); published to Azure DevOps Wiki via `spike publish` |
| Recommendation | Inline in `FINDINGS.md` — stakeholder paragraph FIRST, technical table second |
| Refined estimate | Inline in `FINDINGS.md` |
| Prototype | `spike-<slug>/` folder — archived, never merged |
| State log | `spike-<slug>/.spike-state.json` — written at each phase boundary |

---

## Research Configuration

The research subagent's model, thinking, and effort level are configurable per spike.
Set them in the Spike Brief (Phase 0) or override them at any time via `spike run`.

| Setting | Default | Options |
|---|---|---|
| `model` | `claude-opus-4-7` | `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5` |
| `thinking` | `enabled` | `enabled`, `disabled` |
| `thinking_budget_tokens` | `8000` | Any integer; raise to `16000` for high-ambiguity questions |
| `effort` | `thorough` | `thorough` (≥5 searches, full docs), `standard` (3–4 searches), `quick` (1–2 searches) |

These values are stored in `.spike-state.json` under `research_config` and applied when the
researcher subagent is spawned. They can be changed between runs without restarting the spike.

See `references/researcher-prompt.md` for full effort-level definitions.

---

## Rules (enforce strictly)

1. **One question per spike.** Decompose multiple unknowns into separate spikes; ask which to run first.
2. **Time-boxed.** Acknowledge the timebox at the start. Do not expand scope.
3. **The prototype is functional.** It must run. It must pass its own tests. Do not present a broken prototype.
4. **The prototype is archived, not merged.** State this explicitly in all outputs.
5. **Definition of Done = question answered.** The spike ends when the original question has a documented, evidenced answer.
6. **No estimation of unrelated work.** Only estimate the task that directly follows from this spike's findings.
7. **Recommendations must be substantiated.** A Conditional GO whose condition was not tested within the timebox must carry the label `UNVALIDATED — follow-on required`. A bare conditional without this label is a gate failure.
8. **Stakeholder paragraph precedes technical table.** Any output that leads with the technical GO/NO-GO table instead of the stakeholder paragraph is malformed and must be corrected before closing.

---

## Execution Phases

### Phase 0 — Frame the Spike *(hard gate — Phase 1 cannot start until confirmed)*

Output a **Spike Brief** and ask the user to confirm before proceeding:

```
## Spike Brief
- **Question:** <exact question being answered>
- **Type:** technical | functional | risk
- **Timebox:** <agreed duration>
- **Success criteria:** <concrete, testable conditions — how we will know the question is answered>
- **Out of scope:** <explicit exclusions>
- **Research config:** model=<default claude-opus-4-7> · effort=<default thorough> · thinking=<default enabled>
```

The **Research config** line is optional — if omitted, the defaults from the Research Configuration
section apply. Write the resolved values to `.spike-state.json` under `research_config` on confirmation.

**Success criteria must be specific and testable.** "Question answered" is not sufficient.
Examples of good success criteria:
- "Login flow runs end-to-end on Android against the local emulator without manual intervention."
- "The CI job boots an emulator and executes the login flow in under 10 minutes on a standard runner."

Write Phase 0 checkpoint to `.spike-state.json` after confirmation.

---

### Phase 1 — Investigate

Phase 1 is owned by the **researcher subagent**. Read `references/researcher-prompt.md` to
understand the researcher's full protocol, output format, and effort levels.

**Spawn the researcher subagent with this brief:**

```
You are a technical researcher. Read references/researcher-prompt.md for your full instructions.

Your inputs:
- spike_brief: [paste confirmed Spike Brief from Phase 0]
- reference_file: [paste full content of references/technical.md OR functional.md OR risk.md]
- code_context: [paste code_context if provided, or "None"]
- research_config: [paste spike_state.research_config]
- existing_findings: [paste spike_state.phases["1"].findings if resuming, or "None"]

Return the JSON object specified in references/researcher-prompt.md. Nothing else.
```

**On receiving the researcher's JSON output:**

1. Validate that `tool_surface.gaps` is empty or that each gap is explicitly listed in `unresolved`.
2. If `ci_checkpoint_fired: true` and `ci_findings` is null — this is an error. Re-spawn with a note to complete the CI investigation.
3. Write Phase 1 checkpoint to `.spike-state.json`:
   - `phases["1"].findings` ← researcher's `findings` array (labels only, for state log brevity)
   - `phases["1"].dead_ends` ← researcher's `dead_ends` array
   - `phases["1"].tool_surface` ← researcher's `tool_surface`
   - `phases["1"].ci_checkpoint_fired` ← researcher's `ci_checkpoint_fired`
   - `phases["1"].recommendation_draft` ← researcher's `recommendation_draft`
   - `phases["1"].unresolved` ← researcher's `unresolved`
   - `phases["1"].status` ← `"complete"`
4. Store the full researcher JSON in `.spike-research-raw.json` alongside the state log.
   This is the source of truth for Phase 3 input assembly.

---

### Phase 2 — Build Prototype (if applicable)

Required when:
- Spike type is **technical** and the question requires code to answer.
- Spike type is **risk** and feasibility must be demonstrated, not just argued.

Read `references/technical.md` § Implementation Notes before building.

If building a prototype:
1. Build the minimal version that answers the question — no extras.
2. Structure as a proper project (layout, dependency manifest, `.gitignore`).
3. Match `code_context` patterns if provided.
4. Must run from a clean environment — no hidden setup, no broken imports.
5. Include tests scoped to proving the investigated mechanism works.
6. Include `README.md`: what it demonstrates, prerequisites, run instructions, expected output, scope exclusions.
7. Mark all source files: `// SPIKE PROTOTYPE — ARCHIVED — NOT FOR PRODUCTION`
8. Package as `spike-<slug>/`.
9. Include a `/ci` folder with a pipeline stub if the CI/CD checkpoint (Phase 1) fired.

**Phase 2 gate — two checks, both required:**

**Gate A — Execution:**
- [ ] Every flow/test claimed as PASS has been run live and output matches README.
- [ ] Written-not-executed paths are labelled ⚠️ and excluded from default runs.
- [ ] If failing: fix it. Do not advance with a broken prototype.

**Gate B — Substantiation:**
- [ ] The specific claim the recommendation rests on has been demonstrated, not just argued.
- [ ] If the recommendation is conditional, go to the Validate Recommendation step below before advancing.

**Validate Recommendation (mandatory if recommendation is conditional):**
> Identify the condition. Attempt to exercise it empirically within the remaining timebox.
> - If exercised and passes: label the recommendation `VALIDATED`.
> - If exercised and fails: update the recommendation accordingly.
> - If cannot be exercised (time/environment constraint): label it `UNVALIDATED — follow-on required`
>   and state the reason. Do NOT issue a bare Conditional GO.

Write Phase 2 checkpoint to `.spike-state.json` after gates pass.

**GitHub publish (pause — wait for user approval):**
Do not push to GitHub automatically. Write a summary of what will be pushed and ask for approval.
Once approved:
1. `git init` inside the prototype folder
2. `git branch -M main`
3. `git add . && git commit -m "chore: spike prototype — archived, not for production"`
4. `gh repo create spike-<slug> --private --source=. --remote=origin --push`
5. Record URL in `.spike-state.json`.

---

### Phase 3 — Produce Deliverables

This phase uses two specialised subagents in sequence: a **writer** that drafts `FINDINGS.md`
from the investigation inputs, and a **proofreader** that audits the draft against the rules.
The writer self-corrects on all fixable issues. Only unresolved blockers are escalated to the user.

---

#### Step 3.1 — Assemble inputs for the writer

Collect the following from the state log and Phase 1–2 outputs:

| Input | Source |
|---|---|
| `spike_state` | `.spike-state.json` (full JSON) |
| `findings_list` | `phases["1"].findings` from state log |
| `prototype_summary` | What passed, what was written-not-executed, prototype path, GitHub URL |
| `estimate_table` | Markdown table of work items, estimates, confidence, assumptions |
| `parking_lot` | `parking_lot` array from state log |
| `evidence_list` | Each file in `evidence/` with a one-sentence description |

---

#### Step 3.2 — Spawn writer subagent

Spawn a subagent with the following brief (fill in the bracketed values):

```
You are a technical writer. Read references/writer-prompt.md for your full instructions,
then read references/example-findings.md as your quality standard.

Your inputs:
- spike_state: [paste full .spike-state.json content]
- findings_list: [paste phases["1"].findings]
- prototype_summary: [paste summary]
- estimate_table: [paste table]
- parking_lot: [paste list]
- evidence_list: [paste list]

Produce a complete FINDINGS.md. Return it as a fenced markdown block followed by
a one-line summary of any [MISSING] placeholders you inserted.
```

---

#### Step 3.3 — Spawn proofreader subagent

Pass the writer's draft to the proofreader:

```
You are a technical editor. Read references/proofreader-prompt.md for your full instructions,
then read references/example-findings.md as your quality standard.

Your inputs:
- draft: [paste writer's FINDINGS.md output]
- spike_state: [paste full .spike-state.json content]
- missing_list: [paste writer's missing-placeholder summary]

Run every checklist item. Return a JSON object in the exact format specified in
references/proofreader-prompt.md.
```

---

#### Step 3.4 — Writer self-correction loop

1. Read the proofreader's JSON output.
2. For each issue where `requires_user_input: false`: apply the fix directly to the draft.
3. Re-run the proofreader on the corrected draft.
4. Repeat until no `requires_user_input: false` issues remain.

**If `unresolved` is empty after correction:** proceed to Step 3.5.
**If `unresolved` contains only `warning` items:** proceed to Step 3.5; note warnings to the user as non-blocking.
**If `unresolved` contains any `blocker` item with `requires_user_input: true`:** stop. Present the unresolved blockers to the user with the quoted evidence and ask for clarification before continuing.

---

#### Step 3.5 — Write the final document

Write the corrected FINDINGS.md to the spike folder.
Write Phase 3 checkpoint to `.spike-state.json` with `findings_doc_path`.

---

#### Recommendation rules (enforced by proofreader — do not skip)

- Stakeholder paragraph BEFORE the technical table. Always. No exception.
- Stakeholder paragraph: plain language, max 5 sentences, zero code or jargon.
- Technical decision label must match `spike_state.phases["2"].recommendation_validation` exactly:
  `GO ✅` / `NO-GO ❌` / `CONDITIONAL GO VALIDATED ✅` / `CONDITIONAL GO UNVALIDATED ⚠️ — follow-on required`
- Never "it depends" — commit with documented rationale.

---

### Phase 4 — Close the Spike

Verify the success criteria registered in Phase 0 are met. Echo each criterion and its status.

```
## Spike Closed
- **Question answered:** Yes / Partially / No
- **Success criteria met:** <list each criterion and Yes/Partially/No>
- **Recommendation validation:** VALIDATED | UNVALIDATED — follow-on required | N/A
- **If partially/no:** reason and recommended follow-up spike
- **Prototype archived:** Yes / N/A
- **Next action:** <specific next step for the team>
```

Write final checkpoint to `.spike-state.json` with status `closed`.

---

## What NOT to do

- Do not write production code during a spike.
- Do not merge or propose merging prototype code.
- Do not present a prototype that does not run or whose tests fail.
- Do not leave the original question unanswered without explicitly flagging it.
- Do not expand scope — log tangential findings as Parking Lot items.
- Do not skip the Spike Brief confirmation step.
- Do not produce a recommendation without the stakeholder paragraph first.
- Do not issue a bare Conditional GO — it must be VALIDATED or UNVALIDATED.
- Do not push to GitHub or publish to the ADO wiki without user approval.

---

## Parking Lot

Log important unknowns and tangential questions here. These become candidates for future spikes or tickets.

---

## State Log Schema

Each phase boundary writes `.spike-state.json` in the spike folder:

```json
{
  "spike_id": "string",
  "question": "string",
  "type": "technical | functional | risk",
  "timebox": "string",
  "success_criteria": ["string"],
  "status": "phase-0 | phase-1 | phase-2 | phase-3 | phase-4 | closed",
  "research_config": {
    "model": "claude-opus-4-7 | claude-sonnet-4-6 | claude-haiku-4-5",
    "thinking": "enabled | disabled",
    "thinking_budget_tokens": 8000,
    "effort": "thorough | standard | quick"
  },
  "phases": {
    "0": { "status": "complete | skipped", "brief_confirmed": true, "timestamp": "ISO8601" },
    "1": {
      "status": "complete | in-progress",
      "findings": ["F1 title", "F2 title"],
      "dead_ends": [],
      "tool_surface": {},
      "ci_checkpoint_fired": false,
      "recommendation_draft": {},
      "unresolved": [],
      "timestamp": "ISO8601"
    },
    "2": { "status": "complete | in-progress | skipped", "flows_run": [], "flows_blocked": [], "recommendation_validation": "VALIDATED | UNVALIDATED | N/A", "timestamp": "ISO8601" },
    "3": { "status": "complete | in-progress", "recommendation": "GO | NO-GO | CONDITIONAL-GO-VALIDATED | CONDITIONAL-GO-UNVALIDATED", "timestamp": "ISO8601" }
  },
  "prototype_path": "string | null",
  "ci_stub_path": "string | null",
  "findings_doc_path": "string | null",
  "github_repo_url": "string | null",
  "wiki_page_url": "string | null",
  "parking_lot": ["string"]
}
```

Raw researcher output is stored separately in `.spike-research-raw.json` (same folder).
This file is the authoritative source for Phase 3 input assembly and for resuming research.

---

## Resume

`spike resume <path-to-spike-folder>`

1. Read `<path>/.spike-state.json`. If it does not exist, offer to initialize a new state log.
2. Report: spike ID, question, current phase, summary of completed phases and their key findings.
3. Ask: continue from the current phase, or re-run a specific phase?
4. On confirmation, continue per the relevant phase instructions above.

---

## Run

`spike run <path-to-spike-folder> [--model <model>] [--effort <level>] [--thinking <enabled|disabled>] [--thinking-budget <tokens>]`

Triggers research and findings generation in one command. If the spike folder already exists,
it composes — it does not overwrite. Use this as the primary driver after the Spike Brief
is confirmed. Also use it to refresh an existing spike after new information becomes available.

**Optional flags** (all override `research_config` in `.spike-state.json` for this run only):

| Flag | Values | Default |
|---|---|---|
| `--model` | `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5` | from `research_config` |
| `--effort` | `thorough`, `standard`, `quick` | from `research_config` |
| `--thinking` | `enabled`, `disabled` | from `research_config` |
| `--thinking-budget` | integer (tokens) | from `research_config` |

**Steps:**

**1. Resolve state**

- If `.spike-state.json` does not exist: this is a first run. Require that Phase 0 (Spike Brief) has been confirmed before proceeding. If not confirmed, jump to Phase 0.
- If `.spike-state.json` exists: this is a compose run. Read the existing state. Report what is already known (existing findings, current recommendation, FINDINGS.md status). Ask the user: "Run fresh research and compose with existing findings?" before proceeding.

**2. Apply flag overrides**

Merge any `--flag` values into a `run_config` object. This does not permanently modify `research_config` in the state log — it applies only to this invocation.

**3. Spawn researcher subagent**

Spawn the researcher per Phase 1 instructions, passing:
- `existing_findings`: the current `phases["1"].findings` from state log (or `"None"` on first run).
- `research_config`: `run_config` (flag overrides merged over stored config).

The researcher returns its JSON report. On a compose run, it continues the findings sequence
from the last existing label and marks any revised findings with a `-revised` suffix.

**4. Diff and confirm**

Before writing anything, present the user with a structured summary:

```
## Research complete — proposed changes

New findings:     F4, F5, F6
Revised findings: F2-revised (updated rationale)
Dead ends added:  2
Parking lot:      3 new items
Recommendation:   [old] → [new] (if changed)

Proceed? [y/n]
```

Do not modify any files until the user confirms.

**5. Merge and write state**

On confirmation:
- Merge new and revised findings into the state log.
- Write updated `.spike-research-raw.json` (append, do not replace existing entries).
- Update `.spike-state.json` Phase 1 checkpoint.

**6. Run writer→proofreader pipeline**

Assemble Phase 3 inputs using the merged research (Phase 3 Step 3.1).

If `FINDINGS.md` already exists:
- Pass the existing document to the writer as `existing_document` (same as `spike rewrite`).
- The writer improves and extends it rather than regenerating from scratch.

If `FINDINGS.md` does not exist:
- Run the full Phase 3 pipeline from scratch.

Run the proofreader. Self-correct. Escalate unresolved blockers to user if any.

**7. Write output**

- Back up any existing `FINDINGS.md` as `FINDINGS.md.bak`.
- Write the new `FINDINGS.md`.
- Update `.spike-state.json` Phase 3 checkpoint.
- Report: what changed, path to new document, any unresolved warnings.

---

## Rewrite

`spike rewrite <path-to-spike-folder>`

Rewrites the existing `FINDINGS.md` as a publication-ready document by running it through
the writer→proofreader pipeline as an editorial pass. Use this when:
- The document exists but was written without the writer/proofreader agents (e.g. produced by an earlier version of the plugin or written manually).
- The document needs a quality pass after `spike add-context` made structural changes.
- The user explicitly asks to improve, rewrite, or clean up the findings document.

**Difference from `wikigen`:** `wikigen` regenerates from scratch using the raw state log inputs.
`rewrite` takes the existing `FINDINGS.md` as its primary source of truth and improves it — it
does not discard content, it restructures and edits it.

**Steps:**

1. Read `.spike-state.json`. If it does not exist, create a minimal one from the FINDINGS.md content (extract spike ID, question, and status).
2. Read the existing `FINDINGS.md` in full.
3. Pass the document to the writer subagent with this brief:

```
You are a technical writer. Read references/writer-prompt.md for your full instructions,
then read references/example-findings.md as your quality standard.

You are rewriting an existing FINDINGS.md, not producing one from scratch.
Preserve all factual content. Your job is to restructure, deduplicate, and
rewrite prose to comply with the rules — not to invent or remove facts.

Your inputs:
- existing_document: [paste full current FINDINGS.md content]
- spike_state: [paste .spike-state.json content]

Apply every rule from references/writer-prompt.md to the existing document.
Return the rewritten document as a fenced markdown block followed by a one-line
summary of any [MISSING] placeholders you inserted.
```

4. Run the proofreader on the writer's output (Phase 3 Steps 3.3–3.4).
5. On pass: back up the original as `FINDINGS.md.bak`, write the new version to `FINDINGS.md`, report the path and a one-line summary of what changed.
6. Update `findings_doc_path` in `.spike-state.json`.

---

## Wikigen

`spike wikigen <path-to-spike-folder>`

Requires: Phases 1 and 2 complete in `.spike-state.json`. Fails with an error otherwise.

1. Read `.spike-state.json` and all files in the spike folder.
2. Assemble inputs as described in Phase 3 Step 3.1.
3. Run the writer→proofreader pipeline (Phase 3 Steps 3.2–3.5).
   Do not re-run any investigation or re-execute any flows.
4. Write the regenerated file and report the path.

---

## Add Context

`spike add-context <path-to-spike-folder> <url | file-path | inline-text>`

1. Read `.spike-state.json` and current `FINDINGS.md`.
2. Ingest the new information (fetch URL, read file, or accept inline text).
3. Identify which findings (F1, F2, …) are affected by the new information.
4. Propose specific updates to affected F-entries and the recommendation.
5. If the new context touches any finding with a hypothesized root cause, trigger the
   Validate Recommendation gate for that finding.
6. Do not modify any files until the user confirms the proposed changes.

---

## Publish

`spike publish <path-to-spike-folder>`

Two operations in sequence. Pause for confirmation between them.

**Step 1 — GitHub (prototype):**
- If `github_repo_url` is already set in `.spike-state.json`, skip and report.
- Otherwise: summarize what will be pushed, ask for approval, then:
  `gh repo create spike-<slug> --private --source=. --remote=origin --push`
  Record URL in `.spike-state.json`.

**Step 2 — Azure DevOps Wiki (findings doc):**
- Display the full `FINDINGS.md` content in the terminal for review.
- Ask for approval and the target wiki path before writing.
- On approval: use the `azure-devops` MCP server to create or update the wiki page.
- Record `wiki_page_url` in `.spike-state.json`.
