# Canonical Example — Spike Findings Document

This is the reference standard for `FINDINGS.md` quality.
It is derived from a real 2-day technical spike and annotated with margin notes (in `> ✎` blocks)
explaining why each section is written as it is.

Both the writer and proofreader subagents must read this before processing any document.
When in doubt about tone, length, or structure — match this example.

---

# THE EXAMPLE

---

[[_TOC_]]

> ✎ `[[_TOC_]]` is required by the Azure DevOps Wiki renderer. Always present, always first.

# Spike: Maestro E2E Feasibility on `aplicatudo`

|||
|---|---|
| **Spike ID** | SPIKE-maestro-aplicatudo |
| **Type** | Technical |
| **Timebox** | 2 days |
| **Date** | 2026-05-28 |
| **Project** | `aplicatudo` |
| **Tools** | Maestro 2.6.0 · Flutter 3.35.0 · Firebase Emulator Suite 15.16.0 |
| **Status** | Closed — GO VALIDATED |

> ✎ Metadata table: terse, factual, no prose. Tool versions are exact — "latest" is not acceptable.
> Status reflects the final state after the spike is closed, not "in progress".

## Question

> Can the test framework **Maestro** be used **effectively** for E2E testing on the `aplicatudo` project?

> ✎ The question is copied verbatim from the confirmed Spike Brief. It is a single question.
> "Effectively" is defined by the success criteria below — not left vague.

## Success Criteria

- SC1: Login flow runs end-to-end on Android against the local Firebase emulator without manual intervention.
- SC2: Create-student flow is either green or blocked with a diagnosed root cause and validated fix path.

> ✎ Criteria are concrete and testable. SC2 accepts a diagnosed blocker as a valid answer —
> the spike is time-boxed, not a guarantee of full green. This is honest scoping.

## Answer (TL;DR)

**GO — unconditional.** Maestro drives `aplicatudo` end-to-end on Android with no app-side changes
required. Both probe flows — login (F2) and create-student (F3) — pass live against the local
Firebase emulator. The full Maestro toolchain — CLI, Viewer for debugging, and Cloud for pipeline
execution — is available and suited to this project (F4). The primary follow-on investment is
CI/CD wiring (F5), which carries the most residual uncertainty and is recommended as its own spike.

> ✎ TL;DR: one paragraph, bottom-line first, finding labels as cross-references.
> It does NOT repeat the content of F2, F3, F4, F5 — it only names them.
> A reader who wants detail goes to the Findings section.
> The recommendation label (GO — unconditional) appears here and is echoed in the Recommendation section.

## What Was Done

| Activity | Result |
|---|---|
| Installed Maestro 2.6.0, booted AVD `flutter_emulator_2`, built and installed `local` flavor APK | ✅ |
| Started Firebase emulators (auth 9099, Firestore 8080, Functions 5001) | ✅ |
| Verified seeded account against Auth emulator REST API | ✅ |
| Ran Android login flow (`01_login.flow.yaml`) | ✅ PASS |
| Ran Android create-student flow (`02_create_student.flow.yaml`) | ✅ PASS |
| Investigated full Maestro toolchain: CLI, Viewer, Cloud | ✅ |
| Authored GitLab CI stub (`ci/maestro-android.gitlab-ci.yml`) | ✅ |
| Web flows authored | ⚠️ Written, not executed |

> ✎ This section enumerates *what was attempted*, not *what was found*.
> Results are one-line statuses (✅ / ⛔ / ⚠️), not explanations.
> Explanations belong in Findings.
> The ⚠️ on web flows is honest — it neither claims success nor buries the gap.

## Findings

> ✎ This is the factual core of the document. Every finding:
> - States one fact per entry.
> - Is written as a reference statement, not a story.
> - Includes an Implication that connects the fact to the recommendation.
> - Does NOT repeat what another finding already said.

---

### F1 — Maestro selects by accessibility tree; Flutter Keys are invisible

Maestro operates at the OS level against Android's accessibility tree. It runs outside the Flutter
runtime and cannot read Flutter `Key`s (`ValueKey`, `GlobalKey`, etc.). Flutter Keys remain
valuable for widget reconciliation and widget tests — this is a Maestro constraint, not a reason
to remove them.

The recommended selector is `Semantics(identifier: 'name')` in the widget tree, referenced in
YAML as `tapOn: { id: "name" }`. This selector is stable across locales, copy changes, and Flutter
version upgrades. Text selectors (`tapOn: { text: "Entrar" }`) work for standard Material widgets
and are used throughout this prototype.

**Implication:** For Maestro test selectors, add `Semantics(identifier:)` alongside existing Keys
on widgets that lack visible text. No Keys need to be removed.

