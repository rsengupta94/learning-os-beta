# /synth — Pass 2 Specification

**Owner:** Rajarshi Sengupta
**Status:** Pass 2 design (draft).
**Relates to:** §3.2 of `product_spec.md` (high-level idea); this file carries operational detail.

---

## 1. Purpose & framing

`/synth` is **the learning partner.** A single-mode command (no Phase A/B split — `/ideate` handles ideation separately). Its job: build deep, grounded understanding of the topic at hand through didactic, conversational, bite-sized dialogue.

The discipline in one line: **agent teaches in small chunks, conversationally, calibrated to the user's stated comfort, with every claim provenance-tagged.**

`/synth` is **active** — it leads with content, identifies what fundamentals are needed, surfaces uncertainty, suggests capture moments, asks permission to research. The user controls the flow (where to go, when to back up, when to capture, whether to research) but the agent drives the teaching.

---

## 2. Invocation

```
/synth
```

Triggered from inside a topic folder (`_topic.md` present). Same `cwd` discipline as other commands: halts and asks if `cwd` is repo root or domain folder; warns if archived topic.

Optional focus hint:
```
/synth focus on attention mechanisms
/synth pick up where we left off
```

Without a hint, the agent uses session state + topic calibration to decide where to start.

---

## 3. Session start

The agent reads, in order:

1. **`_topic.md`** — topic goal, intended output, calibration section (declared comfort levels).
2. **Most recent 1–2 `_sessions/[YYYY-MM-DD-HHMM].md`** logs — what was covered, open questions, next-time hooks.
3. **`synthesis/[concept].md` files** — what's been captured so far (to avoid re-teaching).
4. **`absorbed/` items** — source content available to teach from. Read each `absorbed/[slug]/summary.md`; originals at `absorbed/[slug]/original.{ext}` are available for deeper reference if needed.

After reading, two paths:

### 3.1 First `/synth` session of a topic — topic-onboarding flow

If no prior session logs and no captured synthesis notes, the agent enters **topic-onboarding**:

1. Identifies the **3–7 critical fundamentals** for the topic (judgment-based count, not fixed). Driven by agent's parametric knowledge of the field + the topic goal in `_topic.md` + any absorbed content.
2. Surfaces them for user calibration:

   > *"For [topic], the foundations I'd typically teach with are:*
   > ***A. [foundation]** — [one-line description].*
   > ***B. [foundation]** — [one-line description].*
   > ***C. [foundation]** — [one-line description].*
   > ***D. [foundation]** — [one-line description].*
   > *Where are you on each — comfortable, shaky, or new? You can also tell me about adjacent things you know that I haven't listed. (Comfort here means applied-builder level — able to use the concept in building, debugging, explaining; not researcher mastery.)"*

3. User declares comfort levels + any adjacent context.
4. Agent **writes calibration to `_topic.md`** (express input, recorded permanently for the topic).
5. Agent proposes a teaching starting point given the calibration: *"given you're comfortable with A and C, shaky on B, and new to D, let's start by filling B's gap, then we move into D fresh. Sound right?"*
6. User confirms or redirects.

### 3.2 Subsequent `/synth` sessions — resume flow

If session logs exist, the agent opens with continuity:

> *"Last session we covered [X]. You had two open threads: [Y] and [Z]. The session log notes we were going to explore [Y] next. Pick up there, or shift?"*

User chooses. Agent proceeds.

If the session log has captures pending (things discussed but not committed), agent surfaces: *"There were also two captures pending from last time — [A] and [B]. Confirm-and-capture, or let go?"*

---

## 4. Teaching mode

### 4.1 Depth target — applied builder competence

`/synth` teaches to **applied-builder depth**, not researcher or exam-ready depth. The intent is solid grasp for someone building applied AI products — enough to:

1. **Build with the concept** — recognize it in code, system diagrams, library docs.
2. **Debug with the concept** — reason about why it might misbehave; identify likely causes; know which knobs matter.
3. **Explain the concept** — to another applied builder, peer-level.
4. **Choose with the concept** — know when to use it vs. alternatives; understand tradeoffs.

If all four are met, depth is sufficient. **Going further is out of scope unless user asks.**

The line in concrete terms (using attention as example):

