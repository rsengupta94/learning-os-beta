# Learning OS — V1 Product Specification

**Role:** This is the source of truth for what we are building.

---

## 1. What the product is — at a glance

The Learning OS exists to **deepen understanding of AI developments and ship applied AI projects from that understanding** — the two outputs that anchor every design decision.

The product is four things:

1. **A directory of files** as the entire data layer (see §2).
2. **System spec — `CLAUDE.md` and `AGENTS.md`.** Each contains the Learning OS orientation, cross-cutting disciplines (grounding §5, research §6, continuity §7, showcase criteria), command summaries, and non-negotiables. Auto-loaded by the respective agent at session start; kept in conversation context throughout. **Progressive disclosure** — operational detail per command lives in `specs/[command].md` and is loaded on command invocation, not duplicated in this file. Target: under 200 lines each (per Anthropic best practice). ~95% shared content between `CLAUDE.md` and `AGENTS.md`; ~5% diverges on platform-specific notes (Codex sub-agent fallback, web research tools). See §11.1.
3. **Four slash commands** — `/absorb`, `/synth`, `/ideate`, `/apply` (see §3). Plus *research* as a keyword-triggered + agent-introspective capability (see `specs/research.md`).
4. **The current working directory is the active topic.** No global state file, no topic-management command. `cd topics/[name]/` puts you in that topic.

No scripts, no scheduler, no schemas (frontmatter conventions only), no schema validation, no orchestration code, no MCP integrations, no custom skills — initially. Each can be added later if real usage shows the pain.

Build time: a weekend at most.

---

## 2. Directory layout

```
learning-os/                          # repo root
├── README.md                         # user-facing manual
├── CLAUDE.md                         # system spec for Claude Code (orientation + pointers to specs/; load-bearing — §11.1)
├── AGENTS.md                         # system spec for Codex + Codex-specific platform notes (user-written, pending)
├── product_spec.md                   # this file — design history + source of truth
├── .gitignore                        # OS/editor cruft only
├── .claude/commands/                 # slash command bodies (auto-loaded by Claude Code on invocation)
│   ├── absorb.md
│   ├── synth.md
│   ├── ideate.md
│   ├── apply.md
│   └── done.md                       # utility: signals session close, writes session log
├── specs/                            # full operational specs (read on command invocation per progressive disclosure)
│   ├── verifier.md                   # Gate 2 sub-agent verifier protocol
│   ├── absorb.md
│   ├── research.md                   # research capability (keyword + agent-introspective + /apply auto-fire)
│   ├── synth.md
│   ├── ideate.md
│   └── apply.md
├── topics/
│   ├── [domain]/                     # optional grouping folder; no _topic.md of its own
│   │   ├── [topic-name]/             # topic (contains _topic.md)
│   │   │   ├── _topic.md             # goal, intended output, calibration
│   │   │   ├── _inbox/               # filesystem-drop staging (user drops files here; /absorb processes & clears)
│   │   │   ├── _sessions/            # session logs (§7.1)
│   │   │   │   └── [YYYY-MM-DD-HHMM].md
│   │   │   ├── absorbed/             # one folder per absorbed item (summary + original)
│   │   │   │   └── [YYYY-MM-DD]-[slug]/
│   │   │   │       ├── summary.md    # agent's extraction
│   │   │   │       └── original.{ext}  # PDF / image / text / transcript — preserved
│   │   │   │       # for URLs: source.md (URL + access date + fetched content as markdown)
│   │   │   ├── synthesis/
│   │   │   │   ├── [concept].md
│   │   │   │   └── applications.md
│   │   │   │   # corpus-overview.md created on-demand by /synth
│   │   │   │   # topic-summary.md created at archival (consolidation pass)
│   │   │   ├── projects/
│   │   │   │   └── [name]/
│   │   │   │       └── spec.md
│   │   │   └── verification.md
│   │   └── [sibling-topic]/          # siblings share the domain folder
│   ├── [standalone-topic]/           # standalone — no parent domain
│   │   └── ...                       # same topic structure
│   └── archived/
│       ├── [domain]/[old-topic]/     # archived under its domain
│       └── [old-standalone-topic]/
└── inbox/                            # optional repo-root: un-topiced captures awaiting triage
```

