# research — Pass 2 Specification

**Owner:** Rajarshi Sengupta
**Status:** Pass 2 design (locked).
**Relates to:** §6 of `product_spec.md` (high-level summary); this file carries operational detail.

---

## 1. Purpose & framing

**Research is a capability, not a slash command.** Like Claude Code's `WebSearch` and `WebFetch` tools, or the `ultrathink` keyword pattern, it fires when needed from any active conversation. It does not have its own session or top-level invocation.

The capability serves three jobs:

1. Fill gaps surfaced in `/synth` teaching, when absorbed sources don't cover a concept.
2. Validate technical feasibility of candidate applications in `/ideate`.
3. Verify technical claims (libraries, APIs, model names) in `/apply` before a spec ships.

`/absorb` advisor mode does **not** fire research — by design, it points toward conceptual gaps and source types rather than specific items. There's nothing specific to validate.

Research findings ground the agent. They are not blindly trusted — Tier-based source priority and verifier re-fetch at capture time still apply.

---

## 2. Invocation paths

### 2.1 User keyword

In any chat session, the user can trigger research with a natural-language phrase:

- *"research X"*
- *"can you research this?"*
- *"look this up"*
- *"verify that externally"*

The agent recognizes the intent + scope, **confirms the scope** if ambiguous (*"you want me to research [specific thing], right?"*), then fires.

For unambiguous cases (e.g., immediately after a specific claim), the agent fires without asking.

### 2.2 Agent-introspective with permission

The agent self-detects when it needs external grounding and **asks permission** before firing:

> *"This claim about [X] is in uncertain territory — [specific reason]. Should I verify externally?"*

The user says yes/no. No autonomous firing.

### 2.3 Auto-fire exception — `/apply` technical verification

**Inside `/apply`, live-doc verification on technical claims fires automatically.** No permission ask. This covers:

- Library names and versions (does the named package exist?)
- API signatures (is the function/method signature correct?)
- Model identifiers (does the named model version exist as claimed?)
- Endpoint URLs and parameter schemas

The cost of skipping verification on these is too high — wrong specs cascade into build failures. The user implicitly authorizes this by invoking `/apply`.

The agent announces the auto-fire (*"verifying [API X] against live docs"*) so the user knows it's happening, but doesn't pause for permission.

---

## 3. Trigger discipline (when the agent considers firing)

The agent considers asking permission to fire research when **any** of these conditions hold:

1. About to teach a `[MODEL-UNCERTAIN]` claim the user is likely to build understanding on.
2. Source conflict detected (e.g., absorbed item A contradicts absorbed item B on a specific claim).
3. Claim about to be captured fails the writer's internal confidence check.
4. User asks a question that hits the agent's parametric knowledge boundary.
5. *(Auto-fire, no ask)* `/apply` is about to write a spec with a technical claim.

This is **discipline, not a rigid checklist.** The agent uses judgment. The verifier catches whatever introspection missed at capture time.

---

## 4. The permission protocol

When the agent decides to ask, the ask is **specific about what and why** — yes/no, low friction:

| Trigger | Ask format |
|---|---|
| `[MODEL-UNCERTAIN]` claim | *"This claim about [X] is in uncertain territory — [recent / attribution-prone / specific number]. Should I verify externally before continuing?"* |
| Source conflict | *"The item you just absorbed conflicts with [Y from absorbed/] on [specific claim]. Should I research this to resolve?"* |
| Knowledge boundary | *"I don't have strong knowledge of [Z]. Should I search externally?"* |
| Confidence-check fail | *"Before capturing this, I'd like to verify [specific point]. Research it?"* |
| Advisor validation | *"I think [paper title, author, year] is relevant. Should I quickly verify this exists before recommending?"* |

The user can say *"yes"*, *"no"*, *"yes but quickly"*, *"yes and absorb the finding"*, or pre-authorize broader scope (§5).

---

## 5. Session-level pre-authorization (optional)

The user can grant the agent autonomy for the current session at any point:

- *"go ahead and research whenever you need to without asking"* — full pre-auth.
- *"for all `[MODEL-UNCERTAIN]` claims this session, just verify"* — scoped pre-auth by tag.
- *"verify advisor suggestions without asking"* — scoped pre-auth by trigger type.

Pre-authorization is **session-scoped** — it lapses when the session ends (logged in the session log, not persisted across sessions).

The user can revoke at any time: *"stop researching without asking"* — agent returns to ask-per-fire mode.

This is a flow-optimization pattern. For users in deep teaching mode who want minimal interruptions, it removes friction.

---

## 6. Findings handling

When research fires (whether user-keyword, agent-asked, or auto-fired), findings flow as follows:

1. **Agent performs external search** — applies source priority (§7) and budgets (§9).
2. **Agent synthesizes findings in chat** — with cite-and-quote discipline (§9), `[RESEARCH]` provenance tag, and source URLs.
3. **Findings stay in chat by default.** The agent does NOT auto-write findings to `absorbed/` or any persistent file. The findings exist in the conversation, available for the agent to teach from / cite / use in the current session.
4. **User can explicitly `/absorb`** any finding worth keeping. If user says *"absorb that"* or runs `/absorb` on a finding URL, the standard absorb flow runs — and the item enters `absorbed/` as a fully provenance-tagged source.
5. **Session log captures key findings.** When the session ends, the session log notes what was researched and the key findings — even if not absorbed. Preserves cross-session continuity.

