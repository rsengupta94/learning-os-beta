# Verifier — Pass 2 Specification

**Owner:** Rajarshi Sengupta
**Status:** Pass 2 design (locked).
**Relates to:** §5 of `product_spec.md` (which carries the high-level summary; this file carries the operational detail).

---

## 1. Purpose

A lightweight sub-agent verifier whose job is to **confidently classify each captured claim as correct** — across model knowledge, absorbed sources, and external research — so the user's upskilling never builds on a foundation of hallucination, drift, or hand-waving.

Model knowledge is a **primary teaching source, not a fallback.** The verifier supports it, does not punish it. The discipline in one line: **surface uncertainty, don't surface garbage.**

---

## 2. When the verifier fires

**At capture-time**, not session-close. The verifier reviews the artifact being committed to disk — not the in-flight dialogue.

| Command | Verifier fires on |
|---|---|
| `/absorb` | Not fired (Gate 1 only — single-item extraction) |
| `/research` | Each write to `synthesis/corpus-overview.md` |
| `/synth` | Each write to `synthesis/[concept].md` |
| `/ideate` | Each candidate appended to `synthesis/applications.md` |
| `/apply` | The `projects/[name]/spec.md` write |

Mandatory at every capture event. Not opt-in. If a claim fails verification, **the write is blocked** until the writer fixes it.

---

## 3. The five provenance tags

Every claim in any captured artifact carries one of these tags. The writer attaches them at generation time. The verifier uses them to determine *what kind* of verification applies.

| Tag | Meaning |
|---|---|
| `[ABSORBED]` | From a source in this topic's `absorbed/` folder |
| `[RESEARCH]` | From the agent's external research this session |
| `[INFERENCE]` | Agent's reasoning over `[ABSORBED]` or `[RESEARCH]` evidence |
| `[MODEL-STABLE]` | Foundational, well-established model knowledge — textbook material, stable for 2+ years, widely-taught |
| `[MODEL-UNCERTAIN]` | Model knowledge in hallucination-prone territory — recent work, specific numbers, named entities, version-specific APIs, attributions |

### 3.1 The boundary rule between `[MODEL-STABLE]` and `[MODEL-UNCERTAIN]`

Writer-agent rule:

> **Default to `[MODEL-STABLE]`** for: textbook concepts, mathematical foundations, well-established theory, broadly-understood mechanisms.
>
> **Promote to `[MODEL-UNCERTAIN]`** when the claim involves: specific paper findings, specific numbers / benchmarks, attributions (*"X invented Y"*), recent releases, version-specific APIs, model-version specifics, niche subfield details, or anything the writer feels itself hedging on internally.

Writer errs toward `[MODEL-UNCERTAIN]` when on the fence — it triggers more verification, which is the right side to err on for upskilling.

If the writer tagged `[MODEL-STABLE]` on something that's actually attribution-prone or recent, the verifier downgrades it to `[MODEL-UNCERTAIN]` and demands hedge / grounding.

---

## 4. The five-step check

For each claim in the artifact about to be written:

1. **Provenance check.** Does the claim have one of the five tags (§3)? Untagged → **FAIL** (writer must tag or remove).
2. **Cite-match.** For `[ABSORBED]`: re-open the cited source (`absorbed/[slug]/summary.md`, falling back to `absorbed/[slug]/original.{ext}` if deeper verification is needed); does it contain the quoted text verbatim? For `[RESEARCH]`: re-fetch the cited URL and check the quoted text. Quote drift → **FAIL**.
3. **Inference logic.** For `[INFERENCE]`: does the inference logically follow from the cited evidence? Trace the logic. Unsupported leap → **FAIL**.
4. **Drift / over-claim check.** Does the summary represent what the source actually says? Over-claiming, scope-creep, conflation → **FLAG** (writer downgrades or removes).
5. **Hand-wave check.** Vague hedges without specific backing (*"often," "typically," "it's known that"*) → **FLAG** (writer tightens or removes).

Per-tag checks layer on top — see §5.

---

## 5. Per-tag verification protocol

What "correct" requires varies by tag:

| Tag | What "correct" means |
|---|---|
| `[ABSORBED]` | Verbatim source match + no drift |
| `[RESEARCH]` | Re-fetched source still says it; URL still resolves |
| `[INFERENCE]` | Logic traces from cited evidence without unsupported jumps |
| `[MODEL-STABLE]` | Consistent with itself and with widely-known material; no common misconception present. **No external citation required.** |
| `[MODEL-UNCERTAIN]` | The hedge is intact, **OR** external corroboration is attached. The *uncertainty* is what's faithfully reported. |