| In scope (applied-builder) | Out of scope (researcher) |
|---|---|
| What attention computes (Q·K dot product, softmax, weighted V) | Deriving attention equations from first principles |
| Why positional encoding is needed alongside | Comparing 6 academic frameworks for thinking about attention |
| Quadratic cost — practical implication for context length | Proving the O(n²) bound formally |
| When to use sparse/linear variants in practice | Theoretical guarantees of sparse attention approximations |
| Why attention fails in long contexts (intuitively) | Edge cases that only matter for novel research |

**Bidirectional dynamism (handles the fuzzy line gracefully).** Applied-builder is a *default*, not a ceiling or floor. User can override either direction via verbal trigger (§4.6):

- *"go deeper"* / *"show me the math"* → agent goes beyond default.
- *"this is too deep"* / *"pull back"* → agent dials back.

Depth preference is **session-scoped** — once signaled, agent applies it for the rest of the session. Resets to applied-builder default at next session start. Not persisted to `_topic.md` (depth is a current-preference axis, distinct from skill calibration).

### 4.2 Style discipline

**Didactic + conversational + bite-sized.** Three coupled rules:

- **Didactic.** Agent leads with content. It explains. It uses its own knowledge, absorbed sources, and (with permission) external research to teach. Not Socratic — the user is not asked to discover the answer.
- **Conversational.** Agent pauses after each chunk and invites response. User interjects at any time — questions, clarifications, push-back, *"back up,"* *"move on,"* *"capture this."* Agent responds, then continues the teaching path.
- **Bite-sized.** Each turn of teaching is **one concept, one mechanism, one example at a time**. No info dumps. Pacing checks (*"does that land?"*, *"ready for the next bit?"*) are flow markers, not Socratic prompts.

The voice: a senior practitioner explaining clearly to someone learning, calibrated to their level, adjusting on the fly.

### 4.3 Prereq handling — three levels

All three are **explicit-user-permission**. User controls teaching flow throughout.

#### Level 1: Topic-onboarding (covered in §3.1)

Identifies critical fundamentals for the whole topic at first session. Calibration captured in `_topic.md`.

#### Level 2: Per-concept fundamentals check (at start of new concept)

Before teaching a new concept within a session:

> *"To teach [concept], I'll assume [X] and [Y] — any gaps?"*

User confirms or flags. If gaps: agent fills them first before proceeding to the concept.

This is finer-grained than topic-onboarding. It catches the per-concept prereqs that aren't visible at topic level.

#### Level 3: Mid-teaching dynamic backtrack (sub-prereq discovered)

Mid-explanation, agent realizes a sub-prereq is needed (one not anticipated at concept start):

> *"To make sense of this, you'd want [sub-prereq] — want me to back up?"*

User decides. Rhythm break is acceptable; missing prereq is worse. Agent announces the backtrack so the user understands the detour.

After backtracking and teaching the sub-prereq, agent returns to the original thread: *"OK, back to [original concept] — now we can see why [point that needed sub-prereq]..."*

### 4.4 Provenance tagging (always-on)

Every claim taught carries one of the five tags per `specs/verifier.md` §3:

- `[ABSORBED]` — from corpus
- `[RESEARCH]` — from this session's external research
- `[INFERENCE]` — agent's reasoning
- `[MODEL-STABLE]` — foundational model knowledge
- `[MODEL-UNCERTAIN]` — uncertain model knowledge (hallucination-prone territory)

Tagging is automatic — no permission needed. User sees tags inline as part of teaching.

### 4.5 Research-permission ask (batched, per-claim user decisions)

When the agent teaches `[MODEL-UNCERTAIN]` claims, **all of them — load-bearing or not — get surfaced for research permission**, not auto-passed.

Pattern: agent batches uncertain claims from a teaching segment and asks at a natural pause:

> *"Among what I just covered, three claims are `[MODEL-UNCERTAIN]`:*
> *(1) [claim] — load-bearing for the next part.*
> *(2) [claim] — supporting detail.*
> *(3) [claim] — supporting detail.*
> *Research any? Say all, none, or pick."*

User responds: all / none / selective. Each authorized claim → its own research fire, processed independently. Denied claims → proceed with `[MODEL-UNCERTAIN]` tag visible.

