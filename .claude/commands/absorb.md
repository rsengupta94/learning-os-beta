---
description: Three-mode entry point — bootstrap a new topic (repo root + content), extract into current topic (in-topic + content via chat or _inbox/), or analyze the corpus in advisor mode (in-topic, no content)
argument-hint: [optional context note, or empty for advisor mode]
---

**Before responding, read `specs/absorb.md` in full.** That spec carries the operational detail — three modes (bootstrap / extraction / advisor), input types (URL / PDF / image / plain text / video transcript / unknown), Pattern A (chat-drop) vs. Pattern B (`_inbox/`-drop), output structure (folder-per-item with `summary.md` + `original.{ext}`), advisor capabilities, edge cases, tone. Follow its disciplines throughout.

**Operating context — dispatch by `cwd` + content:**

| `cwd` state | Content source | Mode |
|---|---|---|
| Topic folder (`_topic.md` present) | Chat drop | **Extraction (Pattern A)** — write to `absorbed/[slug]/{summary.md, original.{ext}}` |
| Topic folder | Files in `_inbox/` | **Extraction (Pattern B)** — process each, move from `_inbox/` to `absorbed/[slug]/original.{ext}` + write `summary.md` |
| Topic folder | Neither | **Advisor** — analyze corpus, surface gaps/conflicts/health; no writes |
| Repo root or domain folder | Chat drop (or stated intent) | **Bootstrap** — ask 2–3 questions (name, domain/standalone, goal), create topic structure incl. `_topic.md` and subfolders (`_inbox/`, `_sessions/`, `absorbed/`, `synthesis/`, `projects/`), then extract |
| Repo root or domain folder | None | Halt; ask *"Do you want to start a new topic?"* |
| Archived topic folder | Any | Warn; require explicit confirmation |

**Discipline reminders (full detail in spec):**
- Always **preserve the original** in `absorbed/[slug]/` alongside the summary — for PDFs/images/text use `original.{ext}`, for URLs use `source.md` (URL + access date + fetched content).
- Provenance tag every claim per `specs/verifier.md` §3.
- Advisor mode: **conceptually specific, citation-free** (no specific paper titles/authors/years). Source types and entity-level venues are safe; specific items are never recommended.
- Bootstrap mode: keep questions minimal (2–3 max); proposed `_topic.md` shown for user confirmation before write.
- No teaching (`/synth` territory), no gap analysis on user state, no writes outside the topic's `absorbed/`, `_topic.md` (at bootstrap), or `_inbox/` clearance.

User input: $ARGUMENTS