`[MODEL-STABLE]` flows through verification cleanly when it's actually stable material — the model gets to teach freely, and the verifier only intervenes when something feels off (inconsistency, common misconception, mis-tagged).

---

## 6. Verdicts and treatment

Four possible verdicts → four treatments. The discipline: **surface uncertainty, don't surface garbage.**

| Verdict | Trigger | Treatment |
|---|---|---|
| **PASS** | Claim passes all applicable checks | Written as-is, with provenance tag visible in note |
| **FLAG: SURFACED-WITH-DISCLAIMER** | Substantive `[MODEL-UNCERTAIN]` claim, or claim needing external follow-up | Surfaced in note with disclaimer attached — user sees the claim *and* the epistemic flag |
| **FLAG: TIGHTEN-OR-DROP** | Vague hand-wave (regardless of confidence) | Writer must tighten or remove — vague filler doesn't teach |
| **FAIL: SUPPRESS** | Hallucination — fabricated number, citation, attribution; or untagged claim writer cannot tag | Removed from note. Logged in `verification.md` for transparency |

### 6.1 Why this division

- **Uncertainty is honest.** Disclaiming an uncertain claim still teaches. Suppressing it would create silent gaps in the user's conceptual base — worse than knowing-with-a-flag.
- **Hand-waving is not honest.** Vague filler dressed as teaching doesn't fill any gap. The user gets nothing from it. Force tightening or drop.
- **Hallucination is dishonest.** Disclaiming a fabrication still gives it credence. No disclaimer salvages this — suppress.

---

## 7. Disclaimer format

Inline alongside the claim in the captured note:

```markdown
**Claim:** Transformers handle long contexts better than RNNs largely because attention has no distance term.
[MODEL-UNCERTAIN] · VERIFIER: SURFACED — needs verification
*The intuition is supported by the structure of attention, but the specific comparison to RNNs may have nuances (e.g., LSTM gating, vanishing gradient specifics). Recommend external check before treating as established.*
```

The reader sees: what the claim is, that it's not externally verified, and *why* the verifier flagged it — so they can judge whether to dig in.

---

## 8. Verifier output

Two destinations per session:

1. **The captured note itself** — inline annotations on each claim (`PASS` / `SURFACED` / `TIGHTENED` / `SUPPRESSED`), with provenance tag visible.
2. **`topics/[name]/verification.md`** — appended audit log per session. Records: claims checked, verdicts assigned, what was suppressed and why. Serves as the trust ledger for the topic.

---

## 9. Mid-dialogue protection

Capture-time verification does *not* catch teaching errors that happen mid-dialogue before any capture. Two mitigations:

- **Writer discipline (Gate 1 in `/synth`).** Writer-agent is forbidden from teaching a concept without attaching provenance in the same turn. *"Let me check this,"* *"I don't know,"* and *"this is `[MODEL-UNCERTAIN]`"* are first-class moves.
- **User-triggered verification check.** A verbal keyword pattern — *"verify that"*, *"check what you just said"*, *"is that right?"* — fires a focused sub-agent verification pass on the immediately preceding teaching turn(s). Available on demand. You use it when something feels off. Same mechanism as Gate 2, scoped to the recent turn(s); output goes to chat (no `verification.md` write, since nothing was captured).

Even with both, some mid-dialogue errors leak through before capture. The user's own reading remains the final gate.

---

## 10. Codex fallback

Codex lacks Claude Code's native sub-agent invocation mechanism. Fallback: the writer-agent invokes a second-pass self-check using the verifier's adversarial system prompt within the same agent context.

Weaker than true sub-agent (shared context = higher shared-bias risk) but preserves the discipline structure. Documented in `AGENTS.md`.

---

## 11. Residual risk

Be honest about what this design does *not* catch:

- **Shared model bias on `[MODEL-STABLE]` claims.** If the writer and verifier both share a subtle wrong belief (rare for textbook material but possible), neither catches the error. The adversarial verifier prompt mitigates but doesn't eliminate.
- **Source-correctness.** If an absorbed paper is itself wrong, the verifier won't catch it — it verifies that the *teaching* is faithful to the source, not that the source is right.
- **Mid-dialogue errors** before any capture occurs (see §9).
- **Subtle misinterpretation.** Quote is verbatim but the writer's reading of it is wrong.

The user's own reading is the ultimate gate (§11.2 of `product_spec.md`).

---

**End of verifier specification.**