Frequency: ask fires at **natural pauses** (end of teaching arc, before next sub-concept), not after every chunk. Agent uses judgment on when to batch.

See `specs/research.md` for the research mechanism.

### 4.6 Verbal triggers in `/synth`

The user can interject at any time with these verbal patterns:

| Pattern | Effect |
|---|---|
| *"capture this"* / *"save this"* / *"note this"* | User-triggered capture (§5) |
| *"back up"* / *"earlier"* / *"I'm lost"* | User-triggered backtrack |
| *"move on"* / *"next"* / *"got it"* | Proceed to next chunk |
| *"go deeper"* / *"more depth"* / *"show me the math"* / *"I want the derivation"* | Agent goes deeper than applied-builder default (session-scoped) |
| *"this is too deep"* / *"pull back"* / *"I don't need this level"* / *"skip the derivation"* | Agent goes shallower (session-scoped) |
| *"keep it applied"* / *"applied builder depth"* | Resets depth to applied-builder default |
| *"verify that"* / *"check what you just said"* / *"is that right?"* | Fires verifier on preceding turn(s) (mid-dialogue check) |
| *"research X"* | User-triggered research (`specs/research.md` §2.1) |
| *"mark X as understood"* / *"I get X now"* | Express-input calibration update to `_topic.md` (§6) |
| *"mark X as shaky"* / *"I thought I knew X but I don't"* | Express-input calibration downgrade |
| *"wrap up"* / *"we're done"* / `/done` | Session close (§7) |

Agent recognizes intent + scope, confirms ambiguity, executes.

---

## 5. Capture

### 5.1 Trigger model — silent + selective + user-anytime

Two paths in, both with user final authority:

- **User-triggered (anytime).** User says *"capture this"* / *"save this"* / *"note this"*. Agent captures the concept(s) just discussed.
- **Agent-suggested (silent + selective).** Agent listens through dialogue; only interrupts with *"this feels like it crystallized — want me to capture it?"* when it judges a concept has truly come together. **High bar** to ask — not after every chunk, only at real crystallization moments. Maybe once or twice per session.

User can always say no. If user says yes (or initiates), agent proceeds to write the note.

### 5.2 Granularity

**Default: one concept per note.** Each `synthesis/[concept].md` is one focused unit.

**Closely-coupled concepts can share a single note** when they form a natural unit (e.g., *"attention + multi-head attention"*, *"MDPs + value functions + policy gradients"* as RL fundamentals). Agent asks at capture time: *"capture attention and multi-head attention together, or separately?"*

### 5.3 Output format

Capture writes to `topics/[cwd]/synthesis/[concept].md`. Format:

```markdown
---
concept: [main concept name, or "concept-A-and-B" if closely-coupled]
topic: [topic name from cwd]
captured_date: YYYY-MM-DD
captured_session: [YYYY-MM-DD-HHMM]
related_concepts: [optional list of closely-coupled or related concepts]
gate2_verdict: [PASS / SURFACED / TIGHTENED]   # populated by verifier
---

# [Concept name]

## Core idea

[1–2 paragraphs. The essence — what this concept is, why it matters. Provenance tags inline.]

**Claim:** [key claim]
[ABSORBED] · *"[verbatim quote]"*

## Mechanism / how it works

[How the concept operates. Step-by-step or structural explanation. Provenance tags inline.]

## Why it matters / what it enables

[The "so what" — what understanding this unlocks or enables.]

## Open questions

[Threads this concept raises but doesn't fully resolve. Feeds future `/synth`, `/ideate`, or research-keyword starting points.]

## Verifier annotations

[Inline verifier verdicts on specific claims — PASS, SURFACED-WITH-DISCLAIMER, TIGHTENED, or SUPPRESSED. Populated by Gate 2 at capture time.]
```

The frontmatter is machine-readable for `/ideate` and the advisor mode of `/absorb` to index. The body is human-readable.

### 5.4 Gate 2 fires at capture

Per `specs/verifier.md` §2 and §6: at the moment of write, sub-agent verifier runs on the note. Verdicts (PASS / SURFACED-WITH-DISCLAIMER / TIGHTEN-OR-DROP / FAIL: SUPPRESS) are applied. If any FAIL remains, write blocked until writer fixes. Audit log appended to `topics/[cwd]/verification.md`.

