# Learning OS

A personal AI learning system that helps you **deepen understanding of AI developments** and **ship applied AI projects from that understanding**.

This file is the operator's manual — what the tool is, what it's made of, and how to use it end-to-end. For the agent-facing system spec see [`CLAUDE.md`](./CLAUDE.md). For full operational detail per command see [`specs/`](./specs/). For design history and rationale see [`product_spec.md`](./product_spec.md).

---

## 1. What this is (and isn't)

**Is:**
- A single-user, CLI-only tool that runs inside an AI coding agent (Claude Code; Codex works once you write `AGENTS.md`).
- A markdown-file-based knowledge system organized around **topics** — focused study areas you work for weeks or months.
- Designed for an **applied-AI builder** — depth is *"build / debug / explain / choose"* competence, not researcher-level mastery.

**Isn't:**
- A general note-taking app or knowledge graph.
- A passive chatbot. The agent is **active** — proposes structure, identifies gaps, pressure-tests ideas, surfaces conflicts.
- A workflow engine. No scheduler, no cron, no notifications. Does nothing when you're not in a session.

## 2. Purpose

The tool exists to produce two outputs:

1. **Real conceptual depth** in AI developments at applied-builder level (enough to build, debug, explain, and choose with the concept — not researcher mastery).
2. **A portfolio of shippable applied AI projects** that demonstrate cutting-edge capability.

Every design decision in the system traces back to one of those two outputs. If a behavior helps neither, it isn't load-bearing.

---

## 3. Mental model

Everything is organized around **topics** — focused study areas you work for weeks or months. A topic flows through four layers, top to bottom:

```
   ┌─────────────────────────────────────────────────────────┐
   │ 1. ABSORB    Ingest content into the topic's corpus     │
   │              (URLs, PDFs, images, text, transcripts)    │
   ├─────────────────────────────────────────────────────────┤
   │ 2. SYNTH     Agent teaches you the topic;               │
   │              you capture concept notes                  │
   ├─────────────────────────────────────────────────────────┤
   │ 3. IDEATE    Generate applied project candidates,       │
   │              anchored to the topic                      │
   ├─────────────────────────────────────────────────────────┤
   │ 4. APPLY     Pick a candidate, develop a buildable      │
   │              spec.md the build session executes from    │
   └─────────────────────────────────────────────────────────┘
   ↓ exit the tool — open a separate agent session in a build
     directory, point it at the spec, build.
```

You can also loop backwards at any time (mid-`/synth` you can `/absorb` a fresh paper; mid-`/apply` you can `/ideate` again). The system never auto-transitions between layers — you type the next slash command when you're ready.

**Cross-cutting throughout all layers:** research (fetch external info), verification (check the agent's claims), capture (commit what's worth keeping), provenance tags (every claim labeled with its source), and skill calibration (per-topic). All explained below.

---

## 4. Components

### 4.1 Topics and the folder layout

A **topic** is a folder under `topics/` containing a `_topic.md`. Topics can group into **domains** for organization (a folder that holds topics but has no `_topic.md` of its own — pure grouping, no synthesis). Two-level hierarchy only.

```
topics/
├── agentic-ai/                        # domain (no _topic.md)
│   ├── managed-agents/                # topic (has _topic.md)
│   └── tool-use/                      # topic
├── rag/                               # standalone topic (no domain)
└── archived/                          # finished topics
```

Inside a topic:

```
topics/[domain]/[topic-name]/
├── _topic.md                          # goal + intended output + skill calibration
├── _inbox/                            # filesystem-drop staging for /absorb
├── _sessions/                         # session logs (YYYY-MM-DD-HHMM.md)
├── absorbed/[date]-[slug]/            # one folder per ingested source
│   ├── summary.md                     # agent's structured summary
│   └── original.{ext} | source.md     # the original file or URL snapshot
├── synthesis/                         # populated by /synth + /ideate
│   ├── [concept].md                   # concept notes from /synth
│   └── applications.md                # candidates from /ideate
├── projects/[candidate-slug]/         # populated by /apply
│   └── spec.md                        # buildable spec
└── verification.md                    # capture-time verifier audit log
```

You don't have to `mkdir` any of this. Bootstrap mode (`/absorb` from repo root with content) creates the structure for you.

### 4.2 The five slash commands

Four learning-flow commands plus one utility:

| Command | One-line purpose |
|---|---|
| `/absorb` | Ingest content, analyze the corpus, or bootstrap a new topic |
| `/synth` | Agent teaches the topic at applied-builder depth |
| `/ideate` | Generate applied project candidates anchored to the topic |
| `/apply` | Convert a chosen candidate into a buildable spec |
| `/done` | Utility — signals session close, writes the session log |

Each command's full operational detail lives in `specs/[command].md`. Each command body in `.claude/commands/[command].md` instructs the agent to read the spec before responding.

### 4.3 Cross-cutting capabilities

These capabilities work across all commands. The user journey section (§6) explains exactly how to invoke each.

| Capability | What it does | How it fires |
|---|---|---|
| **Research** | Fetch external info to ground a claim | User keyword (*"research X"*), agent-asks-permission, or auto-fire inside `/apply` |
| **Verification** | Check whether the agent's claims hold up | Auto at every capture (`/synth`, `/ideate`, `/apply` writes); manual mid-dialogue with *"verify that"* |
| **Capture** | Commit what's worth keeping to a file | User verbal (*"capture this"*) or agent suggestion at crystallization moments; never auto-written |
| **Provenance tags** | Every claim labeled with its source | Always-on; you see them inline as the agent teaches |
| **Calibration** | Per-topic comfort levels on critical fundamentals | Set at first `/synth` session; updates only by express input (*"mark X as understood"*) |
| **Session logs** | Continuity across sessions | Written by `/done` (or verbal *"wrap up"*) — what was covered, captures, open threads |

### 4.4 Provenance tags — the five labels

Every claim the agent makes carries one of five tags. You see them inline as the agent teaches; the capture-time verifier uses them to decide what kind of check applies.

| Tag | Meaning |
|---|---|
| `[ABSORBED]` | From a source in this topic's `absorbed/` |
| `[RESEARCH]` | From the agent's external research this session |
| `[INFERENCE]` | Agent's reasoning over the above |
| `[MODEL-STABLE]` | Foundational model knowledge (textbook material; no citation required) |
| `[MODEL-UNCERTAIN]` | Model knowledge in hallucination-prone territory (recent, specific, attribution-prone) |

Model knowledge is a **primary teaching source**, not a fallback. The system's job is to make uncertainty legible, not suppress it.

---

## 5. Requirements + quick start

### 5.1 Requirements

- **A CLI coding agent.** Claude Code is the primary supported environment. Codex works once you write `AGENTS.md` (analog of `CLAUDE.md`).
- **Git.** For cross-machine sync via a private remote.
- **No other dependencies.** No Python, no Node, no databases — just markdown files.

### 5.2 Quick start

```bash
git clone <your-private-repo-url> learning-os
cd learning-os
claude
```

Then in chat:

```
[paste a URL, drop a PDF, or paste a note about what you want to learn]
/absorb
```

The agent enters **bootstrap mode** — asks 2–3 brief questions, creates the topic structure, writes `_topic.md`, and processes your first content. From there, see §6 for the full user journey.

---

## 6. The user journey

This section walks the operational mechanics top to bottom — how each command actually works in dialogue, what the agent will say and do, what you can interject with, and what ends up on disk.

### 6.1 Starting a topic (bootstrap mode)

Bootstrap is the lowest-friction way to start. You don't `mkdir` anything.

```
# Terminal (at repo root):
cd ~/learning-os
claude
```

```
# In chat, paste any of: URL / PDF / image / text / transcript, then:
You: I want to start learning about decision transformers and applying them
     to small RL agents. Here's the paper:
     https://arxiv.org/abs/2106.01345
     /absorb

Agent: Starting a new topic. Should I call it `decision-transformers`?
       Standalone, or under a domain like `agentic-ai` / `rl`?

You: Under agentic-ai.

Agent: Brief — what's your goal? Understanding, applied project, or both?

You: Both. Want to build a small RL agent demo.

Agent: Drafting _topic.md:
       # Decision Transformers
       ## Goal: Understand decision transformers + apply to build a small
       RL agent demo.
       ## Intended output: Both.
       ## Starting status: Fresh start. First absorbed: arxiv.org/abs/2106.01345.

       Creating topics/agentic-ai/decision-transformers/ with subfolders.
       Writing _topic.md.
       Now absorbing the paper:
       → absorbed/2026-05-17-decision-transformer/{summary.md, source.md}

       Done. Want me to point at gaps this raises? Or run /synth to start
       learning?
```

After bootstrap, `cd` into the new topic folder for everything else.

**Manual alternative** (if you prefer):
```bash
mkdir -p topics/[domain]/[topic-name]/{_inbox,_sessions,absorbed,synthesis,projects}
cd topics/[domain]/[topic-name]
# Write _topic.md by hand
```

