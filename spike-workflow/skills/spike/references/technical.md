# Technical Spike — Investigation Reference

Agent-read guidance. Dense and prescriptive. Read this before Phase 1 on any technical spike.

---

## 1. Investigation Checklist

Work through every item. Document the result of each — including "not applicable" and "blocked."
Do not skip items silently; skipped items are Parking Lot candidates.

### 1.1 — Read before you run

- [ ] Official documentation for the tool/framework (current version, not cached training data — use WebFetch)
- [ ] Changelog / release notes for the version under investigation
- [ ] Known issues tracker (GitHub issues, forum, community Slack)
- [ ] Any existing usage in the codebase (`grep` / Glob for imports, config files, usage patterns)
- [ ] Competing alternatives (even if not in scope — knowing what was rejected and why is a finding)

### 1.2 — Enumerate the full tool surface (mandatory)

Before writing a single finding, document:

| Surface | Description | Status |
|---|---|---|
| Primary API / SDK | Core commands, selectors, assertions | Investigated |
| Debugging tool | How a developer debugs a failing test | Investigated |
| CI / remote-run | How this runs in a pipeline or device farm | Investigated |
| Device / env requirements | OS, emulator, browser, runtime versions | Investigated |
| Cloud / device-farm option | Vendor-hosted execution, if available | Investigated |

A finding that does not address all five rows is incomplete. If a row cannot be investigated
within the timebox, it becomes a Parking Lot item with an explicit "not investigated" note.

### 1.3 — CI/CD checkpoint

Ask: will this tool run in CI? For a testing framework, a build tool, or anything in the
delivery pipeline — the answer is YES.

If YES:
- Identify the runner requirements (OS, CPU, memory, installed tools).
- Identify the emulator/device requirements (headless emulator? real device? cloud device?).
- Write a minimal pipeline stub that boots the environment and runs the green-path test.
  - GitHub Actions: `.github/workflows/spike.yml`
  - GitLab CI: `ci/spike.gitlab-ci.yml`
  - Use the project's actual pipeline format if known.
- Run the stub locally or in CI. Document the result. This is a finding.
- Estimate pipeline duration from timing data (not guesses).

### 1.4 — Run, don't guess

- Install the tool and run the minimal example from the official docs. Document output verbatim.
- Run the failure case — deliberately trigger the blocker you expect. Document what happens.
- Capture version numbers of everything: tool, runtime, OS, emulator API level.
- If something fails unexpectedly: search the issue tracker before diagnosing internally.

### 1.5 — Document dead ends

Every dead end is a finding. Structure:

```
Dead end: <what was attempted>
Expected: <what was expected>
Actual: <what happened>
Conclusion: <why this path was abandoned>
```

---

## 2. Diagnosing Blockers — Diagnosed vs. Hypothesized

Before labelling a blocker finding, determine whether the root cause is **diagnosed** or **hypothesized**.

**Diagnosed** = you have direct evidence for the cause.
**Hypothesized** = you have inferred the cause from a known pattern without direct confirmation.

### Procedure to diagnose

1. **Capture the raw failure output.** Error message, stack trace, or tool output verbatim.
2. **Inspect the runtime state.** For UI tests: use the debugging tool (see §3) to capture
   the view hierarchy or accessibility tree at the point of failure.
3. **Read the source.** If the problem is in app code, read the relevant widget/component.
   If the problem is in the tool, read the error docs or issue tracker.
4. **Propose a specific cause.** State it as a falsifiable claim: "The button is invisible
   because it is wrapped in `ExcludeSemantics`" — not "the button is probably not semantic."
5. **Attempt to falsify it.** Make the minimal change that the proposed cause predicts would fix
   the problem. If the symptom changes as predicted, the cause is diagnosed.

If step 5 is not completed within the timebox: the cause remains hypothesized.
Label accordingly. A hypothesized root cause on a critical finding triggers the
Validate Recommendation gate.

---

## 3. Stack-Specific Pitfalls — Flutter + Maestro

Seeded from the Maestro/aplicatudo spike (2026-05-28). Extend this section as more spikes run.

### Maestro fundamentals

- Maestro reads **Flutter's Semantics/accessibility tree**, not the widget tree.
  It operates at the OS level, not inside the Dart runtime.
- **Flutter `Key`s are invisible to Maestro.** `ValueKey`, `GlobalKey`, `Key('foo')` —
  none of these are accessible. Do not design flows around them.
- The recommended selector is `Semantics(identifier: 'name')` in the widget tree,
  selected in YAML via `tapOn: { id: "name" }`.
  This is stable across locales, copy changes, and Flutter version upgrades.
- Text selectors (`tapOn: { text: "Entrar" }`) work for visible text but break on
  copy changes and i18n. Use them only when semantic IDs are absent.

### Why a text selector can fail even though text is visible

