# Risk Spike — Investigation Reference

Agent-read guidance. Read before Phase 1 on any risk spike.

A risk spike investigates a threat to the project — technical, legal, operational, or
architectural — to quantify the risk and validate at least one mitigation before committing
to a course of action. The output is a quantified risk and a validated mitigation path,
not just a list of concerns.

---

## 1. Investigation Checklist

### 1.1 — Define the threat precisely

Before investigating, state the threat as a falsifiable claim:

```
Threat: <specific thing that could go wrong>
Trigger condition: <under what circumstances does this threat materialise>
Impact if triggered: <what breaks, who is affected, magnitude>
Current probability: <estimated likelihood before mitigation — High/Med/Low + basis>
```

Vague threats ("performance might be an issue") cannot be investigated or mitigated.
Restate as specific claims ("the Firestore query on collection X may exceed 1 s under 10k documents")
before beginning.

### 1.2 — Enumerate threats before investigating any

List all candidate threats first. Then prioritise by `impact × probability`. Investigate in
priority order. Stop when the timebox is exhausted — lower-priority threats become Parking Lot
items. Do not investigate low-priority threats before high-priority ones.

### 1.3 — Prototype scope

A prototype is required when feasibility must be demonstrated, not just argued.
Examples where a prototype is mandatory:

- Performance claim: must be measured, not estimated.
- Integration claim: must connect the actual systems, not simulate.
- Compliance claim: must confirm against actual regulatory text, not paraphrase.

If the risk can be resolved by reading docs or reviewing existing evidence — no prototype needed.

### 1.4 — Validate at least one mitigation

The spike is not done when the risk is quantified. It is done when at least one mitigation
has been empirically validated — meaning it was applied in the prototype and the threat
was demonstrably reduced.

```
Mitigation tested: <what was changed>
Result: <measured outcome after mitigation>
Residual risk: <remaining exposure after mitigation>
```

If no mitigation can be validated in the timebox, the spike closes as `Partially` answered.
The threat is quantified but the mitigation remains hypothetical — label it explicitly.

---

## 2. Risk Matrix Template

Include this table in the findings doc for each threat investigated.

| | Low impact | Medium impact | High impact |
|---|---|---|---|
| **High probability** | Monitor | Mitigate immediately | Spike / stop work |
| **Medium probability** | Accept | Monitor | Mitigate |
| **Low probability** | Accept | Accept | Monitor |

Position each threat on the matrix. The recommendation follows from the cell.

---

## 3. Finding Template for Risk Spikes

```
FN — <Threat title>

**Threat:** <specific claim>
**Trigger condition:** <when does this materialise>
**Impact:** High / Medium / Low — <one sentence>
**Probability:** High / Medium / Low — <basis for estimate>
**Matrix position:** <cell from matrix above>

**Mitigation tested:** <what was changed or applied>
**Result:** <measured or observed outcome>
**Residual risk:** <remaining exposure>

**Root cause:** diagnosed | hypothesized
**Basis:** <what confirms the root cause>

**Recommendation:** Mitigate now / Monitor / Accept / Requires follow-on spike
```

---

## 4. Definition of "Question Answered"

For a risk spike, the question is answered when:

1. Every high-priority threat is quantified (probability × impact assessed with evidence).
2. At least one mitigation per high-priority threat has been empirically validated.
3. Lower-priority threats not investigated are explicitly listed in the Parking Lot.

The spike is NOT answered by a list of threats with no mitigation validation.
