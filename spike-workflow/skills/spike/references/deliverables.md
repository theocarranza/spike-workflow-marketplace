# Spike Deliverables — Reference & Templates

This file is agent-read guidance. It is dense and prescriptive by design.
Use it every time you produce a findings document. Do not improvise the structure.

---

## 1. Grounding — What Good Looks Like

Before writing anything, internalize these standards. Real spike artifacts from teams
that practise XP/SAFe share four properties:

1. **Reference, not narrative.** The document states facts. It does not narrate discovery.
   A reader who was not present should get the same information as one who was.
2. **One fact, one place.** Each finding is stated once in the Findings section.
   Every other section cross-references by label (F1, F2, …). Repetition is a defect.
3. **Concrete evidence.** Every claim traces to something observable: a test run, a
   screenshot, a log line, a doc quote. "It seems like" and "probably" are not findings.
4. **Honest incompleteness.** Unresolved questions are explicitly labelled — not omitted,
   not buried in prose. A spike that admits "we don't know X" is more valuable than one
   that implies it does.

**Example of a well-written finding:**

> **F2 — Login flow passes end-to-end on Android**
> The flow in `.maestro/android/01_login.flow.yaml` was run live against the Auth emulator
> (account `criador@beta.bhave.life`). It exits the login screen within 20 s on every run.
> **Implication:** Maestro can drive the app on Android. This is the primary "yes it works" proof.
> **Root cause:** N/A (positive result).

**Example of a narrative finding (do not write this):**

> We tried logging in and it worked after some fiddling with the emulator. The email and
> password fields were found by text since keys don't work. So Maestro can see the app.

The second version narrates; the first states. Write the first version.

---

## 2. Findings Document — Full Template

Copy this skeleton for every spike. Fill every field. Do not remove sections.
Sections that are genuinely N/A get a one-line "N/A — [reason]" rather than being deleted.

```markdown
[[_TOC_]]

# Spike: <Title>

|||
|---|---|
| **Spike ID** | SPIKE-<slug> |
| **Type** | Technical | Functional | Risk |
| **Timebox** | <duration> |
| **Date** | YYYY-MM-DD |
| **Project** | `<project-name>` |
| **Tools** | <tool versions> |
| **Status** | Open | Closed — recommendation issued |

## Question

> <Exact question, verbatim from the Spike Brief>

## Success Criteria (registered at Phase 0)

> Copied verbatim from the confirmed Spike Brief. These are echoed in the Close-Out.

- SC1: <criterion>
- SC2: <criterion>

## Answer (TL;DR)

> One paragraph. States the bottom-line answer to the question. References the key finding
> labels (F1, F2, …) — does not repeat their content. Includes the recommendation label:
> GO / NO-GO / CONDITIONAL GO [VALIDATED | UNVALIDATED — follow-on required].

## What Was Done

| Activity | Result |
|---|---|
| <action taken> | ✅ / ⛔ / ⚠️ + one-line result |

_Purpose of this section: enumerate what was attempted, not what was found. Results go in Findings._

## Findings

_Each finding is labelled F1, F2, … State each fact once. Cross-reference from other sections._
_Every blocker finding includes a root-cause label (see template below)._

### F1 — <Title>

<Fact. One paragraph. What is true, what was observed, what it means.>

**Implication:** <what this means for the recommendation or next steps>

### F2 — <Title (blocker example)>

<Fact.>

**Root cause:** diagnosed | hypothesized
**Basis:** <one sentence: what was observed/read/run that supports this label>
**Implication:** <what must change for this to be resolved>

_[Continue F3, F4, … as needed]_

## Recommendation

_Stakeholder paragraph MUST appear before the technical table. This is a hard rule._

### For stakeholders

<Plain language, max 5 sentences, no code or jargon.
Structure: what was investigated → what was found → what is recommended → what happens next.>

### Technical

**Decision:** GO ✅ | NO-GO ❌ | CONDITIONAL GO [VALIDATED ✅ | UNVALIDATED ⚠️ — follow-on required]

| | |
|---|---|
| **Rationale** | <2–3 sentences> |
| **Conditions** | <conditions that must hold, or "None"> |
| **Confidence** | High / Medium / Low |

_If UNVALIDATED: state the condition that could not be tested and the reason it was not tested._

## Refined Estimate

_Only if the spike unblocks a specific implementation task._

| Work item | Estimate | Confidence | Key assumptions |
|---|---|---|---|
| <task> | <pts or time> | High/Med/Low | <assumptions> |

_CI/CD wiring, if not investigated, carries its own line with confidence Low._

## Parking Lot

_Tangential unknowns discovered during the spike. Candidates for future spikes or tickets._

- <item>

## Evidence

_List each artifact. For screenshots, state what the screenshot shows, not just its filename._

- `evidence/01-<name>.png` — <one sentence: what this shows and why it matters>

## Prototype

_Cross-reference the prototype folder and its README. Do not repeat setup instructions here._

Archived (not merged): `spike-<slug>/`. See `README.md` for run instructions.
GitHub: <URL or "pending publish">

## CI/CD Feasibility

_Required if the CI/CD checkpoint fired in Phase 1. N/A otherwise._

<Findings specific to pipeline integration: runner requirements, emulator provisioning,
artifact capture, estimated pipeline time. Reference the `/ci` stub in the prototype.>

---

# Spike Close-Out

|||
|---|---|
| **Question answered?** | Yes / Partially / No |
| **Success criteria** | SC1: met/not met. SC2: met/not met. |
| **Recommendation validation** | VALIDATED / UNVALIDATED — follow-on required / N/A |
| **Timebox** | Held / Exceeded by <duration> |
| **Code merged?** | No — prototype archived |

**Knowledge produced:** <2–3 sentences summarizing what is now known that was not known before>

**Deliverables:**
- [x] Runnable prototype
- [x] Findings doc
- [x] Stakeholder summary + GO/NO-GO
- [x] Refined estimate
- [x] Parking Lot
- [x] Evidence
- [ ] Prototype published to GitHub — pending user approval
- [ ] Wiki published to Azure DevOps — pending user approval

**Recommended follow-on tickets:**
1. <ticket>
```

