---
description: Run a guided consultation end to end, drafting each answer from your real Talentbook data for approval
argument-hint: "[optional spec_key — otherwise lists what's available and asks]"
---

Drive one guided consultation to completion using the Talentbook MCP tools, following the
draft-then-approve pattern from `skills/talentbook/SKILL.md`.

1. If $ARGUMENTS gives a spec_key, use it. Otherwise call `get_available_consultations` and
   ask which one to run — a themed group lists its real startable children under its own
   `children` field; offer those, never the group's own spec_key.
2. Call `start_consultation` with a fresh `idempotency_key`; note the returned
   `consultation_id`.
3. Loop: call `get_next_consultation_question(consultation_id)`. For each question returned
   (until it reports `{"done": true}`):
   - Read the user's relevant real data first — `get_my_goals`, `get_my_journal`,
     `get_my_skills`, `get_my_gap_analysis` as relevant to the question.
   - Draft a candidate answer grounded in that data, matching the question's `type`/`options`
     exactly (the exact label(s) offered, not a paraphrase).
   - Show the draft and get approval, an edit, or a redirect — never submit before approval.
   - Call `answer_consultation_question` with the approved value and a fresh
     `idempotency_key`.
   - If a call returns a rate-limit error (`-32000`), back off briefly and retry that one
     call — don't treat it as a real failure.
4. Once `get_next_consultation_question` reports `{"done": true}`, call
   `generate_consultation_report`.
5. Poll `get_consultation_result(consultation_id)` until its `status` is `"complete"` or
   `"failed"`, then show the report.

If the session is interrupted partway, keep every answer you've already had approved. If
`get_next_consultation_question`/`get_my_consultations` later comes back with no trace of this
consultation_id, don't treat it as a bug — just `start_consultation` again and replay the same
approved answers rather than re-drafting from scratch.
