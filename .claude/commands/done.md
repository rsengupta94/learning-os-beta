---
description: Signal session close — write the session log for the current command (if applicable) and clean up
---

**Triggered when the user wants to end the current focused session.**

Detect which command was active based on conversation context and act accordingly:

| Context | Action |
|---|---|
| `/synth` active | Write session log to `topics/[cwd]/_sessions/[YYYY-MM-DD-HHMM].md` per `specs/synth.md` §7.2 (what we covered, captures committed, calibration updates, open questions, next-time hooks). Read draft back to user; user can edit verbally; then write. |
| `/ideate` active | Write session log per `specs/ideate.md` §6.2 (directions explored, captures, killed candidates, pending threads, next-time hooks). Same draft-and-confirm flow. |
| `/apply` active | Write session log per `specs/apply.md` §6.2 (spec status, verifications fired, open questions, next-time hooks). Same draft-and-confirm flow. |
| `/absorb` extraction (only) | No session log — extraction is single-transaction. Confirm: *"no session log (absorb extraction mode)."* |
| `/absorb` advisor mode | Brief log if substantive exploration happened; otherwise no log. (Full discipline TBD per `specs/absorb.md` §4.3 step 7.) |
| No clear command context | Confirm: *"nothing active to close."* |
| Session already closed this turn | Echo: *"already closed."* |

After log is written (or skipped), echo: *"Session closed."*

The agent does **not** invoke any other command after `/done`. The user explicitly invokes the next command when ready.
