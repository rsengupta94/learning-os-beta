# /absorb — Pass 2 Specification

**Relates to:** §3.1 of `product_spec.md` (high-level idea); this file carries operational detail.

---

## 1. Purpose & framing

`/absorb` is **extraction + active curation partner**. Three modes share one command, dispatched by `cwd` + presence of content:

- **Bootstrap mode** — invoked from repo root or a domain folder with content (or stated intent to start a topic). Agent walks the user through topic setup (2–3 minimal questions), creates the topic folder structure, writes a minimal `_topic.md`, then proceeds with extraction on the provided content.
- **Extraction mode** — invoked from inside a topic folder with content (URL, PDF, image, plain text, video transcript, screenshot). Ingests the content faithfully into `topics/[cwd]/absorbed/[slug]/`, preserving both a markdown **summary** and the **original** file/snapshot.
- **Advisor mode** — invoked from inside a topic folder without content. Analyzes what's been absorbed against the topic's stated goal and points toward critical gaps, conflicts, and questions. Conversational. Does not write files.

The distinction with neighboring commands:

| Command | Question it answers |
|---|---|
| `/absorb` bootstrap mode | *"How do I start a topic on this material?"* — frames intent, creates structure. |
| `/absorb` extraction mode | *"How do I capture this material faithfully?"* — preserves source + creates summary. |
| `/absorb` advisor mode | *"What should be in this corpus?"* — analyzes shape, surfaces gaps. |
| `/synth` | *"What does this corpus teach me?"* — about content meaning. |
| `/ideate` | *"What applications could come from this understanding?"* |
| `/apply` | *"How do I build this specific application?"* |

`/absorb` is an **active layer** — it leads with content, asks framing questions when needed, analyzes the corpus, and points the user toward useful next moves. Not a passive extractor.

---

## 2. Invocation

`/absorb` dispatches by `cwd` + content + content-location:

| `cwd` state | Content source | Mode |
|---|---|---|
| Topic folder (`_topic.md` present) | Content dropped in chat | **Extraction (chat)** |
| Topic folder | Files in `_inbox/` | **Extraction (filesystem)** |
| Topic folder | None of the above | **Advisor** |
| Repo root or domain folder | Content dropped in chat | **Bootstrap → extraction** |
| Repo root or domain folder | None | Halt, ask: *"Do you want to start a new topic?"* |
| Archived topic folder | Any | Warn, require explicit confirmation before proceeding |

### 2.1 Bootstrap mode (new topic)

Invoked when `cwd` is repo root / domain folder + content provided (or user stating intent to start).

Agent asks 2–3 brief questions:
1. *"What should I call this topic?"* (suggests a name from the content if obvious)
2. *"Standalone, or under a domain?"* (if `cwd` is repo root)
3. *"Brief — what's the goal? Understanding, applied project, or both?"*

Then:
- Creates `topics/[domain]/[topic-name]/` (or `topics/[topic-name]/` standalone)
- Creates subfolders: `_inbox/`, `_sessions/`, `absorbed/`, `synthesis/`, `projects/`
- **Drafts** a minimal `_topic.md` from stated intent, **echoes the draft content to the user** before writing, and proceeds to write after a brief moment (user can interject *"wait, change X"* to refine before write)
- Echoes the topic structure created: *"Created `topics/[path]/` with `_topic.md` (drafted above) + subfolders."*
- Proceeds with extraction on the provided content (Pattern A flow)

### 2.2 Extraction mode (chat-based — Pattern A)

User drops content directly in chat:

```
[paste URL]
/absorb

[drop PDF attachment]
/absorb

[paste screenshot or text]
/absorb "context: from a Karpathy talk"
```

Agent extracts the content + saves the original to `absorbed/[slug]/original.{ext}` (or `source.md` for URLs).

### 2.3 Extraction mode (filesystem-based — Pattern B)

User has saved files into the topic's `_inbox/`:

```
topics/agentic-rl/_inbox/
├── decision-transformer.pdf
├── meetup-notes.txt
└── karpathy-talk.pdf

# Then in chat:
/absorb
```

(No content needed in chat — agent scans `_inbox/`.)

Agent processes each file:
- Generates slug + date
- Moves the file from `_inbox/` to `absorbed/[date]-[slug]/original.{ext}`
- Writes `summary.md` in the same folder
- Repeats for each `_inbox/` item

`_inbox/` is **cleared** as files are successfully processed. Files stay in `_inbox/` if absorb fails (retry-able).

### 2.4 Advisor mode

No content provided + already inside a topic:

```
/absorb
/absorb what should I look for next?
/absorb any gaps in my current corpus?
/absorb summarize my corpus health
```