Topic switching and archival are filesystem operations: `cd topics/X`, `mv topics/X topics/archived/`. **Topic creation has two paths:** manual (`mkdir` + write `_topic.md`) or agent-assisted (`/absorb` bootstrap mode — from repo root or a domain folder with content, agent asks 2–3 questions and creates the structure). No commands needed for state tracking — the directory you're in *is* the active topic; slash commands write relative to `cwd`.

**Topic vs. domain rule.** A folder is a *topic* if it contains `_topic.md`. A folder that holds topic folders but has no `_topic.md` of its own is a *domain* — pure grouping, no synthesis. Two-level hierarchy only (domain → topic, no deeper). Cross-topic synthesis happens at retrospective time, not as a built-in concept. If `cwd` is a domain folder + the user runs `/absorb` with content, the agent enters bootstrap mode to create a new topic under that domain. Other commands or `/absorb` without content from a domain folder → halt and ask which topic.

---

## 3. The four slash commands

This section captures the **core idea** of each command. Full operational detail (capabilities, disciplines, tone, verbal triggers, output format, edge cases) lives in `specs/[command].md` — linked in each subsection.

External research is not a command but a keyword + agent-introspective capability available from any command — see §6 and `specs/research.md`.

### 3.1 `/absorb`

**Idea:** Universal entry point with three modes dispatched by `cwd` + content:
- **Bootstrap** (repo root / domain folder + content): agent asks 2–3 minimal questions, creates topic structure, writes `_topic.md`, proceeds with extraction.
- **Extraction** (in a topic + content): ingests content into `absorbed/[slug]/` as both **summary.md** (agent's extraction) AND **original** (preserved file or fetched snapshot). Two patterns: Pattern A drops content in chat; Pattern B drops files into the topic's `_inbox/` and runs `/absorb`.
- **Advisor** (in a topic, no content): analyzes corpus shape and surfaces gaps, conflicts, source-set health, open questions. Conversational, no file writes.
**Inputs:** URL, PDF, image, plain text, video transcript. Unknown types fall through to a low-confidence attempt — agent flags and asks before writing.
**Output:** `topics/[cwd]/absorbed/[YYYY-MM-DD]-[slug]/` containing `summary.md` + `original.{ext}` (or `source.md` for URLs).
**Grounding:** Gate 1 only.
**Full spec:** `specs/absorb.md`.

### 3.2 `/synth`

**Idea:** The learning partner. Didactic, conversational, bite-sized dialogue to build **applied-builder depth** understanding of the topic — solid grasp for building / debugging / explaining / choosing, not researcher or exam-ready depth. Depth is bidirectional: user can verbally signal *"go deeper"* or *"pull back"* and agent adjusts session-scoped. At topic onboarding (first `/synth` session of a topic), actively identifies critical fundamentals and asks user about comfort on each — calibration captured in `_topic.md`. Captures synthesis notes on explicit command. Can backtrack to teach prerequisite foundations when gaps surface. May ask permission to research when knowledge gaps surface.
**Inputs:** None (reads `_topic.md`, recent `_sessions/`, `absorbed/`, `synthesis/`).
**Outputs:** `synthesis/[concept].md` notes. Express-input updates to `_topic.md` calibration section.
**Grounding:** Gate 1 + Gate 2 at each synthesis-note capture (see `specs/verifier.md`).
**Full spec:** `specs/synth.md`.

### 3.3 `/ideate`

**Idea:** A careful ideation partner — **senior techno-product applied AI builder** voice; generative, grounded-skeptical, critically honest, non-adversarial. Tech-first priority (technical capability and feasibility leads; product judgment applies where it fits, not as a filter on concept demos). Both **concept demos** (notebooks, libraries, repos demonstrating capability) and **applied products** are valid candidates. **Topic-anchored as routing**: every candidate evaluated by the load-bearing test — passes route to main bucket, failures route to **adjacent bucket** with reasoning surfaced. **Unbounded by user state and no gap analysis** — does not read or ask about user skill/knowledge; does not evaluate whether user has the requirements. May fire research to validate technical feasibility. Each candidate has three sections: **strengths** / **weaknesses** (of the idea) / **requirements** (pure factual list). Plus **qualitative paragraph evaluation** against five criteria (topic-anchored / realistic / externally useful / recent AI capability / bounded scope) — written reasoning, no scores.
**Inputs:** Optional focus. Reads current topic's `_topic.md` (excluding calibration section), `absorbed/`, `synthesis/` only — no other topic folders, no profile, no calibration.
**Outputs:** Candidates appended to `synthesis/applications.md` (two-bucket structure: topic-anchored and adjacent).
**Grounding:** Gate 1 + Gate 2 at each candidate write (see `specs/verifier.md`).
**Full spec:** `specs/ideate.md`.

### 3.4 `/apply`

**Idea:** **The bridge from idea to build.** Converts a chosen candidate from `synthesis/applications.md` into a verified, executable plan captured as `projects/[candidate-slug]/spec.md` — so downstream build sessions execute without re-litigating decisions or building on hallucinated specifics. **Discipline in one line: `/apply` is detailed on *WHAT* to build; gives the build session a free hand on *HOW*.** Voice: **practical builder — concrete, technical, verifying, scope-disciplined.** Because the user is not an engineer, `/apply` does the technical heavy lifting: proposes the stack, architecture, data shapes, interface contracts, test plan, phases, done states, risks. User retains scope/product authority; agent owns engineering proposals. Trust comes from auto-fire verification + visible reasoning + honest uncertainty. Live-doc verification on technical claims **auto-fires** — locked exception to user-permission research.
**Inputs:** Candidate name from `applications.md`. Reads candidate context (synthesis, absorbed) only — no skill state, no other topics.
**Outputs:** `projects/[candidate-slug]/spec.md`.
**Grounding:** Gate 1 + Gate 2 at spec write, incl. auto-fired live-doc verification on technical claims (see `specs/verifier.md` and `specs/research.md`).
**Full spec:** `specs/apply.md`.

---

## 4. The topic lifecycle — end-to-end walkthrough

A concrete pass through a topic, start to finish:

1. **Decide to study a topic.** Say, managed agents.
2. **Bootstrap the topic via `/absorb`.** From repo root (or a domain folder), drop your first content in chat — a URL, PDF, screenshot, or keyword note — and run `/absorb`. The agent asks 2–3 brief questions (*"what should I call this topic? standalone or under a domain? what's the goal?"*), creates the folder structure (incl. `_topic.md`, `_inbox/`, `absorbed/`, `synthesis/`, `_sessions/`, `projects/`), and proceeds with extraction on the content you provided. *(You can still `mkdir` manually if you prefer; agent bootstrap is the lower-friction path.)*
3. **Identify gaps and grow the corpus.** Run `/absorb` in advisor mode (no content) to get specific recommendations for what to look for. Find and absorb suggested items via Pattern A (paste/drop in chat + `/absorb`) or Pattern B (save files into the topic's `_inbox/` + `/absorb`). Or say *"research X"* at any point to have the agent fetch externally. The agent may also ask: *"this needs verification — should I research?"* — you decide.
4. **Continuous absorb.** Over days or weeks, drop interesting material in chat or save files to `_inbox/` and run `/absorb`. Each item lands in `topics/[cwd]/absorbed/[date]-[slug]/` as `summary.md` + `original.{ext}` (preserved for re-reference).
5. **Run `/synth`.** Dialogue with the agent as your learning partner. At first session, topic-onboarding identifies critical fundamentals and asks for your comfort on each — captured in `_topic.md`. Capture-on-command when understanding crystallizes. Express-input updates to `_topic.md` calibration as you mark concepts understood or as gaps surface.
6. **Run `/ideate`.** Engage as ideation partner. Shape and pressure-test candidate use-cases together. Promising candidates land in `synthesis/applications.md`.
7. **Pick a candidate.** Run `/apply [candidate-slug]`. Conversational shaping → `projects/[name]/spec.md`. Live-doc verification auto-fires for technical claims.
8. **Build.** *(Outside the tool's scope.)* Open a separate agent session (Claude Code, Codex, or any CLI coding agent) in a different directory and build from `spec.md`. Where built code lives, how the build session references the spec, how stale claims get re-verified — all decisions you make per-build. The tool's job ends at producing the spec artifact.
9. **Archive when done.** Run a final consolidation pass (mechanism TBD in Pass 2) producing `synthesis/topic-summary.md` — the consolidated-understanding artifact. Then `mv topics/managed-agents topics/archived/`.

Multiple topics run in parallel — each in its own agent session (any CLI coding agent), each cd'd into its own folder.

### 4.1 Layer transitions

Layer transitions are **fully user-driven** — you run the slash command for the layer you want. The system never auto-transitions or gates. Two soft mechanisms help orient you:

1. **State-aware suggestions.** Each command checks the topic's existing files at session start. If you run `/ideate` when `synthesis/` is empty, the agent notes the unusual order and offers to switch. Overridable.
2. **Next-time hooks.** Session logs (§7.1) include a *"next time"* line that surfaces at the next session start.

Neither enforces. Both inform.

---

## 5. Grounding architecture

Grounding to prevent hallucination, incorrect assumptions, and unverified claims is **baked into every layer, every step**. Two nested gates, both prompt-discipline (no code).

- **Gate 1 — Generation-time:** every claim is grounded at the moment it's written.
- **Gate 2 — Capture-time:** a sub-agent verifier checks each artifact at the moment it's about to be written to disk, classifying each claim as PASS, FLAG, or FAIL. Full protocol in `specs/verifier.md`.

If Gate 1 misses a claim, Gate 2 may catch it. If both miss, your own reading is the final gate.

### 5.1 Why prompt-discipline over alternatives

We explicitly *don't* build evaluator scripts, knowledge graphs, or MCP fact-checkers in V1. Three reasons: (1) zero build cost; (2) any code-based verification is only as good as its driving prompt; (3) we don't yet know which failure modes matter most — usage will tell us. Reach for code only when prompt-discipline demonstrably fails.

### 5.2 Gate 1 — Generation-time grounding

Five disciplines, encoded in `CLAUDE.md` / `AGENTS.md` and reinforced in each slash command body.

1. **Cite-and-quote, not cite-and-reference.** Every non-trivial claim has an inline reference *plus* a verbatim quote from the source. The agent can't fake a citation without producing source text — forces it to actually open the file.
2. **Provenance tags.** Every claim carries one of five tags:
   - `[ABSORBED]` — from a source in this topic's `absorbed/`
   - `[RESEARCH]` — from agent's external research this session
   - `[INFERENCE]` — agent's reasoning over the above
   - `[MODEL-STABLE]` — foundational, well-established model knowledge (textbook material; no external citation required)
   - `[MODEL-UNCERTAIN]` — model knowledge in hallucination-prone territory (recent, specific, attribution-prone; must hedge or be externally grounded)

   Model knowledge is a primary teaching source, not a fallback. Full tag definitions, boundary rule between the two `[MODEL-*]` tags, and per-tag verification protocol in `specs/verifier.md` §3 and §5.
3. **Read-before-write.** Each slash command opens with: *"Before writing anything, read [these specific files]."* Combats context drift.
4. **Forced output structure.** Captured artifacts (synthesis notes, candidates, specs, absorbed items) follow defined section structures per their respective specs — frontmatter + named sections, never free-form prose. Each claim within carries its provenance tag (and verbatim quote for `[ABSORBED]` / `[RESEARCH]` claims).
5. **Distinguish summary from assertion.** *"The paper claims X,"* not *"X is true."*

### 5.3 Gate 2 — Capture-time (sub-agent verifier)

At capture time — the moment a command is about to write a synthesis note, corpus overview, candidate, or spec to disk — a sub-agent verifier runs via a separate `Agent` invocation with an adversarial system prompt. It performs a five-step check on each claim and issues one of four verdicts: **PASS**, **FLAG: SURFACED-WITH-DISCLAIMER**, **FLAG: TIGHTEN-OR-DROP**, or **FAIL: SUPPRESS**.

The discipline in one line: **surface uncertainty, don't surface garbage.** Substantive uncertain claims are surfaced with disclaimers; hand-waving gets tightened or dropped; hallucination is suppressed.

**Mandatory at every capture event** across `/synth`, `/ideate`, `/apply`. Not opt-in. If a claim fails, the write is blocked until the writer fixes it.

**Output:** inline annotations on the note itself + an audit log appended to `topics/[name]/verification.md`.

**Codex fallback:** single-agent self-check with adversarial prompt within the same context. Documented in `AGENTS.md`.

Full protocol — five-step check, per-tag treatment, disclaimer format, mid-dialogue mitigations, residual risks — in `specs/verifier.md`.

### 5.4 How it bakes into each layer

- **`/absorb`:** Gate 1 only.
- **`/synth`:** Gate 1 + Gate 2 at each synthesis-note capture.
- **`/ideate`:** Gate 1 + Gate 2 at each candidate write.
- **`/apply`:** Gate 1 + Gate 2 at spec write (incl. auto-fired live-doc verification on technical claims).
- **External research capability:** source-priority enforcement (Tier 1–4) applies wherever research fires; see `specs/research.md`.

### 5.5 What it catches — and what it misses

**Catches:** fabricated citations, drift between source and summary, under-supported claims, API / library hallucinations in specs, stale assumptions, hand-waving dressed as teaching, mis-tagged `[MODEL-STABLE]` claims that should have been `[MODEL-UNCERTAIN]`.

**Misses (be honest):** plausible-but-wrong inferences sharing verifier blind spots; subtle misinterpretation where the quote is accurate but its reading is wrong; shared model bias on `[MODEL-STABLE]` claims; mid-dialogue teaching errors before capture; errors in the absorbed source itself. Your reading remains the ultimate gate. Mid-dialogue, verbally trigger a verification check (*"verify that"*, *"check what you just said"*) to fire a focused verification pass on the immediately preceding teaching turn.

---

## 6. External research — capability and quality controls

External research is **first-class**, but expressed as a **capability**, not a slash command. Two trigger paths and three places of use.

### 6.1 How research fires

- **User keyword.** Say *"research X"* in any chat session and the agent fires external search scoped to X, returning findings in chat.
- **Agent-introspective with permission.** When the agent recognizes its own knowledge boundary or a claim's hallucination risk, it asks *"this needs verification — should I research?"* You decide.
- **Auto-fire exception:** `/apply`-time live-doc verification on technical claims (libraries, APIs, model names) runs **automatically** — the cost of skipping is too high (build failures cascade).

Full spec: `specs/research.md`.

### 6.2 Where research fires

1. **`/synth` gap-fill** — when teaching surfaces a concept absorbed sources don't cover, agent asks permission to research and grounds the teaching.
2. **`/ideate` feasibility check** — agent can ask permission to validate technical feasibility claims for a candidate application.
3. **`/apply` technical verification** — auto-fires; verifies libraries / APIs / model names against live docs before spec ships.

(Advisor mode in `/absorb` does **not** trigger research — by design, it points toward conceptual gaps and source types rather than specific items, so there's nothing specific to validate. The user can always invoke the research keyword directly if they want a pointer fleshed out.)

### 6.3 Source priority order

When multiple sources cover the same claim, the agent prefers higher tiers.

- **Tier 1:** peer-reviewed papers, ArXiv preprints from established authors, official lab documentation, API official docs.
- **Tier 2:** lab blog posts (Anthropic, OpenAI, Google DeepMind, etc.), reputable technical newsletters, recognized industry research orgs.
- **Tier 3:** general technical blogs, well-known practitioner posts.
- **Tier 4 (last resort, flagged):** social media, anonymous blogs, model parametric knowledge.

### 6.4 Quality disciplines

- **Multi-source for important claims.** Single-source claims get flagged in synthesis notes.
- **Recency stamp on every external finding.** Agent flags items that look stale relative to AI's pace.
- **§5 grounding applies fully.** Cite-and-quote, `[RESEARCH]` provenance, verifier re-fetch at capture time.
- **Cross-validation against absorbed sources.** If external research contradicts an absorbed source, the agent flags the contradiction rather than picking a side.
- **Findings stay in chat by default.** Agent never auto-writes findings to `absorbed/`. User explicitly invokes `/absorb` on findings worth keeping. Preserves curatorial control over what enters the corpus.

---

## 7. Cross-session continuity

Topics span weeks to months. Sessions are intermittent. You won't keep windows open; you'll work on other topics in between. Session logs preserve context across long gaps.

### 7.1 Session logs

Every `/synth`, `/ideate`, `/apply` session ends by writing a brief log to `topics/[name]/_sessions/[YYYY-MM-DD-HHMM].md`:

- What we did this session (one paragraph)
- Open questions / pending threads
- Captures pending — things discussed but not yet committed as synthesis notes; you can confirm-and-capture or let go
- Next-time hooks (*"we were going to explore X"*)

At session start, the agent reads `_topic.md`, the most recent 1–2 session logs, plus the topic's existing artifacts. This provides conversational continuity even after a 2-month gap.

Discipline: logs are concise — not transcripts. Agent captures what mattered, not everything said.

`/absorb` extraction is single-transaction and skips logging. `/absorb` advisor mode is dialogue-based — its session-log discipline is defined alongside advisor mode in Pass 2.

### 7.2 Skill calibration

Calibration is **topic-local**, not global. Each topic's `_topic.md` carries a calibration section recording your declared comfort levels for the topic's fundamentals — set actively at topic-onboarding (first `/synth` session) when the agent identifies critical prereqs and asks about each.

**Critical rule: calibration updates by express input only.** The agent never infers understanding from dialogue and never auto-updates. The user explicitly says: *"mark X as understood,"* *"I'm shaky on Y, back up,"* *"downgrade Z — I thought I had it, I don't."* `_topic.md` reflects these signals only.

`/synth` reads `_topic.md` calibration at session start to adjust teaching: skips well-understood prereqs, fills declared gaps, calibrates depth, pace, and examples per stated comfort.

`/ideate` does **not** read calibration — it's unbounded by skill (§3.3).

Detailed `_topic.md` schema + parsing discipline defined at build time (Pass 2).

---

## 8. Durable principles

These rules govern every design decision.

### 8.1 Depth over breadth

The product is organized around **topics**, not themes or a global corpus. A topic is something you actively study for weeks, ending in deep understanding, a project, or both. With depth, you have a handful of active topics at a time — taxonomy is overkill, autonomous coverage is unnecessary.

### 8.2 The test for what stays

- A **capability** stays if it serves deepening understanding or shipping applied projects *directly*.
- A **mechanism** stays only if the capability would fail without it.
- Default state for everything: **cut**.
- Burden of proof: on inclusion, not on exclusion.

### 8.3 "Hacky" includes feature and capability cuts

Not just elegance vs. messy implementation. Features and capabilities can be cut outright if they aren't load-bearing for the WHY.

### 8.4 No vendor lock

Architecture must be portable between Claude Code, Codex, and any future CLI coding agent:

- Data layer pure: markdown files with simple frontmatter conventions.
- Any script (currently: none) talks to a generic "ask LLM" function.
- Two convention files (`CLAUDE.md` and `AGENTS.md`) carry top-level orientation + cross-cutting disciplines + pointers; ~95% shared content, ~5% diverges on agent-specific platform notes. Operational detail per command lives in `specs/*` and is loaded on command invocation (progressive disclosure). Each agent's convention file is auto-loaded by its conventions and stays in conversation context throughout the session.
- Don't lean on agent-specific UX as architecture. Slash commands are ergonomic shortcuts — every workflow must also be expressible in plain natural language.

---

## 9. What we explicitly cut

So the cuts don't drift back. If you find yourself wanting one of these, treat it as a signal to re-evaluate — not a green light.

- Autonomous cron / scheduled scanning.
- Ambient awareness across the AI landscape (covered by external aggregators + paste-on-encounter).
- Whitelist as a first-class concept.
- Autonomous pattern-finding / connection-candidate surfacing.
- Custom frontend / dashboard / review queue / nudges / notifications.
- Centralized data access layer / schema validation infrastructure.
- Multi-tier ingestion fallback.
- Multi-stage processing pipeline.
- Knowledge state schema with action routing.
- Build practices profile with versioning.
- Decisions log as a structured workflow.
- Retrospective capture with fragment-type schemas.
- Portfolio state file and showcase alignment mechanism.
- Three customization surfaces (chat / UI / file). Direct file edit only.
- Verbosity levels and per-section overrides.
- Standing priorities + temporal boosts.
- Discovery prompts for new sources.
- Per-source signal tracking, confidence-with-evidence representation, item-level feedback infrastructure.
- Schema migration mechanism.
- High-impact warning system.
- Spiderweb / interconnected-knowledge ambition.
- Code-based verification / evaluator scripts (deferred; §5 prompt-discipline instead).
- `/topic` command and active-topic state file (directory-as-state replaces).
- `/challenge` command (verification-on-demand handled as mid-session verbal pattern; no separate command).
- `/research` as a slash command (research is now a keyword-triggered + agent-introspective capability — see `specs/research.md`).
- Interview-preparedness mechanisms (out of WHY — focus is deepening understanding + shipping projects).
- Global `_profile.md` and cross-topic skill state (replaced by topic-local active calibration in `_topic.md`, set at topic-onboarding via `/synth`).
- Auto-updating skill calibration (express-input only — see §7.2).

---

## 10. What gets given up

Honest list of capabilities lost in the bare-minimum design.

- **No structured feedback signal accumulating.** Bad outputs get deleted; no aggregate data on failure modes.
- **No autonomous anything.** The system does nothing when you're not in a session.
- **No cross-topic indexing.** Archived topics are read-on-demand.
- **No portfolio dashboard or coherence check.** Ask Claude when you want one.
- **No code-based verification.** Grounding relies on prompt-discipline (§5).
- **No discovery of new sources.** You add them as you encounter them.
- **No within-topic update awareness.** During a topic, major releases get found via external aggregators or by accident.
- **No active-topic indicator beyond cwd.** Running a slash command from a wrong topic folder writes to that wrong topic (mitigated by echo-target-before-write). `/absorb` from repo root or domain folder enters bootstrap mode rather than failing, but with other commands `cwd` discipline still matters.
- **No interview-prep tooling.** Out of WHY.
- **No global skill state.** Each topic carries its own calibration in `_topic.md`. No cross-topic memory of skill — re-declare at each topic-onboarding.
- **No automatic skill detection.** Calibration reflects only what you explicitly mark.

Each can be added later. None is needed in V1.

---

## 11. Where the bare minimum might bite

### 11.1 `CLAUDE.md` and `AGENTS.md` are the load-bearing artifacts

Top-level system behavior — north star, directory layout, provenance tags, grounding gates summary, source priority, cross-cutting disciplines, command summaries, non-negotiables — lives in these two convention files. Each agent reads its own at session start and keeps it in conversation context throughout.

**Progressive disclosure (not full inline replication).** Anthropic's best practice: target <200 lines per CLAUDE.md; instruction-budget caps at ~150-200 instructions before compliance drops off. Putting full per-command specs inline would blow the budget. Instead:
- `CLAUDE.md` / `AGENTS.md` carry orientation, cross-cutting disciplines, and pointers to `specs/[command].md` for operational detail.
- Each command's slash body (`.claude/commands/[command].md`) instructs the agent to read `specs/[command].md` first when invoked. The spec then sits in conversation context for that session.
- This satisfies "spec stays in context" reliably — the relevant spec loads at command entry, not via mid-session pointer-following.

**Maintenance:** ~95% of content shared between `CLAUDE.md` and `AGENTS.md`; ~5% diverges on agent-specific platform notes (Codex sub-agent fallback, web research tool mechanics). Manual sync between the two when shared content changes.

**Right-sizing matters.** If too long, the agent selectively follows; if vague, behavior drifts. Current `CLAUDE.md` is ~145 lines — within Anthropic's target.

### 11.2 Residual grounding failure modes after §5

Plausible-but-wrong inferences shared with verifier blind spots; subtle misinterpretation where quotes are accurate but readings are wrong. Your reading is the ultimate gate.

### 11.3 Directory-as-state can route to the wrong topic

If you run a slash command from the wrong *topic* folder (one that exists but isn't the topic you meant), content lands somewhere unintended. Mitigation: every command echoes `"writing to topics/X/"` at start; user catches it.

For `/absorb` specifically: from repo root or domain folder, the command enters **bootstrap mode** (creates a new topic) rather than misrouting. For other commands (`/synth`, `/ideate`, `/apply`) from a non-topic `cwd`: halt and ask which topic.

### 11.4 Calibration drift within a topic

Because `_topic.md` calibration updates only via explicit signals, it can drift behind your actual state mid-topic if you forget to mark concepts as understood. Mitigation: session logs may surface a *"want to update calibration?"* hook when significant new ground was covered. Final discipline defined in Pass 2.

---

## 12. Pass 2 status

Pass 2 (operational specs) is largely complete. The system is V1 build-ready.

### 12.1 Drafted artifacts

- `specs/verifier.md` — Gate 2 verifier protocol (capture-time sub-agent, 5 provenance tags, 5-step check, 4 verdicts).
- `specs/absorb.md` — `/absorb` extraction + advisor-mode spec.
- `specs/research.md` — research capability spec (keyword + agent-introspective, `/apply` auto-fire, findings handling).
- `specs/synth.md` — `/synth` learning-partner spec (applied-builder depth, didactic+bite-sized teaching, three-level prereq handling, batched research-permission asks, silent+selective capture, session log).
- `specs/ideate.md` — `/ideate` topic-anchored unbounded ideation spec (three-section candidates, qualitative criteria evaluation, generative+grounded-skeptical voice, three behavioral traits).
- `specs/apply.md` — `/apply` spec-authoring spec (candidate-anchored, auto-fired live-doc verification, WHAT/HOW principle, 13-section spec output).
- `CLAUDE.md` — agent system spec, ~145 lines, progressive-disclosure pattern.
- `.claude/commands/{absorb,synth,ideate,apply}.md` — slash command bodies (~80 lines total). Auto-loaded by Claude Code; instruct agent to read relevant `specs/*` on invocation.
- `README.md` — user-facing manual (~375 lines).
- `.gitignore` — minimal OS/editor cruft exclusions.

Showcase criteria, session-close triggers, capture triggers, and per-command behavior are all defined in their respective specs.

### 12.2 Partial — defined per command, may refine in real use

These disciplines are covered in the relevant specs but may benefit from cross-cutting refinement once the system is in active use:

- **Session log shape unification** — `/synth`, `/ideate`, `/apply` each define a session log per spec; first real usage will surface whether cross-command consistency adjustments are needed.
- **State-aware suggestion patterns** — each command's spec hints at suggestion phrasings when topic state looks unusual; exact wording will emerge as patterns crystallize during use.
- **Domain-vs-topic detection** — `cwd` discipline defined per command (`specs/absorb.md` §2 sets the pattern; others follow it). Behavior is consistent; the operational details emerge with usage.
- **Calibration-update hook in session logs** — `§11.4` mitigation mentions a *"want to update calibration?"* prompt; not yet in any spec. Will land in session-log discipline alongside first real session.

### 12.3 Pending — for the user / first real usage

- **`AGENTS.md`** — user will write after using `CLAUDE.md` as a template; follows progressive-disclosure pattern + Codex-specific platform notes (sub-agent self-check fallback, web research tool mechanics).
- **`_topic.md` formal schema** — sketched in `specs/synth.md` §3.1 (the calibration section is the load-bearing element); full schema with frontmatter fields, archival markers, and topic-summary section likely emerges during first topic use.
- **Topic archival mechanism** — verbal trigger + agent behavior producing `synthesis/topic-summary.md` as consolidated-understanding artifact. Not in any spec; will be defined when first archival happens.
*(Note: build itself is outside the tool's scope. The tool's job ends at producing `spec.md`. What happens after — where code lives, how the build session references the spec, build mechanics — is not the tool's concern.)*

### 12.4 Optional add-ons — defer until V1 is in use

- Per-topic `watch.md` for within-topic update awareness
- MCP integrations (ArXiv, Scholar)
- Skills, hooks
- Portfolio coherence check (lightweight on-demand prompt)
- External showcase artifact support (README / demo / write-up generation)

---

**End of Pass 1 specification.**