---

## 3. No-Repetition Rule

State each piece of information exactly once. The table below maps content to its canonical section:

| Content type | Lives in | Referenced from |
|---|---|---|
| What was attempted | What Was Done | — |
| Facts / observations / evidence | Findings (Fx label) | TL;DR, Recommendation, Close-Out |
| Bottom-line answer | Answer (TL;DR) | — |
| Plain-language summary | Recommendation § stakeholder | — |
| GO/NO-GO decision | Recommendation § technical | Close-Out |
| Follow-on items | Parking Lot | Close-Out (count only) |
| Prototype instructions | README.md | Prototype section (link only) |

If you find yourself writing the same sentence in two sections, delete one and add a cross-reference.

---

## 4. Recommendation Validation Labels

| Label | Meaning | When to use |
|---|---|---|
| `VALIDATED ✅` | The condition was tested empirically and passed within the timebox | Condition exercised, flow or test confirmed it |
| `UNVALIDATED ⚠️ — follow-on required` | The condition was not tested; reason documented | Time ran out, environment unavailable, or condition is inherently future-work |
| `N/A` | Recommendation is unconditional (GO or NO-GO with no conditions) | Unconditional decisions |

A bare "Conditional GO" without a validation label is malformed.

---

## 5. Real Spike Examples (calibration)

These sources were verified as of 2026-05-29. Read at least one before producing a findings document.

- **Microsoft Engineering Fundamentals Playbook — Technical Spike:**
  https://microsoft.github.io/code-with-engineering-playbook/design/design-reviews/recipes/technical-spike/
  Key principles extracted: (1) goal is fact-finding, not decision-making; (2) include a repeatable environment section (components, installation, configuration) so results can be re-verified; (3) organise with summary first and detailed evidence in appendices; (4) use a table of contents and section headers. Aligns closely with this template.

- **SAFe — Spikes:**
  https://framework.scaledagile.com/spikes/
  Spikes are enabler stories: estimated, implemented, and demonstrated like any other story. Purpose: gain knowledge to reduce risk, understand a requirement, or improve estimate reliability. (Full article requires login — key definition extracted from public summary.)

- **Martin Fowler — SpikeAndStabilize:**
  https://martinfowler.com/bliki/SpikeAndStabilize.html
  (404 as of 2026-05-29 — consult the Fowler bliki index for current URL.)

- **Ron Jeffries — original XP spike definition:**
  https://ronjeffries.com/xprog/articles/spikes/
  (404 as of 2026-05-29 — consult ronjeffries.com for current URL.)

Key observation: **the document is short**. A well-executed 2-day spike produces a 1–2 page findings doc, not a 10-page narrative. Length is not a proxy for quality. If you find yourself repeating content across sections, apply the no-repetition rule.