> ✎ This finding states a framework constraint (fact), clarifies what it does NOT mean (Keys are
> still useful), and gives the concrete action (add Semantics alongside, not instead of).
> It does not narrate how this was discovered.

---

### F2 — Login flow passes end-to-end on Android

`.maestro/android/01_login.flow.yaml` completes against the Auth emulator (account
`criador@beta.bhave.life`) in under 20 s on every run. Sequence:
`launchApp (clearState) → fill email → fill password → tap Entrar → assertNotVisible "Entrar"`.

**Implication:** Maestro can drive the app on Android. This is the primary proof-of-viability result.

> ✎ Short. One paragraph. Cites the exact file, the exact account, the exact timing, the exact sequence.
> Everything is traceable. "This is the primary proof-of-viability result" is the implication, stated once.
> Compare with the BAD version: "We ran the login flow and it worked, which means Maestro can
> probably control our app, which is good news for the team." — narrative, hedged, padded.

---

### F3 — Create-student flow passes end-to-end on Android

`.maestro/android/02_create_student.flow.yaml` passes live. Full sequence:
```
launchApp (clearState)
→ login (via subflow)
→ tapOn "Instituto Beta"
→ assertVisible "Novo estudante" (20 s timeout)
→ tapOn "Novo estudante"
→ assertVisible "Adicionar estudante"
→ tapOn "Nome" → inputText → hideKeyboard → tapOn "Salvar"
→ assertVisible "Aluno Spike Maestro P4"
```
No app-side changes were required.

**Implication:** Multi-step cross-screen journeys — including profile selection, list loading, modal
interaction, and Firestore persistence — are testable with Maestro today.

> ✎ The sequence is shown as a code block because it is precise and machine-verifiable.
> The implication names what the sequence proves (cross-screen, persistence) — it does not just
> say "it works". No mention of how it was reached or what was tried before.

---

### F4 — Maestro toolchain: CLI, Viewer, and Cloud

**CLI** provides subcommands beyond `maestro test`: `hierarchy` (accessibility tree dump to
JSON), `record` (MP4 of a flow run), `cloud` (upload to managed device farm), `check-syntax`,
`list-devices`, `list-cloud-devices`, `start-device`, and `mcp` (Model Context Protocol server
for AI-agent integration).

**Maestro Viewer** is the visual element inspector: a browser/desktop app that embeds a running
device or emulator and provides an element inspector (click any element to get its accessibility
attributes and suggested `tapOn` selector), live YAML execution, flow authoring, and a navigable
hierarchy view. Primary use: diagnosing why a `tapOn` or `assertVisible` fails — the developer
sees exactly what Maestro sees.

**Maestro Cloud** is a managed virtual device farm at `app.maestro.dev`. Available Android devices:
`pixel_6` (android-29–34), `pixel_9` (android-35–36). Upload syntax:
`maestro cloud --api-key $KEY --app-file app.apk --flows .maestro/`. Output: per-flow pass/fail,
screen recording, step log, and UI hierarchy data per run. Eliminates local emulator setup in CI
pipelines; requires a paid plan. Cloud devices reach private/staging/`localhost` backends through
a secure tunnel (tunnel latency unmeasured).

**Implication:** The full toolchain covers local development (CLI + Viewer) and CI/CD execution
(Cloud). Both paths are suitable for this project; see F5 for the CI analysis.

> ✎ This finding covers three distinct tools in one entry because they form a single toolchain
> finding — the implication is about the toolchain as a whole, not about each tool individually.
> Each tool is described with concrete specifics: URL, device models, command syntax.
> No tool is described as "powerful" or "great" — capabilities are stated, not evaluated.

---

### F5 — CI/CD: two integration paths with different trade-off profiles

Two paths were investigated and stubbed. Neither was executed in a live CI environment.

**Path A — Local emulator** (`ci/maestro-android.gitlab-ci.yml`): boots a headless Android
emulator on a self-hosted runner, starts the Firebase emulators, builds the APK, and runs both
flows. Measured timings (local machine, `swiftshader_indirect`): APK build 5m 17s, Firebase
startup ~35s, APK install 3s, login flow 42s, create-student flow 1m 1s. Android emulator cold
start was not measured — the largest remaining unknown. Estimated total with KVM: 11–13 min;
without KVM: 16–23 min.

**Path B — Maestro Cloud** (`ci/maestro-cloud.gitlab-ci.yml`): uploads APK and flows to Maestro's
managed device farm. Eliminates emulator setup entirely. Pipeline reduces to: build APK → one
`maestro cloud` command. Requires a Cloud Plan account; the app can reach a local Firebase
emulator or a dev/staging backend through Cloud's secure tunnel (latency unmeasured).