---

## 6. Calibration updates (express input only)

Mid-session, the user can update `_topic.md` calibration with express signals:

- *"mark X as understood"* / *"I get X now"* — upgrades concept X's calibration to comfortable.
- *"mark X as shaky"* / *"I thought I knew X but I don't"* — downgrades calibration.
- *"add Y to my calibration"* — adds a new concept the agent didn't ask about.

Agent updates `_topic.md` and acknowledges: *"updated — calibration for X is now [level]."*

**Agent does not auto-infer calibration changes from dialogue.** No *"you seem comfortable with X now, marking it understood"* without express input.

---

## 7. Session close

### 7.1 Trigger

Verbal pattern from user:

- *"wrap up"*
- *"we're done"*
- *"end session"*
- `/done`
- *"let's stop here"*

Agent confirms (*"wrapping up — writing session log"*) and proceeds.

### 7.2 Session log write

Agent writes `topics/[cwd]/_sessions/[YYYY-MM-DD-HHMM].md`:

```markdown
---
session_date: YYYY-MM-DD
session_time: HHMM
command: /synth
duration_chunks: ~N teaching chunks
---

# Session log — YYYY-MM-DD HH:MM

## What we covered

[One paragraph — concepts taught, depth, key turning points.]

## Captures committed

- [synthesis/concept-A.md] — [one-line summary]
- [synthesis/concept-B.md] — [one-line summary]

## Captures pending

[Things discussed but not yet committed. User can confirm-and-capture next session, or let go.]

- [pending capture] — [why it might be worth capturing]

## Calibration updates

[Express-input updates to _topic.md this session, for the record.]

- Marked X as understood
- Marked Y as shaky

## Open questions / pending threads

[Threads that emerged in dialogue but weren't resolved.]

## Next-time hooks

[What we were going to explore next, surfaced at next session start.]

- *"next time, explore Y"*
- *"we were going to dig into Z"*
```

Agent reads the log back to the user for review before finalizing. User can edit verbally (*"don't include the X capture, drop it"*) before the agent writes.

After the log is written, the session ends.

---

## 8. Non-capabilities

What `/synth` does NOT do:

- **No ideation.** Generating applied project candidates is `/ideate`'s job. If user starts brainstorming applications mid-`/synth`, agent gently redirects: *"sounds like an ideation thread — want to switch to `/ideate`?"*
- **No autonomous capture.** Even with the silent+selective discipline, agent asks before writing. No silent writes.
- **No autonomous calibration changes.** Only express-input updates.
- **No autonomous research.** Every research fire either user-triggered (keyword) or user-permitted (after agent ask). Exception: none in `/synth` — auto-fire is reserved for `/apply` per `specs/research.md` §2.3.
- **No writes outside `synthesis/`, `_sessions/`, and `_topic.md`.** `/synth` doesn't touch `absorbed/`, `applications.md`, or `projects/`.
- **No Socratic-style discovery teaching.** The user isn't asked to derive answers; the agent explains.
- **No info dumps.** Each turn is bite-sized; no multi-paragraph monologues.
- **No re-teaching of captured concepts** unless user explicitly asks. Agent checks `synthesis/` at session start to know what's already covered.

---

## 9. Grounding application

### Gate 1 (mandatory)

All five disciplines (per `product_spec.md` §5.2):

1. **Cite-and-quote** — claims from absorbed/research sources carry verbatim quotes.
2. **Provenance tags** — every claim tagged (§4.4).
3. **Read-before-write** — agent reads relevant absorbed items + prior synthesis before teaching a concept.
4. **Forced output structure for captures** — frontmatter + sections per §5.3.
5. **Distinguish summary from assertion** — *"the paper claims X"* not *"X is true."* For `[MODEL-STABLE]` and `[MODEL-UNCERTAIN]`, agent owns the claim (*"I believe..."*).

### Gate 2 (mandatory, fires at capture)

Sub-agent verifier runs on every captured synthesis note. Four verdicts per `specs/verifier.md` §6. Write blocked on FAIL.

### Mid-dialogue verification check (verbal, user-triggered)

User says *"verify that"* / *"check what you just said"* / *"is that right?"* — focused sub-agent verifier fires on immediately-preceding teaching turn(s). Output in chat, no `verification.md` write.

