---
description: Bridge from idea to build — convert a chosen candidate into a verified, executable spec.md
argument-hint: [candidate-slug from applications.md, or empty to be prompted]
---

**Before responding, read `specs/apply.md` in full.** That spec carries the operational detail — candidate-anchored authoring, auto-fired live-doc verification, the WHAT/HOW principle, four-rule authoring discipline (agent drives proposals / concrete specification / live-doc verification auto-fires / bounded scope), 12-step development flow, spec.md output format (frontmatter + 13 sections), verifier checks. Follow its disciplines throughout.

**Operating context:**
- Current working directory should be a topic folder (contains `_topic.md`). If not, halt and ask which topic.
- **Session start, in order:** read `synthesis/applications.md` (locate the candidate), `_topic.md`, `synthesis/[concept].md` files, relevant `absorbed/` items, existing `projects/[candidate-slug]/spec.md` if present (resume mode), recent `/apply` session logs.
- **If candidate slug is provided in arguments**, locate it directly in `applications.md`.
- **If no slug provided**, list available candidates from both buckets (main + adjacent) and ask user to pick.
- **If candidate is from the adjacent bucket**, warn that topic isn't load-bearing for current topic; offer to continue here or pivot to a more anchored topic.

**Discipline reminders (full detail in spec):**
- **Discipline in one line: detailed on *WHAT* to build, free hand on *HOW*.** Architecture, data shapes, interface contracts, test plan, stack — fully specified. File organization, internal code structure, framework-specific implementation patterns — left to build session's judgment.
- **The user is not an engineer.** Agent does the technical heavy lifting — proposes stack, architecture, data shapes, interface contracts, test plan, build phases, done states, risks. User retains scope/product authority.
- **Auto-fired live-doc verification** on technical claims (libraries, APIs, model IDs, endpoint URLs, framework versions) — no permission ask needed for this (locked exception per `specs/research.md` §2.3).
- **Trust via three sources:** verification + visible reasoning + honest uncertainty. Flag low-confidence calls and verification failures explicitly.
- **Voice:** practical builder — concrete, technical, verifying, scope-disciplined. Push-back on sprawl is constructive, not adversarial.

Candidate slug: $ARGUMENTS
