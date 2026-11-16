---
description: Compose one written weekly digest across your direct reports' latest progress summaries
argument-hint: "[direct reports, comma-separated names or emails — otherwise asks]"
---

Using the Talentbook MCP tools (requires `read:team`):

1. If $ARGUMENTS lists names or emails, use that list. Otherwise ask who the direct reports
   are — there's no "list my reports" tool in this connector, so the names/emails have to come
   from the user.
2. Call `get_my_team_summary(direct_report=...)` once per name/email. It re-verifies
   server-side that each one actually reports to the connected token's owner, so don't
   pre-filter the list yourself.
3. For anyone it returns an error for (no matching report, or no summary generated yet for
   them), note that plainly rather than silently dropping them from the digest.
4. Compose one written digest across everyone it succeeded for. Pull out what's common across
   the team (a shared risk, a recurring skill gap) as well as what's specific to each person —
   don't just concatenate the raw summaries one after another.