| Dimension | Path A — Local emulator | Path B — Maestro Cloud |
|---|---|---|
| Emulator setup | Required (KVM recommended) | None |
| Firebase backend | Local emulator suite | Reachable via secure tunnel; or a real backend |
| Estimated duration | 11–23 min (KVM-dependent) | Build + Cloud queue (not measured) |
| Cost | Runner compute only | Paid plan required |
| Parallelism | 1 device per runner | Multiple devices, out of the box |
| Results | Log artifacts | Dashboard: recordings, step logs, hierarchy |

**Implication:** Both paths are viable. Path A is preferred when self-hosted KVM runners are
available or when deterministic local test data is required. Path B is preferred on shared runners
or when screen recordings and parallel execution are priorities. They are not mutually exclusive.

> ✎ This finding presents two alternatives with a comparison table — appropriate when the spike
> investigated two approaches rather than proving a single fact. The table replaces prose
> comparison; prose would repeat the same information twice.
> The implication gives a clear decision rule, not "it depends".
> The measured timings are exact (5m 17s, not "about 5 minutes").

---

### F6 — Flutter web requires explicit semantics tree activation

Flutter web does not build the accessibility/semantics tree by default. Without it, Maestro
cannot find any elements — all `assertVisible` and `tapOn` calls fail silently. The fix is one
guarded call in `main.dart`:

```dart
if (Environment.isRunningIntegrationTests()) {
  SemanticsBinding.instance.ensureSemantics();
}
```

The guard is critical: unguarded `ensureSemantics()` degrades production rendering performance.
The patch is authored at `patches/0001-enable-semantics-for-e2e.patch`. Web flows were written
but not executed within this spike's timebox.

**Implication:** Web E2E coverage requires shipping the guarded patch and executing the web flows.
The mechanism is confirmed by the same semantics-tree dependency observed on Android.

> ✎ Code block used because the exact call signature matters. The guard rationale is explained
> because omitting it would be a production defect — this is not padding, it is a necessary
> safety note. The "not executed" fact is stated plainly, not buried.

---

## Recommendation

> ✎ Stakeholder paragraph ALWAYS before the technical table. This is non-negotiable.
> The stakeholder paragraph uses no code, no jargon, max 5 sentences.

### For stakeholders

We confirmed Maestro can automatically test the app across both the login journey and the full
add-a-student journey — both run hands-free against our local test backend on Android. The
framework includes a desktop debugging tool (Maestro Viewer) that lets developers see exactly
what the automated test sees, and a cloud device farm (Maestro Cloud) for running tests in CI
without managing emulators locally. To adopt Maestro, the team needs to standardise on
accessibility identifiers for reliable selectors, establish a maintained E2E suite in the main
repository, and wire it into the CI pipeline. The recommendation is: **go ahead**.

### Technical

**Decision:** GO ✅ — unconditional

|||
|---|---|
| **Rationale** | Both probe flows pass live on Android against the local Firebase emulator. No Maestro limitation was found that prevents E2E coverage. The full toolchain (F4) is suited to the project's delivery model. |
| **Conditions** | None. Flows work today without app-side changes. |
| **Confidence** | High |

> ✎ Rationale cites F4 by label rather than repeating toolchain content.
> Conditions is "None" — this is an unconditional GO. A conditional GO would list the condition
> and its validation status (VALIDATED ✅ or UNVALIDATED ⚠️).

---

## Refined Estimate

Estimate for the team's real adoption of Maestro E2E on `aplicatudo`, not for the prototype.

| Work item | Estimate | Confidence | Key assumptions |
|---|---|---|---|
| Adopt `Semantics(identifier:)` as the selector standard on tested screens | 2–3 pts | High | Applied incrementally per screen brought under E2E coverage |
| Add the guarded `ensureSemantics()` call for web E2E | 0.5 pt | High | Single guarded call in `main()` |
| Establish a maintained E2E flow suite in the main repo (starting with login + create-student) | 2–3 pts | High | Flow authoring is straightforward; covers the first journeys |
| **Subtotal — app + test-suite** | **~5–6 pts** | **High** | |
| CI wiring: integrate the flows into the delivery pipeline; verify KVM; configure caches | 2–4 pts | Medium | Local execution timings measured (F5); emulator boot on CI runner is the remaining unknown |
| **Total (incl. CI)** | **~7–10 pts** | **Medium** | CI is the largest residual unknown |

> ✎ Every row has estimate, confidence, and key assumptions — no row is missing a column.
> Work items target the team's real adoption, NOT edits to the throwaway prototype.
> WRONG: "Standardise login selectors — 3 elements in the login form" (edits the PoC).
> RIGHT: "Adopt Semantics(identifier:) as the standard on tested screens" (project-scale).
> Confidence levels reflect what was actually proven: app-side is High because the mechanism
> was demonstrated; CI is Medium because emulator boot on the real runner was not measured.