### 6.2 Adding content (extraction modes)

Once inside a topic, two patterns add content. Both end at `absorbed/[date]-[slug]/` with a structured summary and the preserved original.

**Pattern A — chat-drop (best for one item at a time):**

| What you have | How to drop it |
|---|---|
| URL | Paste the URL as text |
| PDF | Attach in chat |
| Image | Paste or attach |
| Plain text / notes | Paste the text |
| Video transcript | Paste the transcript |

Then `/absorb`. Optional context note in quotes: `/absorb "from the Karpathy NeurIPS talk"`.

The agent detects the type, writes `absorbed/[date]-[slug]/summary.md`, and saves the original alongside (`original.pdf` / `original.png` / `original.txt` / `source.md` for URLs).

**Pattern B — `_inbox/`-drop (best for batches):**

Save files into the topic's `_inbox/` folder however you like (drag-and-drop, `cp`, browser save-as):

```
topics/agentic-ai/decision-transformers/_inbox/
├── ppo-paper.pdf
├── stable-baselines-docs.pdf
└── karpathy-rl-notes.txt
```

Then in chat (from inside the topic folder):

```
/absorb
```

No chat content needed — agent scans `_inbox/`, processes each file (moves into `absorbed/[slug]/original.{ext}`, writes `summary.md`), clears `_inbox/` on success. Failed items stay in `_inbox/` for retry.

**Mixed (A + B in one invocation):** drop something in chat AND have files in `_inbox/`. Agent processes both in one pass.

**Common questions:**

- **Same content twice?** Agent detects duplicates and asks before overwriting (skip / overwrite / save as `-v2`).
- **Edit `summary.md` after the agent writes it?** Yes — plain markdown. The verifier checks claims when content is later pulled into synthesis, so edits should stay faithful to the source.
- **Agent's summary missed something?** Original is still in the same folder. Edit `summary.md` directly, or re-run `/absorb` against the original with explicit focus (*"absorb again, focus on section X"*).

### 6.3 Surveying the corpus (advisor mode)

Once you have items absorbed, run `/absorb` from inside the topic **with no content** to enter advisor mode. Voice: senior research advisor reviewing your corpus.

```
/absorb what should I look for next?
/absorb any gaps in my current corpus?
/absorb summarize my corpus health
/absorb any conflicts between sources?
```

Four capabilities:

1. **Gap surfacing** — identifies critical conceptual gaps, points to *source types* and entity-level venues (e.g., *"Anthropic's blog"*, *"NeurIPS-era papers"*). **Never specific titles/authors/years** (hallucination risk).
2. **Cross-source conflict detection** — flags tensions between absorbed items; doesn't pick a side.
3. **Source-set health summary** — number of items, source-type distribution, perspectives represented, notable absences, topic drift relative to `_topic.md`.
4. **Open-questions surfacing** — pulls accumulated open questions from across `absorbed/[slug]/summary.md` files. Natural entry points to `/synth`.

**No file writes in advisor mode.** Findings stay in chat. If a finding is worth keeping, you explicitly `/absorb` on it (with a URL or pasted content).

### 6.4 Learning the topic (`/synth`)

`/synth` is the learning partner. Didactic, conversational, bite-sized teaching at **applied-builder depth** — enough to build, debug, explain, and choose with the concept. Not researcher mastery.

```
cd topics/agentic-ai/decision-transformers
/synth
```

Optional focus hint: `/synth focus on the offline RL framing` or `/synth pick up where we left off`.

**Session start** — the agent reads `_topic.md`, the most recent 1–2 session logs, existing `synthesis/[concept].md` files, and absorbed items. Then one of two paths:

**Path A — first `/synth` session of the topic (topic-onboarding):**

The agent identifies 3–7 critical fundamentals for the topic at applied-builder level and asks where you stand on each:

> *"For decision transformers, the foundations I'd typically teach with are:*
> *(A) Sequence modeling and transformers as a sequence model.*
> *(B) The offline-RL framing — trajectories as sequences with returns-to-go.*
> *(C) Position encoding and how it carries timestep info.*
> *(D) Causal masking + autoregressive prediction.*
> *Where are you on each — comfortable, shaky, or new?"*

You answer. The agent writes the calibration to `_topic.md` (permanent, topic-local). Then it proposes a starting point given your answers, and you confirm or redirect.

**Path B — subsequent session (resume flow):**