Agent reads `_topic.md` + `absorbed/`, responds analytically. No file writes.

After extraction mode completes a write, the agent may offer a soft follow-up: *"want me to point at gaps this raises?"* — skippable.

---

## 3. Extraction mode — operational detail

### 3.1 Inputs

Five explicit types + unknown fallback:

#### URL

- Fetch the page content; resolve redirects.
- Extract main content; strip nav, ads, footers, sidebars; preserve heading hierarchy.
- Capture: URL, page title, author (if findable), publication date (if findable), accessed-on date.
- Paywall / login-wall: capture accessible portion; flag the barrier; do not fabricate.
- **Original storage:** write `source.md` containing URL + access date + the fetched content as markdown (protects against link rot).

#### PDF

- Read all pages; preserve structure (sections, subsections, figures with brief description, tables by location).
- Capture: title, authors, publication venue / date (if findable).
- For >50 pages: section-by-section summarization. Preserve specifics (numbers, formulas, terminology).
- **Original storage:** the PDF file is preserved as `original.pdf`.

#### Image

- Vision-describe visual content + extract embedded text (OCR-equivalent).
- Treat visual and textual content as separate threads in the summary.
- For screenshots of webpages / papers / documents: ask for source context before extracting.
- **Original storage:** the image file is preserved as `original.{png|jpg|...}`.

#### Plain text

- Take as-is — user has already curated.
- Ask for context if unclear what the text is.
- Preserve exact text in a quoted block within the summary.
- **Original storage:** verbatim text written to `original.txt`.

#### Video transcript / talk summary

