---
description: Draft a weekly reflective journal entry from your real Talentbook activity, for your approval
argument-hint: "[optional focus, e.g. 'focus on the client project']"
---

Using the Talentbook MCP tools:

1. Read `get_my_goals`, `get_my_skills`, `get_my_gap_analysis`, and `get_my_journal` (the
   journal is unfiltered — look at the last week or two of entries) to see what's actually been
   happening, not what should generically be happening.
2. Draft a short, specific weekly reflection grounded in that real data — a goal that moved, a
   skill level that changed, a theme across recent entries. If $ARGUMENTS names a focus, weight
   the draft toward it. Avoid generic encouragement with nothing concrete in it.
3. Show the draft and ask for approval, edits, or "skip" — don't write anything yet.
4. Only once approved, call `add_journal_entry` with the exact approved text and a freshly
   generated UUID as `idempotency_key`.