The agent opens with continuity from the last session log: *"Last session we covered X. Open threads were Y and Z. Next-time hook says we were going to explore Y. Pick up there, or shift?"* You choose.

**During teaching — verbal triggers (interject anytime):**

| What you say | What happens |
|---|---|
| *"capture this"* / *"save this"* / *"note this"* | The agent captures the concept just discussed into `synthesis/[concept].md` |
| *"back up"* / *"earlier"* / *"I'm lost"* | Backtrack to prerequisite |
| *"move on"* / *"next"* / *"got it"* | Proceed to next chunk |
| *"go deeper"* / *"show me the math"* | Session-scoped depth bump (resets next session) |
| *"this is too deep"* / *"pull back"* | Session-scoped depth dial-back |
| *"keep it applied"* | Reset to applied-builder default |
| *"verify that"* / *"is that right?"* | Fires mid-dialogue verifier on the preceding teaching turn (see §6.10) |
| *"research X"* | User-triggered research fire (see §6.9) |
| *"mark X as understood"* / *"shaky on Y"* | Updates the calibration in `_topic.md` (see §6.11) |
| *"wrap up"* / `/done` | Closes the session and writes the log (see §6.8) |

**How `synthesis/` gets populated.** Capture is the only way concept notes land in `synthesis/`. Two trigger paths, both gated on you:

- **You trigger it.** Mid-dialogue, you say *"capture this"*. The agent writes the concept to `synthesis/[concept].md`.
- **The agent suggests it.** At a real crystallization moment (high bar — not after every chunk), the agent says *"this feels like it crystallized — want me to capture it?"* You say yes/no.

Either way, **the agent never writes silently.** Each capture also fires the Gate 2 verifier — if a claim FAILs, the write is blocked until the writer fixes it.

A captured note has frontmatter (concept name, topic, date, session, verifier verdict) and sections: Core idea → Mechanism → Why it matters → Open questions → Verifier annotations. Notes are one concept per file, with closely-coupled concepts sometimes sharing a note (the agent asks at capture: *"capture attention and multi-head attention together, or separately?"*).

**Research-permission asks (batched).** When the agent teaches `[MODEL-UNCERTAIN]` claims, it batches them and asks at a natural pause: *"Among what I just covered, three claims are `[MODEL-UNCERTAIN]` — research all, none, or pick?"* You decide per claim. (More in §6.9.)

### 6.5 Generating project ideas (`/ideate`)

`/ideate` is the ideation partner. Voice: senior techno-product applied AI builder — generative, grounded-skeptical, critically honest, non-adversarial. Both **concept demos** (notebooks, libraries, repos demonstrating capability) and **applied products** (with users, with product fit) are first-class outputs.

```
cd topics/agentic-ai/decision-transformers
/ideate
```

Optional focus hint: `/ideate focus on user-facing demos` or `/ideate brainstorm broadly`.

**Session start** — the agent reads `_topic.md` (skipping the calibration section; skill state is out of scope for ideation), `synthesis/[concept].md` files, existing `applications.md` if any, absorbed items, and recent `/ideate` logs.

**First `/ideate` session — opens with breadth:**

> *"Given your topic on decision transformers and what you've covered in synthesis, here are directions:*
> *(1) An offline-trained game-playing agent that learns from a curated replay buffer — demonstrates trajectory-as-sequence end-to-end.*
> *(2) A 'returns-to-go' study notebook comparing supervised return-conditioning vs. behavioral cloning on a toy domain.*
> *(3) An RL-driven A/B test orchestrator that learns which experiments to prioritize — applies the sequence-modeling lens to meta-decision.*
> *(4) A live offline-RL fine-tuning harness for existing LLM agents trained on collected logs.*
>
> *Want to dig into any of these, or hear more directions?"*

You pick one to dig into, or ask for more, or kill some.

**Two load-bearing concepts unique to `/ideate`:**

1. **The load-bearing test (routing).** For every developed candidate, the agent asks: *if you removed the topic, does this idea still make sense?* Failure → routes to **adjacent bucket** (not pre-discarded; surfaced for you to decide). Pass → **main bucket** (topic-anchored, primary).
2. **Unbounded by user state.** The agent does not read your calibration, does not perform gap analysis on you, does not filter for *"comfortable to build."* The strength of the *idea* is the bar. The requirements list states what the idea needs; **you** evaluate gaps yourself.