- Treat as plain text with provenance hint (note it's a transcript).
- Capture speaker / source if findable, recording date if known.
- Mark passages as *"speaker said..."* — distinguishes from slides/handout if present.
- **Original storage:** transcript verbatim to `original.txt`.

#### Unknown / ambiguous

- Make a low-confidence extraction attempt.
- Flag `source_type: unknown` in frontmatter.
- **Ask the user before writing.**
- **Original storage:** preserve whatever was provided as-is.

### 3.2 Capabilities

- Detect input type from content + context note.
- Apply type-appropriate extraction discipline.
- **Preserve the original** in the absorbed folder (for re-reference, re-extraction, provenance).
- Generate a faithful summary preserving citable specifics (numbers, formulas, names, terminology).
- Tag every claim with provenance per `specs/verifier.md` §3.
- Cite-and-quote source text for `[ABSORBED]` claims.
- Write structured markdown (frontmatter + body) as `summary.md`.
- Detect potential duplicates and ask before overwriting.
- Scan `_inbox/` and process filesystem-dropped files.
- Process content in any language; note original language in frontmatter.

### 3.3 Behavior — step-by-step

1. **Detect `cwd`.** Determine mode per §2.
   - If bootstrap mode triggered: run §2.1 flow (questions, create structure), then continue to step 2.
   - If extraction mode: proceed.
   - If advisor mode: skip to §4.
2. **Echo write target.** *"Writing to `topics/[name]/absorbed/[slug]/`."*
3. **Gather content.** From chat (Pattern A) or `_inbox/` (Pattern B). If `_inbox/` has items + chat also has content, process both in order.
4. **For each item:**
   - **Detect input type.** Ask if ambiguous.
   - **Generate slug** from source title (or topic + descriptive fragment) + absorbed-on date.
   - **Check for duplicates.** Scan `absorbed/` for matching URL / title / near-duplicate. Ask before overwriting.
   - **Create folder** `absorbed/[date]-[slug]/`.
   - **Save original** to `absorbed/[date]-[slug]/original.{ext}` (or `source.md` for URLs). For Pattern B: **move** from `_inbox/`. For Pattern A: copy/save from chat content.
   - **Extract per type** (§3.1 discipline).
   - **Generate summary** following Gate 1 disciplines (cite-and-quote, provenance tags, distinguish summary from assertion).
   - **Generate frontmatter** (§3.4).
   - **Write** `summary.md` in the same folder.
5. **Echo result** for each item. One line per item: *"Wrote `absorbed/[slug]/` (summary + original)."*
6. **Offer advisor follow-up** (soft, once at end): *"want me to point at gaps these raise?"*

### 3.4 Output

File layout per absorbed item:

```
absorbed/[YYYY-MM-DD]-[slug]/
├── summary.md             # agent's extraction (always)
├── original.{ext}         # for PDFs, images, text, transcripts
└── source.md              # for URLs only (URL + access date + fetched content as markdown)
```

`YYYY-MM-DD` is the absorbed-on date, not source-publication date.

`summary.md` format:

```markdown
---
source_type: url | pdf | image | text | transcript | unknown
source_url: https://...                     # if applicable
source_title:
source_author:                              # if findable
source_date_published: YYYY-MM-DD           # if findable
source_language: en                         # default en
absorbed_date: YYYY-MM-DD
slug:
topic: [topic-name from cwd]
context_note:                               # optional, from user
original_file: original.pdf                 # filename of preserved original (or source.md for URL)
gate1_passed: true
---

# [Source title]

**Source:** [URL or filename or "user-pasted text"]
**Type:** [url / pdf / image / text / transcript]
**Accessed:** [YYYY-MM-DD]
**Original preserved:** [original.{ext} or source.md]

## Summary

[Brief context — what this is, who wrote it, what it's about. 1–2 sentences. [INFERENCE] if the agent inferred the framing.]

## Key claims

**Claim:** [claim text]
[ABSORBED] · *"[verbatim quote from source]"*

**Claim:** [another claim]
[ABSORBED] · *"[quote]"*

## Notable specifics

[Numbers, formulas, named entities, terminology — preserved verbatim for downstream reference.]

## Open questions

[Things the source mentions but doesn't fully address. Accumulates across `absorbed/` items; surfaced by advisor mode capability D.]
```

Frontmatter is machine-readable so `/synth` and advisor mode can index. Body is human-readable. Original file lives alongside for re-reference.

### 3.5 Downstream reads

When `/synth`, `/ideate`, or the verifier reads an absorbed item, they read `absorbed/[slug]/summary.md` (the summary). The original (`original.{ext}` or `source.md`) is available as a fallback for deeper verification or re-extraction.

---

## 4. Advisor mode

Advisor mode reads `_topic.md` + every `absorbed/[slug]/summary.md`, then exercises analytical judgment. Four capabilities. All conversational — no file writes.

### 4.1 The four capabilities

#### A. Gap surfacing & directional recommendations

The advisor identifies **critical conceptual gaps** in the corpus and points toward what to look for + roughly where to look. Output discipline:

- **What to look for** — the conceptual gap + why it matters for the `_topic.md` goal. Conceptually specific (mechanisms, topics, angles), not citation-specific.
- **Where to look** — *type* of source (paper, blog post, video, tutorial, official docs, lab cookbook, talk, GitHub README) and sometimes *venue* at entity level (ArXiv, Anthropic blog, DeepMind, Stanford CS courses). **Never specific item by title/author/year/URL.**
- **Number is judgment-based.** Surface all critical gaps the corpus has, prioritized by impact on the topic goal. Could be one, could be many. Anti-overwhelm comes from quality bar (only *critical* gaps), not numeric cap.
- If no critical gaps exist, say so: *"corpus looks well-covered for your stated topic goal."*

**Example output:**

> "Your corpus covers attention mechanism design but lacks training dynamics and optimization — these are critical for understanding why transformers work in practice. Look at foundational training-stability papers from the original transformer era, lab blog posts on training tricks (Anthropic, DeepMind), or tutorial walkthroughs. Specific concepts to look for: optimizer choice (Adam vs. AdamW), warmup schedules, gradient clipping, the role of layer normalization."

**What this is not:**

- Not *"find the Vaswani 2017 paper"* (specific citation, hallucination risk).
- Not generic (*"look at AI papers"* — too vague to act on).
- Not a fixed-count list (*"here are 5 things"* — quantity for its own sake).

#### B. Cross-source conflict detection

When extraction mode writes a new item, the advisor scans existing `absorbed/[slug]/summary.md` files for claim conflicts with the new item. Tensions are flagged, not resolved.

**Example output:**

> "This paper claims attention scales linearly with sequence length under sparse-attention modifications. You absorbed an article last week claiming the cost remains quadratic in practice for typical workloads. Flagging the tension — `/synth` can help resolve, or you can dig in directly."

The advisor does not pick a side. Resolution belongs to `/synth` or your own reading.

#### C. Source-set health summary

On request (*"summarize my corpus"*, *"how's my coverage?"*), the advisor reports the corpus state at a navigational level:

- Number of items, date range.
- Source-type distribution (papers / blogs / videos / docs / transcripts).
- Source-tier distribution (Tier 1–4 per `product_spec.md` §6.3).
- Perspectives represented (which labs / authors / schools-of-thought — at entity level, not by specific item).
- **Notable conceptual absences** — gaps you might not have noticed.
- **Topic drift** — whether absorbed items still align with `_topic.md` or have wandered.

~10-line navigational view. Not exhaustive.

#### D. Open-questions surfacing

Each `summary.md` has an "Open questions" section. Advisor mode reads across all absorbed items and surfaces accumulated open questions:

> "Across your absorbed items, these threads are open: (1) the cost-benefit of mixture-of-experts vs. dense models at your scale, (2) how attention sparsity interacts with context-length scaling, (3) whether training-stability tricks generalize across architectures. These are good `/synth` or research-keyword starting points."

Bridges extraction → synthesis. Provides natural entry points for the next layer.

### 4.2 Specificity discipline

Three rules govern advisor output to keep it useful and hallucination-free:

1. **Conceptually specific, citation-free.** Name concepts, mechanisms, topics, angles, terminology — never specific papers / authors / years / URLs.
2. **Source-type and venue tiers of safety:**
   - **Safe (always OK):** source types — *"blog posts on training tricks"*, *"video walkthroughs"*, *"official docs"*, *"lab cookbooks"*.
   - **Mostly safe (entity-level venues):** *"Anthropic's blog"*, *"DeepMind's research page"*, *"Stanford CS courses"*, *"papers in NeurIPS-era"*.
   - **Never:** specific items — *"Karpathy's video on attention"*, *"Vaswani et al. 2017"*, *"the Layer Normalization paper"*.
3. **Active analysis, not list-generation.** Read, think, surface what matters. The voice is a senior research advisor reviewing your work — not a search interface returning hits.

### 4.3 Behavior — advisor mode flow

1. **Detect `cwd`.** Topic folder required (same as extraction mode).
2. **Read state.** Open `_topic.md` + scan all `absorbed/[slug]/summary.md` files.
3. **Identify intent** from the user's prompt — gap surfacing? health summary? open-questions? conflict check?
4. **Analyze.** Apply judgment per the relevant capability (A / B / C / D).
5. **Surface findings.** Conversational, prioritized, specific-without-naming-specifics.
6. **No file writes.** Findings stay in chat. User can run extraction mode on anything worth keeping.
7. **Session-log discipline** — TBD in Pass 2 alongside session-log spec.

---

## 5. Non-capabilities

What `/absorb` explicitly does NOT do:

- **No teaching or explanation.** That's `/synth`. Extraction summarizes; advisor points. Neither teaches.
- **Cross-source connections at the corpus-shape level only.** Advisor mode surfaces conflicts, gaps, patterns about *the source set itself*. Cross-source connections about *content meaning* (*"this concept relates to that concept"*) still belong to `/synth`. The line: advisor analyzes the *shape* of the corpus; `/synth` analyzes the *substance* within.
- **No opinion or evaluation of individual sources.** Extraction is faithful to the source. Advisor mode comments on the corpus, not on whether a given source is "good" or "important."
- **No writes outside `absorbed/`, `_topic.md` (at bootstrap), or `_inbox/` (clearing after process).** Never writes to `synthesis/`, `applications.md`, `projects/`. Advisor mode writes nothing.
- **Chat-drop is typically one item at a time; for batch, prefer `_inbox/` Pattern B.** Agent CAN process multiple items dropped in a single chat invocation, but extraction is heavyweight per item — `_inbox/` Pattern B is the cleaner pattern for batches.
- **No specific-item recommendations in advisor mode.** Per §4.2 — no titles, authors, years, URLs. Conceptual + source-type pointers only.
- **No Gate 2 verifier invocation.** Extraction is below the verifier's threshold; advisor mode produces no captured artifact. (Content absorbed gets verified when synthesis pulls it in.)
- **No inference of source intent or implicit claims in extraction.** If the source says X, the summary says X. If the source implies Y, the summary doesn't claim Y.

---

## 6. Grounding application

### Gate 1 (mandatory — extraction mode)

All five disciplines apply (per `product_spec.md` §5.2):

1. **Cite-and-quote** — every key claim has a verbatim source quote.
2. **Provenance tags** — every claim tagged. Almost all `[ABSORBED]`. `[INFERENCE]` for summarization choices. `[MODEL-*]` rare and minimal.
3. **Read-before-write** — full source read before summary generation.
4. **Forced output structure** — frontmatter + body sections per §3.4.
5. **Distinguish summary from assertion** — *"the paper says X"*, not *"X is true."*

### Gate 1 (advisor mode)

Advisor mode also applies Gate 1, but lighter:

- **Cite the corpus state.** Claims about the corpus (*"you have 8 items, all from 2024"*) are verifiable from the file system.
- **Provenance tags** apply to advisor analysis (most claims are `[INFERENCE]` over absorbed content; occasional `[MODEL-STABLE]` for field-knowledge framings).
- **No fabrication discipline.** Per §4.2: no specific items.

### Gate 2

**Not invoked in either mode.** Extraction is single-source (below verifier threshold). Advisor mode writes no captured artifact. (Absorbed content gets verified when synthesis pulls it in.)

---

## 7. Edge cases

- **Wrong `cwd` with no content** — repo root or domain folder, no content: halt, ask *"Do you want to start a new topic? Or specify which topic to operate in?"*
- **Wrong `cwd` with content** — repo root or domain folder + content: enter **bootstrap mode** (§2.1).
- **Archived topic `cwd`** — warn; require explicit confirmation.
- **Duplicate content (extraction)** — flag; ask to overwrite, skip, or save with `-v2` suffix.
- **Very large source** — section-by-section summarization (>50 pages or >2hr transcript); preserve specifics.
- **Paywall / login wall** — capture accessible portion; flag the barrier.
- **Multimodal embedded** — handle layers separately and label which content came from which layer.
- **Non-English source** — process as-is; note `source_language`; preserve key terms in original.
- **Source agent cannot access** (broken URL, corrupted PDF, low-res image): explain the failure; do not fabricate. Original (if file-based) stays in `_inbox/` for retry.
- **`_inbox/` has unsupported file types** — flag, ask user, do not fabricate extraction.
- **Empty `absorbed/` in advisor mode** — *"no items absorbed yet; advisor mode needs corpus state to analyze. Start with `/absorb [content]` or paste anything you have."*
- **Empty extraction input** (`/absorb` with no chat content + empty `_inbox/`) in a topic folder — fall through to advisor mode + offer help.
- **Empty or invalid extraction input** in bootstrap — *"I don't see content to start a topic with; tell me about what you want to learn first."*

---

## 8. Tone & interaction

Three modes, three tones.

### 8.1 Bootstrap mode

Conversational but efficient. Agent asks 2–3 minimal questions, doesn't over-interrogate.

- *"What should I call this topic?"* (with a suggestion drawn from content if obvious)
- *"Standalone or under a domain?"* (if cwd is repo root)
- *"Brief — what's the goal?"*

Then proceeds. Doesn't ask for elaborate goal statements — minimal viable `_topic.md` is the bar.

### 8.2 Extraction mode

Minimal dialogue. Mostly an extraction operation.

- **Echo the write target** before extracting. One line.
- **Ask only when necessary** — unknown type, duplicate, missing image context, archived topic.
- **Confirm completion** with a one-line result per item.
- **No teaching, no elaboration** in chat. Summaries hold the content; chat confirms what happened.
- **Soft offer of advisor follow-up** after write — single line, skippable.

### 8.3 Advisor mode

Conversational, analytical, discerning. The voice is a **senior research advisor** reviewing your work.

- **No padding.** Don't pre-amble. Surface the analysis directly.
- **Specific without claiming specifics.** Conceptual depth, source-type pointers; no fabricated citations.
- **Discerning quality bar.** Only critical gaps. Be willing to say *"corpus looks well-covered"* if true.
- **Reasoning visible.** Each direction has a *why*.
- **Not a list-machine.** Don't manufacture suggestions to fill space.
- **Invites the user in.** End with a hook: *"want me to dig deeper on any of these?"* or *"want to take any to `/synth`?"*

---

## 9. Residual risks

- **Lossy summarization (extraction).** Agent might drop nuance. Mitigation: "Notable specifics" section captures precise details verbatim; original preserved in `absorbed/[slug]/`, user can re-run absorb with explicit instruction if a fragment was lost.
- **Wrong topic (any mode).** Wrong-`cwd` invocation routes to wrong topic. Mitigation: every invocation echoes the target before doing anything.
- **Mis-detected type (extraction).** Agent guesses wrong type. Mitigation: type visible in frontmatter + echo; user corrects and re-runs.
- **Drift in summary (extraction).** Subtle divergence from source. Mitigation: verifier catches at capture time when content gets pulled into synthesis; original preserved for re-read.
- **Source itself is wrong.** `/absorb` does not validate source correctness. User's call (and `/synth`'s opportunity to cross-check).
- **Advisor venue/source-type drift.** Even without specific item claims, naming venues can have small hallucination surface. Discipline in §4.2 mitigates: source types always safe, venues at entity level mostly safe, specific items never recommended.
- **Bootstrap drift.** Agent might create a topic with too-narrow or too-broad scope from the user's brief intent. Mitigation: agent surfaces the proposed `_topic.md` content before writing; user can refine.
- **`_inbox/` file conflicts.** Two files with the same slug; binary content the agent can't process. Mitigation: agent flags + asks; file stays in `_inbox/` until resolved.

---

**End of `/absorb` specification.**
