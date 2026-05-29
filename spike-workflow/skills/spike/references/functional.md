# Functional Spike — Investigation Reference

Agent-read guidance. Read before Phase 1 on any functional spike.

A functional spike investigates *what* the system should do — requirements, user flows,
acceptance criteria — not *how* to build it technically. The output is a validated
understanding of intended behaviour, not a code prototype.

---

## 1. Investigation Checklist

### 1.1 — Frame the ambiguity

The spike exists because a user story or requirement is ambiguous. Before investigating,
state the ambiguity explicitly:

```
Ambiguity: <exact sentence or concept that is unclear>
Options considered:
  A: <interpretation A>
  B: <interpretation B>
The spike resolves this by: <method — user interview, prototype, doc review, etc.>
```

If you cannot state the ambiguity precisely, the spike question is not specific enough.
Go back to Phase 0 and sharpen the question.

### 1.2 — Identify the right decision-maker

Functional spikes are validated by a human, not by a test passing. Identify who must
confirm the answer before the spike can close:

- Product owner / stakeholder: for business rule questions
- End user / user research: for UX flow questions
- Domain expert: for regulatory or compliance questions
- Developer: for implementation-boundary questions (what is technically expressible)

If the right person is unavailable within the timebox, the spike closes as `Partially` answered
and the confirmation is a Parking Lot item.

### 1.3 — Prototype scope

A prototype is required only when the ambiguity involves a UI/UX flow that must be
validated visually. In that case:

- Build the minimal visual mockup (static screens, not working code).
- Tools: Figma, pen-and-paper, or a static HTML/Flutter shell — match the fidelity
  needed to resolve the ambiguity, not production fidelity.
- The deliverable is the stakeholder's reaction to the mockup, not the mockup itself.
- Mark all prototype files: `// SPIKE PROTOTYPE — ARCHIVED — NOT FOR PRODUCTION`

If the ambiguity can be resolved by reading docs, interviewing a stakeholder, or writing
acceptance criteria — no prototype is needed.

### 1.4 — Document the decision

The finding is the decision, not the discussion. Structure each finding as:

```
FN — <Decision title>
Decision: <what was decided>
Decided by: <role / name>
Rationale: <one sentence — why this interpretation over the alternatives>
Implication: <what changes in the implementation or story definition>
```

---

## 2. Definition of "Question Answered"

For a functional spike, the question is answered when:

1. The ambiguity identified in §1.1 is resolved by a documented decision (not an inference).
2. The decision-maker (§1.2) has confirmed the decision — verbally, in writing, or by
   approving the prototype.
3. The resolved behaviour is expressed as testable acceptance criteria that can be added
   to the user story.

**The spike is NOT answered by:**
- A prototype that "looks good" without stakeholder validation.
- An interpretation that seems reasonable but was not confirmed.
- A design that is technically feasible but not confirmed to be what stakeholders want.

---

## 3. Output — Acceptance Criteria

The primary deliverable of a functional spike (beyond the findings doc) is a set of
acceptance criteria ready to be added to the parent user story.

Format:

```
Given <context>
When <action>
Then <observable outcome>
```

Write one criterion per resolved ambiguity. These go in the findings doc under a dedicated
"Acceptance Criteria" section and are cross-referenced from the recommendation.

---

## 4. Refined Estimate

After a functional spike, the estimate is for the story that was unblocked. Confidence
is typically higher than before the spike because the acceptance criteria are now defined.
State the key assumptions the estimate depends on (remaining technical unknowns, dependency
on other stories, etc.).