**Candidate development flow** — when you pick a direction, the agent develops it through dialogue: sketches the idea fully → applies the load-bearing test (routes to main or adjacent) → evaluates against five criteria qualitatively → surfaces strengths / weaknesses / requirements → invites refinement (*"any of this you want to narrow, shift, or stretch?"*) → settles or you signal capture/kill/develop further.

**The five criteria, evaluated as written paragraphs (no scores or labels):**

- **Topic-anchored** — does the load-bearing test pass?
- **Realistic** — capability-anchored: is the technology there?
- **Externally useful** — does the built thing demonstrate capability to peers / hiring managers / community?
- **Recent AI capability** — is it at the relevant frontier, or 5-year-old patterns?
- **Bounded scope** — is there a clear "done"?

**Capture format.** Captured candidates land in `synthesis/applications.md`, split into two clearly-labeled buckets (`## Topic-anchored candidates` and `## Adjacent — non-topic-anchored`). Each candidate carries: What it is → How the topic plays in → Strengths → Weaknesses → Requirements (pure factual list, no commentary on you) → Qualitative evaluation against each criterion → Verifier annotations.

**Verbal triggers in `/ideate`:**

| What you say | What happens |
|---|---|
| *"capture this"* / *"save this"* / *"add to applications"* | Captures current candidate to `applications.md` (with bucket routing) |
| *"more options"* / *"another direction"* | Generate additional directions |
| *"develop this"* / *"dig in"* | Deep-dive on current candidate |
| *"narrow it"* / *"stretch it"* | Refine scope (narrower or more ambitious) |
| *"kill this"* / *"drop it"* | Discard current candidate |
| *"why isn't this topic-anchored"* / *"why adjacent"* | Agent surfaces load-bearing test reasoning |
| *"actually it is anchored, here's why"* | User pushback on routing — agent re-evaluates honestly |
| *"check the criteria"* | Run qualitative evaluation explicitly |
| *"research feasibility"* / *"verify this works"* | User-triggered research on a technical claim |
| *"wrap up"* / `/done` | Session close |

### 6.6 Building a spec (`/apply`)

`/apply` is the bridge from idea to build. It converts a chosen candidate into a verified, executable `spec.md` at `projects/[candidate-slug]/spec.md`. Voice: practical builder — concrete, technical, verifying, scope-disciplined.

```
cd topics/agentic-ai/decision-transformers
/apply offline-trained-game-agent       # if you know the candidate slug
/apply                                   # agent lists candidates, asks
```

**Two principles to internalize:**

1. **Detailed on WHAT to build; free hand on HOW.** The spec locks stack (specific libraries + versions), architecture (components, data flow, integration points), data shapes at boundaries, interface contracts, test plan, build phases with done states, acceptance criteria, out-of-scope. The spec does **not** dictate internal file organization, internal function signatures, framework-specific implementation patterns, or error message wording — those belong to the build session.
2. **The user is not an engineer.** Agent does the technical heavy lifting — proposes the stack, architecture, data shapes, contracts, test plan, phases. You review reasoning and retain scope/product authority. Trust comes from three sources: (a) every technical claim is auto-fire verified, (b) every choice carries visible reasoning, (c) every uncertainty is named honestly.

**Auto-fired research — the one exception to permission-gated research.** Whenever a specific technical claim is about to commit to the spec (library name, library version, API endpoint, API signature, model identifier, framework version), the agent fires external research **without permission ask.** It announces the fire (*"verifying FAISS v1.8.0 against live docs"*), findings surface inline, and the spec commits only what's verified. Why no permission ask: the cost of skipping is build failures.

**Development flow** — the agent walks (you approve / refine each step):

1. Confirm candidate scope (re-reads `applications.md` entry).
2. Propose technical stack (specific libraries + versions, models, APIs) — auto-fire verification per claim.
3. Propose architecture / system design.
4. Propose data shapes at boundaries.
5. Propose interface contracts.
6. Propose build phases (MVP → enhancements), each with a concrete done state.
7. Propose test plan (scenarios, "verified working" definition).
8. Propose acceptance criteria.
9. Propose out-of-scope (sprawl prevention).
10. Surface risks and open questions with honest uncertainty.
11. Re-import requirements from the candidate.
12. Settle and capture to `projects/[candidate-slug]/spec.md`.

**Verbal triggers in `/apply`:**