This discipline preserves the user's **curatorial control** over the corpus. Agent-initiated research can never bloat `absorbed/` autonomously.

---

## 7. Source priority enforcement

When firing research, the agent prefers higher-tier sources per `product_spec.md` §6.3:

- **Tier 1:** peer-reviewed papers, ArXiv preprints from established authors, official lab documentation, API official docs.
- **Tier 2:** lab blog posts, reputable technical newsletters, recognized industry research orgs.
- **Tier 3:** general technical blogs, well-known practitioner posts.
- **Tier 4 (last resort, flagged):** social media, anonymous blogs, model parametric knowledge.

The agent fetches multiple sources where the claim is important, and prefers higher tiers when sources conflict. If a Tier 4 source is the only available evidence, the finding is flagged as low-confidence and the user is told.

---

## 8. Quality disciplines per fire

Every research fire follows these disciplines:

- **Multi-source for important claims.** Single-source claims are flagged with `[RESEARCH-SINGLE-SOURCE]` annotation in the finding.
- **Recency stamp.** Every finding includes the source's publication date. Items that look stale relative to the topic's pace get flagged.
- **Cite-and-quote.** Every claim in the finding has a verbatim quote from the source — verifier re-checkable at capture time.
- **Cross-validation against absorbed sources.** If research output contradicts an existing absorbed item, agent flags the contradiction — doesn't pick a side.
- **Per-fire soft budget.** Each fire is bounded by max searches (~5), max time (~2 min), max sources fetched (~5). Agent asks before extending. User can override per-session.

---

## 9. Verifier interaction

Findings produce claims tagged `[RESEARCH]`. When those claims get **captured into a synthesis note / candidate / spec**, the verifier (Gate 2) applies:

- **Re-fetch the cited URL** — does it still resolve? Does the page still contain the quoted text?
- **Quote-match.** Verbatim check.
- **Recency re-check** — is the source still current relative to the topic?
- **Conflict check** — does the captured claim conflict with `[ABSORBED]` items in the corpus?

See `specs/verifier.md` §4–§5 for per-tag verification protocol. `[RESEARCH]` claims have the second-highest trust tier (below `[ABSORBED]`, above `[INFERENCE]`).

---

## 10. Codex compatibility

Claude Code provides `WebSearch` and `WebFetch` as first-class tools. Codex's web access is provided differently but functionally equivalent (tool-augmented browsing or sub-agent web fetch).

The research capability works in both environments — the underlying tool differs, the discipline is identical. Where Codex lacks an equivalent, the agent falls back to:

- Cite-and-quote from parametric knowledge with explicit `[MODEL-UNCERTAIN]` tagging.
- User prompted to perform the external check themselves and paste findings.

Documented in `AGENTS.md`.

---

## 11. What this descopes from prior /research design

Cuts from the prior `/research` slash command spec:

- **No `corpus-overview.md`** as a standard artifact. If a corpus overview is needed, `/synth` can produce one on demand (Pass 2 work for `/synth`).
- **No dedicated research session.** Research fires inline, conversational, returns to current flow.
- **No per-session budgets ceremony.** Replaced by per-fire soft caps + user-controlled pre-auth.
- **No confirmation gates for long sweeps.** Replaced by per-fire permission ask.
- **No topic-launch sweep ritual.** Topic launch uses `/absorb` advisor mode to identify items + manual absorb (or user keyword to fetch).

---

## 12. Residual risks

- **Hallucinated findings.** Agent fires research, web search returns plausible-looking but wrong results (LLM-generated content increasingly populates the web). Mitigation: source-priority enforcement + multi-source + verifier re-fetch at capture time.
- **Stale findings.** Source was correct when published but field has moved. Mitigation: recency stamp + verifier check at capture time.
- **Under-firing.** User declines a legitimate permission ask to keep flow moving; misses grounding. Mitigation: verifier catches at capture time; user can verbally trigger a verification check mid-dialogue (*"verify that"*, *"check what you just said"*).
- **Over-asking.** Agent over-conservatively asks permission too often, becomes noisy. Mitigation: trigger discipline (§3); user can pre-authorize (§5).
- **Findings sprawl.** Lots of research happens in-session; findings exist in chat but aren't captured. Mitigation: session log captures key findings; user can `/absorb` anything worth keeping.
- **Auto-fire abuse in `/apply`.** Auto-fire could in theory be exploited if `/apply` writer over-eagerly classifies things as "technical claims" needing verification. Mitigation: trigger scope is narrow — only library names, API signatures, model IDs, endpoint URLs. Drift here is a Pass 2 prompt-engineering concern.

The user's own reading remains the final gate. Research is the second of three (writer → research → verifier → user).

---

**End of research capability specification.**
