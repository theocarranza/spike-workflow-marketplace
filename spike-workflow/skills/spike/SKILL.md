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
disable-model-invocation: true
allowed-tools: >
  Read Write Edit Glob
  WebFetch WebSearch
  Bash(git init *) Bash(git add *) Bash(git commit *) Bash(git remote add *)
  Bash(git push *) Bash(git branch *) Bash(git log *) Bash(git show *)
  Bash(git rev-parse *) Bash(gh repo create *) Bash(gh repo view *)
  Bash(flutter test *) Bash(fvm flutter test *) Bash(flutter pub get *)
  Bash(dart analyze *) Bash(npm test *) Bash(npm install *)
  Bash(pytest *) Bash(pip install *)
  Bash(mkdir *) Bash(date *)
---

# Spike Skill

Executes a full spike investigation and produces all required deliverables in compliance
with the accepted Agile/XP definition of a spike.

---

## Inputs

Claude must have the required inputs before starting. If any are missing, ask before proceeding.

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
- The prototype must be structurally compatible with the patterns shown — same state shape, same naming conventions, same layer boundaries.
- Use it during Phase 1 to understand the constraints the answer must fit.
- Reference it explicitly in the findings document under "Approach."

---

## Output Destinations

| Deliverable | Destination |
|---|---|
| Findings document | **Azure DevOps Wiki** — format as a wiki page (see `references/deliverables.md`) |
| Recommendation statement | Inline in the findings wiki page — stakeholder paragraph first, then technical detail |
| Refined estimate | Inline in the findings wiki page |
| Prototype | Standalone project folder (`spike-<slug>/`) — archived in the repository or designated spike archive |

---

## Research Configuration

Phase 1 requires deep reasoning. Apply the following:

| Setting | Value | Rationale |
|---|---|---|
| Model | `claude-opus-4` (or strongest available) | Spike questions are often ambiguous; deep reasoning reduces missed failure modes |
| Extended thinking | **On** | Surfaces non-obvious constraints and design trade-offs |
| Web search | **On** | External docs, package changelogs, known issues, community reports |

**Environment notes:**
- **claude.ai** — Research runs inline. Enable web search; use best available reasoning capability.
- **Claude Code / agentic context** — Spawn a dedicated research subagent with the above config for Phase 1 before proceeding to Phase 2.

---

## Rules (enforce strictly)

1. **One question per spike.** If the user's input contains multiple unknowns, decompose into separate spikes and ask which to run first.
2. **Time-boxed.** Acknowledge the timebox at the start. Do not expand scope beyond it.
3. **The prototype is functional.** If a prototype is produced, it must run and be presentable as a proof of concept. It must pass its own tests before being presented.
4. **The prototype is archived, not merged.** Make this explicit in all outputs — the code is reference material, not production-ready.
5. **Definition of Done = question answered.** The spike ends when the original question has a documented answer — not when the code "works."
6. **No estimation of unrelated work.** Only estimate the implementation task that directly follows from this spike's findings.

---

## Execution Phases

### Phase 0 — Frame the Spike

Output a **Spike Brief** before doing any investigation:

```
## Spike Brief
- **Question:** <exact question being answered>
- **Type:** <technical | functional | risk>
- **Timebox:** <agreed duration>
- **Success criteria:** <how we'll know the question is answered>
- **Out of scope:** <explicit exclusions>
```

Ask the user to confirm before proceeding to Phase 1.

---

### Phase 1 — Investigate

Read the relevant reference file for investigation guidance:

- Technical spike → `references/technical.md` § Investigation
- Functional spike → `references/functional.md` § Investigation
- Risk spike → `references/risk.md` § Investigation

Perform the investigation. Document what you tried, what you read, what you ran,
and what you observed. Be explicit about dead ends — they are findings too.

If `code_context` was provided, analyse it before investigating externally:
identify the patterns, constraints, and conventions the solution must respect.

---

### Phase 2 — Build Prototype (if applicable)

A prototype is required when:
- The spike type is **technical** and the question requires code to answer
- The spike type is **risk** and feasibility must be demonstrated, not just argued

A prototype is **not** required for purely functional spikes (unless the question involves
a UI/UX flow that must be validated visually).

