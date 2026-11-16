---
description: Draft new goals targeting your biggest skill gaps, for your approval
argument-hint: "[optional: how many goals to propose, default 3]"
---

Using the Talentbook MCP tools (requires `read:skills`, `write:goals`):

1. Call `get_my_gap_analysis` to see required level vs. self-assessed level for every skill.
2. Pick the largest gaps — $ARGUMENTS gives a count if provided, otherwise propose up to 3.
3. For each, draft a candidate goal: a specific title and detail tied to actually closing that
   gap, category `"work"` unless the gap is clearly personal, and `skill_keys` set to that
   skill's key (check `get_my_skills`/`get_skill_catalog` for the exact key if unsure).
4. Show every draft together and get approval, edits, or "skip" on each — don't create
   anything yet.
5. For each approved draft, call `create_goal` with a freshly generated UUID as
   `idempotency_key`.
