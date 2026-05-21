# /apply — Pass 2 Specification

**Relates to:** §3.4 of `product_spec.md` (high-level idea); this file carries operational detail.

---

## 1. Purpose & framing

`/apply` is **the bridge from idea to build.** Converts a chosen candidate from `synthesis/applications.md` into a verified, executable plan captured as `projects/[candidate-slug]/spec.md` — so downstream build sessions execute without re-litigating decisions or building on hallucinated specifics.

Voice: **practical builder — concrete, technical, verifying, scope-disciplined.**

**The user is not an engineer.** `/apply` does the technical heavy lifting: proposes the stack, architecture, data shapes, interface contracts, test plan, build phases, done states, risks. The user reviews reasoning and retains scope/product authority; the agent owns engineering proposals. **Trust comes from three sources:** every technical claim is auto-fire verified, every choice carries visible reasoning, every uncertainty is named honestly.

**The discipline in one line: `/apply` is detailed on *WHAT* to build; gives the build session a free hand on *HOW*.**

| Spec'd in `/apply` (WHAT) | Left to build session (HOW) |
|---|---|
| Stack: libraries, versions, APIs, models | Exact code organization, file structure |
| Architecture: components, data flow, integration | Internal component structure |
| Data shapes: schemas at boundaries | Internal data structures within a component |
| Interface contracts: function signatures at boundaries | Internal function signatures |
| Test plan: scenarios + what "verified" means | Test framework choice, mock/stub strategy |
| Build phases + done states | File-level build order within a phase |
| Acceptance criteria | Implementation patterns, error message wording |

Three load-bearing constraints:

1. **Candidate-anchored.** Operates on one captured candidate at a time. Doesn't re-ideate; doesn't propose alternative candidates. Selection happened in `/ideate`. *(Reads in §3; flow in §4.)*
2. **Auto-fired live-doc verification.** Technical claims (libraries, APIs, model IDs, endpoint URLs, framework versions) trigger immediate research without permission ask. Locked exception per `specs/research.md` §2.3 — the cost of skipping is build failures. *(Mechanics in §4.1 rule 3; verifier check in §5.3.)*
3. **Buildable output.** The spec must be self-contained enough for a separate build session to execute from it. *(Output format in §5.2. Build itself is outside the tool's scope — the user opens a separate agent session elsewhere, references the spec, and builds. How exactly that handoff happens is per-build user choice, not a tool concern.)*

---

## 2. Invocation

```
/apply [candidate-slug]    # if you know which candidate
/apply                     # agent asks which from applications.md
```

Triggered from inside a topic folder. Same `cwd` discipline as other commands: halts if `cwd` is repo root or domain folder; warns if archived topic.

---

## 3. Session start

The agent reads, in order:

1. **`synthesis/applications.md`** — locates the chosen candidate (or surfaces choices if invoked without slug).
2. **`_topic.md`** — topic context.
3. **`synthesis/[concept].md` files** — technical understanding to draw on.
4. **`absorbed/` items** — source content referenced by the candidate.
5. **Existing `projects/[candidate-slug]/spec.md`** if it exists — resume mode.
6. **Most recent `_sessions/` log** from `/apply` if any — for continuity.

If no `applications.md` or no candidate matching the slug, halt and ask.

---

## 4. Spec authoring

### 4.1 Authoring discipline

Four coupled rules:

- **Agent drives technical proposals.** Agent proposes specific stack, phases, done states, risks — with reasoning. Doesn't punt engineering decisions to user (*"what library do you want?"* / *"how should we break this into phases?"*). Brings engineering judgment to the table; user reviews. User retains scope, product, and override authority; agent owns the technical proposals.
- **Concrete specification required.** Exact library names + versions, exact API endpoints, exact data schemas, exact model identifiers. *"Use a vector DB"* gets rejected; *"Use FAISS v1.8.0 because [reason]"* is the bar. Abstract placeholders are scope creep in disguise.
- **Live-doc verification auto-fires.** Whenever a specific technical claim is about to commit to the spec (library, API signature, model ID, endpoint, version), agent fires external research without permission ask. Findings surface inline; spec commits only what's verified. (Locked exception to user-permission research per `specs/research.md` §2.3.)
- **Bounded scope.** Each spec has explicit out-of-scope. When scope creeps mid-session, agent flags and offers a path: move to out-of-scope, push to phase 2, or accept and revise the done definition. Push-back on sprawl is constructive — preserves shippability, not opposition.

### 4.2 Development flow

Conversational, but **agent drives proposals** at each step. User reviews, refines, and approves.

1. **Confirm candidate.** Re-read the chosen candidate from `applications.md` (strengths, weaknesses, requirements, criteria evaluation). User confirms scope.
2. **Propose technical stack.** Specific tools/libraries/APIs/models with reasoning. Each specific claim → auto-fired live-doc verification → findings inline → commit only if verified.
3. **Propose architecture / system design.** Components (what entities exist), data flow at system level, integration points with external services, major architectural choices (REST vs. event-driven, sync vs. async at boundaries, etc.). NOT internal component structure — that's build-session's call.
4. **Propose data shapes.** Schemas at major boundaries — inputs, outputs, persistent data (DB tables, vector dimensions), key data types. NOT internal data structures within components.
5. **Propose interface contracts.** Where components meet — API endpoints + request/response shapes, function signatures at module boundaries, event/message formats. NOT internal function signatures.
6. **Propose build phases.** Shippable phases (MVP → enhancements) with reasoning. Each phase has a concrete done state. NOT file-level build order within a phase.
7. **Propose test plan.** Scenarios per phase, what "verified working" means, key edge cases. NOT framework choice or mock/stub strategy.
8. **Propose acceptance criteria.** What "this is built" means overall — concrete, verifiable.
9. **Propose out-of-scope.** Agent identifies likely sprawl points; user adds, removes, revises.
10. **Surface risks and open questions.** Technical risks (data, integration, performance, scope), with honest uncertainty.
11. **Re-import requirements from candidate.** Candidate's requirements flow through.
12. **Settle and capture.** When sections feel complete, user says capture (or agent suggests).

Throughout: auto-fired research findings surface inline; reasoning behind each agent proposal is visible; uncertainties are named honestly; the WHAT/HOW line stays clean (no creeping into HOW).

### 4.3 Verbal triggers in `/apply`

| Pattern | Effect |
|---|---|
| *"capture this"* / *"save spec"* / *"finalize"* | User-triggered capture (§5) |
| *"verify this"* / *"check that library"* | User-triggered research on specific technical claim (beyond auto-fire scope) |
| *"narrow scope"* / *"split into phases"* | Refine scope (move to out-of-scope or phase 2) |
| *"this is out of scope"* | Move current item to out-of-scope list |
| *"add a phase"* / *"include X"* | Expand build phases |
| *"wrap up"* / *"we're done"* / `/done` | Session close (§6) |

---

## 5. Capture

### 5.1 Trigger model

Same pattern as `/synth` and `/ideate`: silent + selective agent suggest + user-anytime.

- **User-triggered.** User says *"capture this"* / *"save spec"* / *"finalize"*. Agent captures.
- **Agent-suggested.** Agent suggests *"the spec feels complete — capture?"* when all sections (stack, phases, done definition, out-of-scope, risks) have substantive content.

### 5.2 Output format

Capture writes to `topics/[cwd]/projects/[candidate-slug]/spec.md`. Format:

```markdown
---
candidate: [slug from applications.md]
topic: [topic-name from cwd]
captured_date: YYYY-MM-DD
captured_session: [YYYY-MM-DD-HHMM]
gate2_verdict: [PASS / SURFACED]
status: [draft / approved]
---

# [Project name]

## Overview

[1–2 paragraphs. What's being built; who it's for; what AI capability it demonstrates. Inherits from candidate's "What it is" section.]

## Topic relevance

[How this builds on the topic. Inherits from candidate's "How the topic plays in" section.]

## Technical stack

[Specific tools/libraries/APIs/models. Each line carries a verification annotation.]

- [Tool/library X v.Y] — [purpose in build] — VERIFIED [YYYY-MM-DD] · [URL]
- [API Z (endpoint)] — [purpose] — VERIFIED [YYYY-MM-DD] · [URL]
- [...]

## Architecture / system design

[Components and how they relate. Data flow at the system level. Integration points with external services. Major architectural choices (e.g., REST API vs. event-driven; sync vs. async at boundaries; service composition pattern).]

**Components:**
- [Component A] — [responsibility]
- [Component B] — [responsibility]

**Data flow:**
[Narrative or diagram (mermaid OK) of how data moves through the system end-to-end.]

**Integration points:**
- [External service / API] — [purpose, where it integrates]

**Major architectural choices:**
- [Choice + reasoning, e.g., "Synchronous request-response over a job queue for v1 because user-facing latency is acceptable and the queue adds operational complexity."]

## Data shapes

[Schemas at major boundaries. Inputs, outputs, persistent data, key data types. Concrete enough that a build session knows what to produce/consume.]

**Input shapes:**
```
[example schema for primary inputs]
```

**Output shapes:**
```
[example schema for primary outputs]
```

**Persistent shapes:**
- [DB table / collection / index] — [columns/fields, types, indexes]
- [Vector store schema] — [embedding dimensions, metadata fields]

**Key data types:**
- [Type name] — [structure, validation rules]

## Interface contracts

[Where components meet. Build sessions implement these contracts; they don't redesign them.]

**API endpoints (if applicable):**
- `POST /endpoint` — Request: `{schema}` → Response: `{schema}` — [purpose]

**Module boundaries:**
- `module_a.function_x(arg: Type) -> ReturnType` — [purpose, error behavior]

**Event/message formats (if applicable):**
- [Event name] — `{schema}` — [producer, consumer, purpose]

## Build phases

### Phase 1: MVP
[Concrete deliverable. Done state — what exists when this phase is finished.]

### Phase 2: [name]
[Optional or post-MVP work.]

## Test plan

[What gets tested per phase, what "verified working" means concretely. NOT framework choice — just what scenarios. Build session picks the testing framework / mock strategy.]

**Phase 1 verification scenarios:**
- [Scenario 1 — what's tested, what counts as pass]
- [Scenario 2]

**Edge cases to verify:**
- [Edge case — why it matters]

**End-to-end "verified working" definition:**
[What does "this MVP works" mean as a demonstrable check? E.g., "user can submit query X, system returns response Y within 2s, retrieval citations link to absorbed sources."]

## Acceptance criteria

[Concrete and verifiable. What does "built" mean overall (not just per-phase)?]

- [Criterion 1]
- [...]

## Requirements

[Inherited from candidate's Requirements section. What the build needs beyond core topic.]

- [...]

## Out of scope

[Explicit list of what's NOT being built. Sprawl-prevention.]

- [...]

## Risks and open questions

[Pre-build risks. Questions to resolve before building starts.]

- [...]

## Verification log

[Auto-fired research findings during spec authoring. Audit trail.]

- [YYYY-MM-DD] [tool/library X v.Y] at [URL] — [finding summary]
- [...]
```

### 5.3 Gate 2 fires at capture

Per `specs/verifier.md`: sub-agent verifier runs on the spec at write time. Specific checks for `/apply`:

- **Technical claims verified** — every library, API, model ID has a verification annotation from auto-fired research.
- **Stack concrete** — no vague *"use a vector DB"*; specific choices named with reasoning.
- **Architecture concrete** — components named, data flow specified, integration points identified, major architectural choices reasoned.
- **Data shapes specified** — major data types have concrete schemas, not *"TBD"* or hand-wave.
- **Interface contracts complete** — every component boundary has a contract.
- **Test plan substantive** — concrete scenarios, not *"test everything"*; "verified working" defined.
- **Phases shippable** — each phase has a concrete done state, not just a name.
- **Acceptance criteria verifiable** — concrete and testable, not vague aspirations.
- **Out-of-scope present** — at least one item; sprawl-prevention discipline.
- **Requirements inherited** — candidate's requirements appear in spec's requirements section.
- **WHAT/HOW line clean** — spec doesn't dictate file organization, internal code structure, or implementation patterns. Those are build-session's call.

---

## 6. Session close

### 6.1 Trigger

Verbal pattern: *"wrap up"* / *"we're done"* / *"end session"* / `/done`.

### 6.2 Session log

Writes `topics/[cwd]/_sessions/[YYYY-MM-DD-HHMM].md`:

```markdown
---
session_date: YYYY-MM-DD
session_time: HHMM
command: /apply
candidate: [slug]
---

# Session log — YYYY-MM-DD HH:MM

## What we developed

[One paragraph — what got worked on, key decisions.]

## Spec status

- [draft / approved / etc.]
- [File path if captured]

## Technical verifications fired

[Auto-fired research calls during the session.]

- Verified [X] at [URL] — [finding]
- Could not verify [Y] — flagged for user

## Open questions

[Things user needs to resolve before build starts.]

## Next-time hooks

[If spec isn't complete — what's pending for next session.]
```

---

## 7. Non-capabilities

- **No deep concept teaching.** That's `/synth`. Brief inline reasoning for technical choices is allowed (*"picking FAISS because [one-line reason]"*); going deep into the concept is `/synth` — agent suggests pivoting if user needs depth.
- **No re-ideation.** Doesn't propose alternative candidates. Selection happened in `/ideate`.
- **No autonomous capture.** Always asks before writing the spec.
- **No skipping verification.** Auto-fire is non-disabling. (User can request deeper verification, can't disable.)
- **No vague stack.** *"Use a database"* is rejected; specific choice with reasoning required.
- **No phase without done state.** Each phase must have a verifiable done definition.
- **No HOW prescriptions.** Doesn't specify file organization, internal code structure, framework-specific implementation patterns, error message wording, variable naming, etc. Those are the build session's free hand during build (whichever CLI coding agent runs it).
- **No writes outside `projects/[candidate-slug]/spec.md` and `_sessions/`.**
- **No build execution.** Spec is the artifact; build happens in a separate agent session, in a separate directory, outside the tool's scope.

---

## 8. Grounding application

### Gate 1 (mandatory)

All five disciplines apply. Provenance tagging especially important on technical capability claims.

### Gate 2 (mandatory, fires at capture)

Sub-agent verifier per §5.3.

### Auto-fired research (mandatory)

Per `specs/research.md` §2.3 — technical claims trigger live-doc verification without permission ask. Findings surface in dialogue; persisted to spec's verification log.

---

## 9. Edge cases

- **No `applications.md`** — halt: *"no candidates captured yet; run `/ideate` first."*
- **Candidate slug not found** — list available candidates from both buckets, ask user to pick.
- **Candidate is from adjacent bucket** — warn: *"this candidate routes to adjacent — topic isn't load-bearing for the current topic. Continue with `/apply` here, or revisit under a topic where it would be load-bearing?"* User decides.
- **Existing `projects/[slug]/spec.md`** — resume mode; offer to extend or rewrite.
- **Live-doc verification fails** (URL doesn't exist, library not found, API doesn't support claimed feature) — agent reports honestly; user decides to pick alternative, drop the claim, or override knowing the risk.
- **Verifier blocks capture (FAIL)** — agent reports, fixes or removes failing claim, re-runs.
- **User asks for ideation mid-session** — redirect: *"that's `/ideate` territory; want to capture current spec state and pivot, or stay on `/apply`?"*

---

## 10. Tone & interaction

Voice: **practical builder — concrete, technical, verifying, scope-disciplined.**

`/apply` is execution-focused and **agent-driven on technical proposals.** Less generative than `/ideate`, less coaching than `/synth`. The user is not an engineer; the agent does the technical heavy lifting and earns trust through verification + visible reasoning + honest uncertainty. Push-back on sprawl is constructive — preserves shippability, not opposition.

All capture and refinement decisions gated on user input. The one exception to user-permission is auto-fired live-doc verification on technical claims (locked per `specs/research.md` §2.3).

### Two behavioral traits

**1. Reasoning visible.**
Every technical choice carries its why — library reason, phase reason, scope reason. Trust comes from seeing the reasoning, not from the agent claiming authority. User reviews the thinking, not just the output. If reasoning is shaky, agent says so rather than dressing it up.

**2. Honest uncertainty.**
Agent flags areas of low confidence (*"I'm not sure if X scales for your data size"*), technical risks (*"this approach has trade-off Y"*), and verification failures (*"couldn't verify Z, flagging for review"*). Doesn't paper over what it doesn't know. Better to surface uncertainty and let user decide than commit to a confident-sounding wrong choice.

---

## 11. Residual risks

- **Auto-fire over-classification.** Agent treats too many things as "technical claims" and fires research excessively. Mitigation: trigger scope is narrow — only library names, API signatures, model IDs, endpoint URLs, framework versions. Drift is a Pass 2 prompt-engineering concern.
- **Stale verification.** Verified-now may not be verified-later (libraries version-bump, APIs change). Mitigation: verification log includes date; spec `status` field tracks freshness; build sessions re-verify before executing.
- **Vague-stack drift.** Agent might let abstract placeholders pass. Mitigation: verifier check (§5.3) rejects vague stack items at capture.
- **Phase-without-done.** Agent might list phases without verifiable done states. Mitigation: verifier check (§5.3).
- **Out-of-scope amnesia.** Build sessions might ignore the out-of-scope list and let scope drift. Mitigation: the spec's out-of-scope section is prominent and self-contained; build sessions reading the spec see it explicitly. User discipline (not tool enforcement) is what keeps the build aligned.
- **Candidate drift during authoring.** Agent might drift from candidate's original scope. Mitigation: §4.2 step 1 re-confirms scope; verifier checks alignment with candidate.
- **Build-session mechanics are outside scope.** The tool produces `spec.md` and stops there. How the build session references the spec, where built code lives, whether stale claims get re-verified — all per-build user choice. Mitigation: spec format is designed to be self-contained, so a fresh build session reading it has everything it needs without external context.
- **False confidence.** User isn't an engineer — might trust agent's confident-sounding technical proposals without spotting that they're under-verified or built on shaky reasoning. Mitigation: agent's "honest uncertainty" trait (§10) forces explicit flagging of low-confidence calls; verifier check (§5.3) requires verified annotations on technical claims; verification log gives an audit trail user can spot-check.
- **Engineering blind spots.** Agent might miss technical issues a real engineer would catch (subtle integration bugs, performance gotchas, security implications). Mitigation: verifier checks via auto-fired research; user can request deeper verification at any time; spec status field tracks build-readiness. Honest truth: spec is *defensible*, not *guaranteed* — build sessions will surface what specs can't.

---

**End of `/apply` specification.**