Read `references/technical.md` § Implementation Notes (or the relevant type file)
before building — this section contains known patterns and pitfalls for the stack.

If building a prototype:
1. Build the minimal version that answers the question — no extras
2. Structure it as a **proper project** — not a script dump:
   - Standard project layout for the stack (e.g. `src/`, `tests/`, config files)
   - Dependency manifest (`pubspec.yaml`, `package.json`, `requirements.txt`, etc.)
   - `.gitignore` appropriate for the stack
3. If `code_context` was provided, match its state shapes, naming conventions, and layer boundaries
4. It must **run** from a clean environment — no hidden setup, no broken imports, no placeholder logic in critical paths
5. Include **tests** scoped to proving the investigated mechanisms work:
   - Tests are executable documentation of what the spike proves
   - Focus on the capability under investigation, not coverage
6. Include a `README.md` with: what it demonstrates, prerequisites, run instructions, test instructions, expected output, and explicit scope exclusions
7. Mark all source files with: `// SPIKE PROTOTYPE — ARCHIVED — NOT FOR PRODUCTION`
8. Package into a folder named: `spike-<slug>/`

**Gate before Phase 3 — do not proceed until:**
- [ ] All tests pass (`flutter test`, `npm test`, `pytest`, etc.)
- [ ] The app/script runs and produces the expected output from the README
- [ ] If either fails: fix it. Do not present a prototype that does not run.

**Publish the prototype to GitHub:**

Once the gate passes:
1. `git init` inside the prototype folder
2. `git branch -M main`
3. Stage and commit all files:
   ```
   git add .
   git commit -m "chore: spike prototype — archived, not for production"
   ```
4. Create the private GitHub repository:
   ```
   gh repo create spike-<slug> --private --source=. --remote=origin --push
   ```
5. Confirm the repo URL from `gh repo view spike-<slug> --json url -q .url`
6. Record the URL — it will be linked in the findings document under the Prototype section.

---

### Phase 3 — Produce Deliverables

Always produce all of the following. Use `references/deliverables.md` for templates.

#### 1. Findings Document

Full Azure DevOps Wiki page. See `references/deliverables.md` for the complete template.

#### 2. Recommendation

**Always produce two layers:**

**Layer 1 — Stakeholder paragraph (always first):**
A plain-language paragraph, max 5 sentences, suitable for a product owner or non-technical
stakeholder. No code, no jargon. Structure: what was investigated → what was found →
what is recommended → what happens next.

**Layer 2 — Technical recommendation (immediately after):**

*Go / No-Go* — use when the question is binary:
```
Recommendation: GO ✅ | NO-GO ❌
Rationale: <2-3 sentences>
Conditions: <conditions that must hold, or "None">
```

*Ranked Comparison* — use when multiple approaches were investigated:
```
Recommendation: Approach <X> over <Y> [over <Z>]

| Criterion      | Approach A | Approach B | Approach C |
|---|---|---|---|
| <criterion 1> | ...        | ...        | ...        |

Rationale: <why the winner prevails on the criteria that matter most>
Trade-offs accepted: <what we give up>
```

Never produce a vague "it depends" — commit to a recommendation with documented rationale.

#### 3. Refined Estimate (if applicable)

If the spike unblocks a specific implementation task:
- Task name
- Estimated effort (story points or time range)
- Confidence level: Low / Medium / High
- Key assumptions the estimate depends on

---

### Phase 4 — Close the Spike

```
## Spike Closed
- **Question answered:** Yes / Partially / No
- **If partially/no:** reason and recommended follow-up spike
- **Prototype archived:** Yes / N/A
- **Next action:** <specific next step for the team>
```

---

## What NOT to do

- Do not write production code during a spike
- Do not merge or propose merging prototype code
- Do not present a prototype that does not run or whose tests fail
- Do not leave the original question unanswered without explicitly flagging it
- Do not expand scope during investigation — log tangential findings as Parking Lot items
- Do not skip the Spike Brief confirmation step
- Do not produce a recommendation without the stakeholder paragraph

---

## Parking Lot

If you discover important unknowns or tangential questions during investigation,
log them in a **Parking Lot** section at the end of the findings document.
These become candidates for future spikes or tickets.