| What you say | What happens |
|---|---|
| *"capture this"* / *"save spec"* / *"finalize"* | Writes `projects/[slug]/spec.md` |
| *"verify this"* / *"check that library"* | User-triggered research beyond auto-fire scope |
| *"narrow scope"* / *"split into phases"* | Move items to out-of-scope or push to phase 2 |
| *"this is out of scope"* | Add current item to out-of-scope list |
| *"add a phase"* / *"include X"* | Expand build phases |
| *"wrap up"* / `/done` | Session close |

**Handoff to build (outside the Learning OS).** The spec is the artifact. To build:

```
# Open a separate agent session in a separate build directory:
cd ~/builds/[project-name]
claude
# Point it at the spec, build.
```

How the build session references the spec, where built code lives, whether stale claims get re-verified — all per-build user choice. The spec is designed to be self-contained.

### 6.7 The four-layer flow — moving forward and backward

**Forward through the lifecycle.** Just type the next slash command. The agent reads topic state and orients itself.

| When you're done with… | Type | What happens |
|---|---|---|
| Absorbing material | `/synth` | First session: topic-onboarding (asks about comfort on critical fundamentals). Subsequent: resumes from session log's "next-time" hooks. |
| Learning enough to ideate | `/ideate` | Reads synthesis notes, opens with sketched candidate directions, develops whichever you pick. |
| Picking a candidate | `/apply [slug]` | Locates the candidate in `applications.md`, walks through stack / architecture / data shapes / contracts / phases / acceptance / out-of-scope. |
| Spec is captured | (exit Learning OS) | Open a new agent session in a separate build directory; point it at the spec; build. |

**Looping back / sideways.** Same pattern — type the earlier slash command anytime. Slash commands are stateless; you can run any of them from inside a topic folder anytime.

- Mid-`/ideate`, you need more learning → `/synth`
- Mid-`/synth`, want to grab a fresh source → drop content + `/absorb`
- Mid-`/apply`, want to revisit candidate framing → `/ideate`
- Mid-`/synth`, want to survey the corpus → `/absorb` (with no content → advisor mode)
- Mid-anything, want to check the agent's claim → say *"verify that"*

**Soft state-aware suggestions.** If you run a command in an unusual order (e.g., `/ideate` when `synthesis/` is empty), the agent will flag it (*"no captured synthesis yet; ideation will lean on absorbed/ and `[MODEL-STABLE]` knowledge — proceed?"*) and let you proceed or pivot. The system never auto-transitions or auto-blocks.

**Best practice: close before transitioning.** Run `/done` before switching commands. It writes the session log so the next session has continuity.

### 6.8 Closing sessions (`/done`)

`/done` is the utility command that closes the active focused session. It writes the session log for whichever learning-flow command was active. Verbal equivalents (*"wrap up"*, *"we're done"*, *"end session"*, *"let's stop here"*) trigger the same behavior.

**What `/done` writes by command:**

| Active command | What gets written |
|---|---|
| `/synth` | `_sessions/[YYYY-MM-DD-HHMM].md` — what we covered, captures committed, captures pending, calibration updates, open questions, next-time hooks |
| `/ideate` | `_sessions/[YYYY-MM-DD-HHMM].md` — directions explored, captures (main + adjacent), candidates developed but not captured, candidates killed, pending threads, next-time hooks |
| `/apply` | `_sessions/[YYYY-MM-DD-HHMM].md` — what we developed, spec status (draft/approved/file path), technical verifications fired, open questions, next-time hooks |
| `/absorb` extraction | No log (extraction is single-transaction) |
| `/absorb` advisor | Brief log if substantive exploration happened |

**Draft-and-confirm flow.** Before writing, the agent reads the draft log back to you. You can edit verbally (*"drop the X capture from the list"*, *"add Y to next-time hooks"*) before it commits.

**Example:**

```
[in /synth, just finished a concept]
You: /done

Agent: Wrapping up. Drafting session log:
       ## What we covered
       Walked through attention's role in transformers — Q·K dot product, softmax,
       weighted V. Touched positional encoding briefly.

       ## Captures committed
       - synthesis/attention-mechanism.md — core attention computation

       ## Open threads
       - Positional encoding variants (rope, alibi) — flagged for next session

       ## Next-time hooks
       - "next time, explore rope vs alibi for long-context attention"

       Look right? Or edit?

You: Drop the positional-encoding open thread; we resolved that.

Agent: Updated. Writing to _sessions/2026-05-20-1422.md. Session closed.
```