This is a common diagnostic error. Flutter's `Text` widget does populate the semantics
tree implicitly — but several app-side constructs suppress it:

| Cause | How to confirm | Fix |
|---|---|---|
| `ExcludeSemantics` wrapper | View hierarchy shows no semantic node for the widget | Remove `ExcludeSemantics` or add `Semantics(identifier:)` outside it |
| `MergeSemantics` absorbing children | Hierarchy shows parent node with merged label | Add explicit `identifier` to the merged parent |
| WebView-rendered content | Hierarchy shows a WebView node, not text nodes | Cannot use text selectors; requires native bridge or test-only bypass |
| Custom-painted widget | Hierarchy shows no text node at all | Wrap with `Semantics(label:, identifier:)` |
| Modal barrier blocking input | Input events land on the barrier, not the widget | Dismiss barrier before asserting/tapping |
| Semantics tree not built (Flutter web) | All `assertVisible` calls fail silently | Call `SemanticsBinding.instance.ensureSemantics()` guarded by a test flag |

**How to confirm using Maestro Studio:**
1. Connect Maestro Studio to the running device/emulator.
2. Navigate the app to the failing screen.
3. Open the element inspector. The hierarchy pane shows every semantic node.
4. If the element you want is absent or has no `identifier`/`label`, the cause is a missing
   semantic node — not a Maestro selector error.

### Flutter web

Flutter web does **not** build the semantics tree by default. Without it, Maestro is blind.

```dart
// main.dart — guard with a test-only flag
if (Environment.isRunningIntegrationTests()) {
  SemanticsBinding.instance.ensureSemantics();
}
```

`WidgetsBinding.instance.ensureSemantics()` also works (same underlying call).
The guard is critical — unguarded `ensureSemantics()` degrades production rendering performance.

### Coordinate-based navigation — known pitfall

Tapping by raw screen coordinates bypasses the semantics tree. This has two failure modes:
1. **Coordinates shift** across screen sizes, system font scales, and orientation changes.
2. **Application state is not established** the same way real UI navigation establishes it.
   Flows that depend on state set up by a screen (e.g. a profile-selection screen that
   writes to an account-session stream) will fail downstream even if the coordinate tap
   "works" visually. Label any flow using coordinates as brittle and document the state
   assumption explicitly.

---

## 4. CI/CD Stub — Maestro on Android (starter template)

Use this as the basis for the `/ci` stub when spiking Maestro on a Flutter project.
Replace emulator AVD name, package ID, and flow path as appropriate.

```yaml
# ci/maestro-android.gitlab-ci.yml
# SPIKE PROTOTYPE — ARCHIVED — NOT FOR PRODUCTION
# Runs the Maestro login flow on a headless Android emulator.
# Requires: ANDROID_HOME set on the runner; AVD pre-created or created below.

maestro_android:
  image: ubuntu:22.04
  variables:
    AVD_NAME: "spike_avd"
    API_LEVEL: "33"
    FLUTTER_VERSION: "3.35.0"
  before_script:
    - apt-get update && apt-get install -y openjdk-17-jdk curl unzip
    - curl -fsSL https://get.maestro.mobile.dev | bash
    - export PATH="$HOME/.maestro/bin:$PATH"
    # Install Flutter via FVM or direct download
    - curl -fsSL https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_${FLUTTER_VERSION}-stable.tar.xz | tar xJ -C /opt
    - export PATH="/opt/flutter/bin:$PATH"
    # Boot headless emulator (assumes AVD pre-created on runner)
    - emulator -avd $AVD_NAME -no-window -gpu swiftshader_indirect -no-audio &
    - adb wait-for-device
    - adb shell input keyevent 82  # unlock screen
  script:
    - fvm flutter build apk --flavor local --dart-define=IS_RUNNING_INTEGRATION_TESTS=true
    - adb install -r build/app/outputs/flutter-apk/app-local-release.apk
    - maestro test .maestro/android/01_login.flow.yaml
  artifacts:
    when: always
    paths:
      - .maestro/logs/
      - maestro-output/
```

**Known unknowns this stub does not resolve:**
- AVD provisioning time on CI runners (can be 3–10 min; measure on your runner).
- Firebase emulator startup in CI (add a `firebase emulators:start --only auth,firestore` step).
- Runner availability for hardware-accelerated emulation (requires KVM on Linux runners).

These are the items a follow-on CI spike must quantify.

---

## 5. Implementation Notes

- Build the minimal prototype that answers the question. No extras.
- The CI stub is part of the prototype if the CI/CD checkpoint fired — not an afterthought.
- Record version numbers of every tool at the top of `README.md`. Version drift is a common
  failure mode when reproducing a spike result later.
- Tests are executable documentation. Name them after what they prove, not what they do:
  - Good: `test_maestro_can_drive_login_flow_on_android`
  - Bad: `test_login`