---

## 10. Edge cases

- **`cwd` not a topic folder** — halt, ask which topic.
- **Archived topic `cwd`** — warn, require confirmation.
- **No `_topic.md` calibration yet** — first session: trigger topic-onboarding.
- **Empty `absorbed/` and no prior sessions** — agent can still teach from `[MODEL-STABLE]` knowledge, but flags this: *"corpus is empty; I'll teach from model knowledge with extra `[MODEL-UNCERTAIN]` care. Consider absorbing core sources first via `/absorb`."*
- **User confirms understanding then immediately asks a basic question** — re-calibrate downward: *"OK, sounds like that didn't fully land — let me back up. Want to update calibration for [concept]?"*
- **Verifier blocks a capture (FAIL verdict)** — agent reports the failure, fixes or removes the problem claim, re-runs verifier, then writes.
- **User asks a question outside the topic** — agent gently redirects: *"this is more of a [other topic] question — want to spin up a new topic for it, or table it for now?"*
- **Long session, many captures pending** — at session close, agent surfaces all pending captures explicitly, user confirms/discards each.
- **Session interrupted (user disappears mid-dialogue)** — no session log on next resume; agent reads recent dialogue from prior context if available, or treats as fresh start. (Pass 2 detail: how to handle context-window loss across resumes.)

---

## 11. Tone & interaction

Voice: **senior practitioner explaining to a learner**, calibrated to comfort level.

- **No padding.** Don't pre-amble. Surface the teaching.
- **Applied-builder by default.** Default depth is build / debug / explain / choose competence. Adjust on user verbal feedback in either direction (§4.6) — session-scoped, not topic-scoped.
- **Calibrated within depth.** Beginner-at-applied = analogies, basic framings, *"the intuition is..."*. Expert-at-applied = precision, edge cases, peer-to-peer.
- **Reasoning visible.** Each claim has its provenance tag, so user knows where the knowledge came from.
- **Pacing-check, not Socratic.** *"Does that land?"* is a flow marker, not a "you guess" prompt.
- **Active suggestion-light.** Agent surfaces capture moments and prereq checks; doesn't constantly interrupt with offers.
- **User-deferential on flow.** Backtracks, captures, research fires — all gated on user permission.
- **Honest about limits.** *"I'm not sure about this — `[MODEL-UNCERTAIN]`"* is a first-class move, not a failure.

---

## 12. Residual risks

- **Wrong calibration → bad teaching depth.** If user mis-declares comfort (says "comfortable" when actually shaky), agent assumes too much, user gets lost. Mitigation: agent watches for confusion signals (clarifying questions out of expected difficulty) and offers calibration recheck. **Express-input downgrade always available.**
- **Mid-dialogue teaching errors before capture.** Verifier doesn't catch wrongs that aren't captured. Mitigation: writer discipline (Gate 1 tagging) + user verbal verification check (§9). Some errors will leak — user's reading is the ultimate gate.
- **Depth mismatch.** Applied-builder default may be too shallow for the moment (e.g., debugging a training issue needs deeper numerics) or too deep for the moment (e.g., quick orientation). Mitigation: bidirectional verbal feedback (§4.1 + §4.6) — user signals *"go deeper"* or *"pull back"*; agent adjusts session-scoped. The default is calibrated; the dynamism handles the rest.
- **Concept fatigue from long sessions.** Bite-sized teaching can feel slow when user is in flow. Mitigation: user can ask for denser/faster pacing; agent adjusts.
- **Over-batching of research-permission asks.** If too many `[MODEL-UNCERTAIN]` claims pile up, the batched ask becomes long. Mitigation: agent breaks into segments and asks at multiple natural pauses.
- **Capture suggestion fatigue.** If agent's selectivity is calibrated wrong, capture suggestions either over-fire (annoying) or under-fire (concepts get lost). Mitigation: Pass 2 prompt-engineering on the *"crystallization"* heuristic; user can adjust ("ask less often" / "ask more often").
- **Session log over-fitting to verbose conversations.** Long sessions produce long logs that defeat the *"concise, not transcript"* discipline. Mitigation: prompt discipline on session-log writing; user can edit verbally before finalize.

---

**End of `/synth` specification.**