**What happens if you skip `/done`.** Not catastrophic — just costs the breadcrumb. The next session opens cold without "what we covered" or "next-time hooks." Captures and synthesis notes are still on disk (they're written at capture time, not at session close).

**`/done` does not invoke the next command.** You explicitly invoke the next one when you're ready.

### 6.9 Triggering research

Research is a **capability, not a slash command**. It fires from inside any active session. Three invocation paths:

**Path 1 — user keyword (you trigger it).** In any chat session, use a natural-language phrase:

- *"research X"*
- *"can you research this?"*
- *"look this up"*
- *"verify that externally"*

If scope is ambiguous, the agent confirms (*"you want me to research the FAISS v1.8.0 indexing limit, right?"*) then fires. If unambiguous (e.g., immediately after a specific claim), the agent fires without asking.

**Path 2 — agent asks permission.** The agent self-detects when it needs external grounding and asks:

> *"This claim about [X] is in uncertain territory — [recent / attribution-prone / specific number]. Should I verify externally?"*

You say yes / no / "yes but quickly" / "yes and absorb the finding." No autonomous firing.

**Path 3 — auto-fire inside `/apply`.** Technical claims (library names + versions, API signatures, model IDs, endpoint URLs, framework versions) trigger immediate research without permission ask. This is the **one locked exception** to permission-gated research. The agent announces (*"verifying [API X] against live docs"*) so you know it's happening.

**Session-level pre-authorization (optional).** You can grant broader autonomy for the current session:

- *"go ahead and research whenever you need to without asking"* — full pre-auth for the session.
- *"for all [MODEL-UNCERTAIN] claims this session, just verify"* — scoped by tag.
- *"verify advisor suggestions without asking"* — scoped by trigger type.

Pre-auth is **session-scoped** (lapses when the session ends; logged in the session log, not persisted). Revoke anytime with *"stop researching without asking"*.

**Findings stay in chat by default.** The agent synthesizes findings inline with `[RESEARCH]` tags + URLs + verbatim quotes. **The agent never auto-writes research findings to `absorbed/`.** If a finding is worth keeping, you explicitly `/absorb` on it (paste the URL or content, then `/absorb`). The session log captures key findings even if not absorbed.

### 6.10 Triggering verification

Verification is the **Gate 2 verifier** — a sub-agent with an adversarial system prompt that checks captured claims against the five tags (`[ABSORBED]`, `[RESEARCH]`, `[INFERENCE]`, `[MODEL-STABLE]`, `[MODEL-UNCERTAIN]`). Discipline in one line: **surface uncertainty, don't surface garbage.**

**Capture-time verification (always automatic).** The verifier fires at every artifact write:

| Command | Verifier fires on |
|---|---|
| `/absorb` | Not fired (extraction is single-source, below threshold) |
| `/synth` | Each write to `synthesis/[concept].md` |
| `/ideate` | Each candidate appended to `synthesis/applications.md` |
| `/apply` | The `projects/[name]/spec.md` write |

The verifier emits one of four verdicts per claim:

- **PASS** — claim is written as-is, with provenance tag visible.
- **FLAG: SURFACED-WITH-DISCLAIMER** — substantive `[MODEL-UNCERTAIN]` claim or claim needing follow-up; written with disclaimer attached.
- **FLAG: TIGHTEN-OR-DROP** — vague hand-wave (*"often"*, *"typically"*, *"it's known that"* without grounding); the writer tightens or removes.
- **FAIL: SUPPRESS** — fabricated number / citation / attribution, or untagged claim; removed from the note. Logged in `verification.md` for transparency.

If any FAIL remains, the write is blocked until the writer fixes it. The audit log appends to `topics/[name]/verification.md` — claims checked, verdicts, what was suppressed and why.

**Mid-dialogue verification (you trigger it).** Capture-time verification doesn't catch teaching errors that happen *before* any capture. Use a verbal trigger to fire a focused verification pass on the agent's immediately preceding teaching turn(s):

- *"verify that"*
- *"check what you just said"*
- *"is that right?"*

Output goes to chat (no `verification.md` write, since nothing was captured). Use it whenever something feels off.

**What the verifier checks** (per claim, by tag):

| Tag | What "correct" means |
|---|---|
| `[ABSORBED]` | Verbatim source match + no drift |
| `[RESEARCH]` | Re-fetched source still says it; URL still resolves |
| `[INFERENCE]` | Logic traces from cited evidence without unsupported jumps |
| `[MODEL-STABLE]` | Consistent with itself and widely-known material; no common misconception. **No external citation required.** |
| `[MODEL-UNCERTAIN]` | The hedge is intact OR external corroboration is attached. The uncertainty is faithfully reported. |

**Inline disclaimer format example:**

```markdown
**Claim:** Transformers handle long contexts better than RNNs largely because
attention has no distance term.
[MODEL-UNCERTAIN] · VERIFIER: SURFACED — needs verification
*The intuition is supported by the structure of attention, but the specific
comparison to RNNs may have nuances (LSTM gating, vanishing gradient specifics).
Recommend external check before treating as established.*
```

### 6.11 Updating skill calibration

Calibration is **topic-local** (lives in `_topic.md`), **applied-builder framed** (comfort means build/debug/explain/choose, not exam-ready), and **only updates by express user input.** The agent never auto-infers calibration changes from dialogue.

**Set initial calibration** at the first `/synth` session of a topic. The agent surfaces 3–7 critical fundamentals; you declare comfort on each (comfortable / shaky / new). The agent writes the calibration to `_topic.md`.

**Update mid-session** with verbal triggers:

| What you say | Effect |
|---|---|
| *"mark X as understood"* / *"I get X now"* | Upgrades concept X to comfortable |
| *"mark X as shaky"* / *"I thought I knew X but I don't"* | Downgrades to shaky |
| *"add Y to my calibration"* | Adds a new concept the agent didn't ask about |

The agent acknowledges (*"updated — calibration for X is now [level]"*) and writes the change to `_topic.md`.

**No global skill state.** Calibration is per-topic, not user-level. A new topic re-asks at its own topic-onboarding flow.

### 6.12 Archiving a topic

When a topic is finished, move its folder to `topics/archived/`:

```bash
mv topics/agentic-ai/decision-transformers topics/archived/decision-transformers
```

Archived topics still respond to commands but the agent **warns and requires explicit confirmation** before proceeding. Useful for going back to reference a captured concept or revisit a candidate.

---

## 7. Cross-machine sync

```bash
# On machine A, after a session:
git add -A
git commit -m "Session on decision-transformers: covered offline-RL framing"
git push

# On machine B:
git pull
# Everything's there — your data, the tool, all updates.
```

No sync script. No submodules. One repo, one private remote.

---

## 8. Troubleshooting + FAQ

**Agent doesn't seem to know the system.** Open Claude Code from the **repo root** (so `CLAUDE.md` is in `cwd`). The agent reads `CLAUDE.md` at session start.

**Slash command isn't recognized.** Make sure `.claude/commands/[command].md` exists. Command name comes from filename.

**Agent ignored a discipline you expected.** Re-read the relevant `specs/[command].md` — the spec is the source of truth. If the discipline isn't there, it doesn't exist in the system yet.

**Verifier blocked a capture (FAIL).** The agent will say what failed. Fix or remove the problematic claim, re-run capture. The audit lands in `topics/[name]/verification.md`.

**`synthesis/` is empty even after working on the topic.** Capture is the only way concept notes land there. Either trigger captures yourself (*"capture this"* during `/synth`) or accept the agent's crystallization suggestions. Captures are never auto-written.

**`applications.md` doesn't exist yet.** It's created on first capture during `/ideate`. Before that, the file isn't on disk — no captures, no file.

**Same content absorbed twice — what happens?** Agent detects duplicates by URL / title / near-content match and asks before overwriting (skip / overwrite / save as `-v2`).

**Slash command from outside a topic.** `/synth`, `/ideate`, `/apply` all halt and ask which topic to use. `/absorb` from repo root with content enters bootstrap mode (creates a new topic).

**Confused about tool vs. data files.** **Tool:** `CLAUDE.md`, `AGENTS.md`, `.claude/`, `specs/`, `product_spec.md`, `README.md`. Edit when refining the system. **Data:** everything under `topics/` and `inbox/`. Generated as you use it.

**Updating a spec mid-topic.** Edit `specs/[command].md`. Commit + push. Next session in that command uses the updated spec.

---

## 9. Where to go next

- **Want to refine the system?** Edit the relevant `specs/[command].md`. Cross-cutting changes go in `CLAUDE.md`. Be cautious about adding complexity — the system is intentionally minimal.
- **Want design rationale?** [`product_spec.md`](./product_spec.md) tracks every locked decision, every cut, every residual risk.
- **Want the agent-side spec?** [`CLAUDE.md`](./CLAUDE.md) is what Claude Code reads at session start — orientation + cross-cutting disciplines + pointers to operational specs.
- **Want the per-command operational detail?** Each file in [`specs/`](./specs/) carries the full discipline for one command (or capability — `research.md`, `verifier.md`).

---

**End of README.**