---

## Parking Lot

- **CI/CD spike** — measure emulator cold-start on the actual GitLab runner; verify KVM availability; evaluate Maestro Cloud as the zero-emulator alternative. Stubs ready in `ci/`.
- **Maestro Cloud pricing and backend configuration** — plan cost, APK upload policy, and switching the app to target a real backend not evaluated.
- **iOS** — excluded from this spike. Maestro supports iOS natively via `.app` upload.
- **Web flows** — written, not executed. Requires `ensureSemantics()` patch (F6) and a web build.
- **Selector strategy** — the login form uses `Key`-based selectors in widget tests and text selectors in Maestro flows. Both coexist; adding `Semantics(identifier:)` alongside existing Keys would give Maestro stable selectors without modifying widget tests.

> ✎ Parking Lot items are specific enough to become actionable tickets. Each is one sentence
> with a clear scope boundary. "Look into iOS someday" is not a Parking Lot item — "Maestro
> supports iOS natively via .app upload; not evaluated in this spike" is.

---

## Evidence

- `evidence/01-login-success.png` — app on the profile selector immediately after login completes, confirming the login flow exit.
- `evidence/02-profile-selector.png` — profile/workgroup selector screen with "Instituto Beta" tile visible and selectable.
- `evidence/03-student-list.png` — student list loading after profile selection via `tapOn`.
- `evidence/04-create-student-pass.png` — student list after create-student flow; "Aluno Spike Maestro P4" confirmed in list.

> ✎ Each evidence item has one sentence describing what it shows and why it matters.
> A filename alone ("evidence/01.png") is not acceptable — it tells the reader nothing.

## Prototype

Archived (not merged): `spike-maestro-aplicatudo/`. See `README.md` for run instructions and
`scripts/run-android-all.sh` for the fully automated local run.
GitHub: https://github.com/theocarranza/spike-maestro-aplicatudo (private)

> ✎ Prototype section is a pointer, not a repeat of the README. One sentence for the folder,
> one for the script, one for the GitHub URL. Setup instructions live in the README only.

---

# Spike Close-Out

|||
|---|---|
| **Question answered?** | ✅ Yes |
| **Success criteria** | SC1: met — login flow PASS on Android. SC2: met — create-student flow PASS on Android. |
| **Recommendation validation** | VALIDATED ✅ — both flows run live; GO is unconditional |
| **Timebox** | Held |
| **Code merged?** | No — prototype archived |

**Knowledge produced:** Maestro reads Flutter's accessibility tree; Flutter Keys are invisible to
it but remain valuable for widget reconciliation. Login and create-student flows pass on Android
with no app changes. Maestro Viewer provides an interactive element inspector for writing and
debugging flows. CI wiring is the primary residual unknown; two viable paths are documented with
measured timings for the local-emulator path.

**Recommended follow-on tickets:**
1. Adopt `Semantics(identifier:)` as the project's E2E selector standard on tested screens.
2. Ship the guarded `ensureSemantics()` call to enable web E2E.
3. Establish a maintained Maestro E2E suite in the main repo, starting with login + create-student.
4. **New spike:** Maestro in CI — measure emulator boot on GitLab runner; evaluate Maestro Cloud.

> ✎ Close-Out: every field filled. Success criteria echoed verbatim from the Spike Brief.
> "Knowledge produced" is 3–4 sentences stating what is now known that was not known before —
> not a summary of what the spike did, but what it proved.
> Follow-on tickets are specific enough to estimate; no ticket is "investigate X further".

---

# ANNOTATION SUMMARY — WHAT MAKES THIS DOCUMENT GOOD

1. **Short.** ~2 pages of substantive content. No section padded to look thorough.

2. **Reference, not narrative.** Every sentence states a fact or its implication.
   No sentence describes what the investigator tried, felt, or expected.

3. **One fact, one place.** F2 states the login result. The TL;DR cites F2 but does not
   repeat the timing or sequence. The Recommendation cites F2 but does not repeat the result.

4. **Traced.** Every claim points to a file, a timing, a command, or a direct observation.
   "Maestro can drive the app" → proven by F2 (specific file, specific account, specific timing).

5. **Honest.** Web flows not executed → stated plainly as ⚠️, not omitted.
   CI emulator boot not measured → stated as the largest remaining unknown, not glossed over.

6. **Decision-useful.** A reader who has never heard of Maestro can read this document and
   decide whether to adopt it. The Parking Lot tells them exactly what to investigate next.
