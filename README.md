# spike-workflow

One skill for running a full Agile/XP spike investigation — from question to archived prototype to wiki findings page.

## What's in v0.1.0

### Skill

`/spike-workflow:spike` — interactive walkthrough that takes a spike question and produces all required deliverables.

## Usage

```
/spike-workflow:spike
```

Claude will ask for the required inputs (question, type, timebox, context) and optionally `code_context` (existing code snippets the prototype must be compatible with), then run all four phases.

## Phases

| Phase | What happens |
|---|---|
| 0 — Frame | Spike Brief produced and confirmed before any work begins |
| 1 — Investigate | Deep research: docs, changelogs, community issues, code analysis |
| 2 — Prototype | Proper project scaffolded, tests written, compile gate enforced, pushed to private GitHub repo |
| 3 — Deliverables | Azure DevOps Wiki findings page, stakeholder recommendation, technical recommendation, refined estimate |
| 4 — Close | Explicit closing statement: question answered, prototype archived, next action |

## Deliverables

Every spike produces:

- **Findings document** — formatted as an Azure DevOps Wiki page (`spike-findings.md`)
- **Stakeholder paragraph** — plain-language recommendation for product owners
- **Technical recommendation** — Go/No-Go or ranked comparison
- **Refined estimate** — for the implementation task now unblocked (when applicable)
- **Prototype** — functional project with tests, README, run instructions, pushed to a private GitHub repo named `spike-<slug>`

## Inputs

| Input | Required | Description |
|---|---|---|
| `question` | ✅ | The single question the spike must answer |
| `type` | ✅ | `technical` \| `functional` \| `risk` |
| `timebox` | ✅ | Duration agreed by the team |
| `context` | ✅ | Project, stack, constraints |
| `code_context` | Optional | Existing code snippets the prototype must be compatible with |

## Requirements

- `gh` CLI authenticated (`gh auth login`) — used to create the prototype's private GitHub repo
- `git` — standard
- Stack-specific: `flutter`/`fvm`, `npm`, or `pytest` depending on your project

## Installation

```
/plugin marketplace add <your-github-owner>/spike-workflow-marketplace
/plugin install spike-workflow@spike-workflow-marketplace
```

## Layout

```
spike-workflow/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── spike/
│       ├── SKILL.md
│       └── references/
│           ├── technical.md
│           ├── functional.md
│           ├── risk.md
│           └── deliverables.md
└── README.md
```

## Dependencies

- `bash`
- `jq`
- `git`
- `gh` (GitHub CLI) — `brew install gh` / `sudo apt install gh`
