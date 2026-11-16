---
name: talentbook
description: Use Talentbook's MCP tools well — what each scope unlocks, the idempotency-key and rate-limit rules every write/multi-step flow must follow, and the draft-then-approve pattern for anything written on the user's behalf. Trigger whenever a talentbook:* tool is available and the user asks about their goals, skills, OKRs, journal, consultations, team, org roster, baselines, or work-job templates — or asks to run a check-in, a consultation, a team digest, a roster sync, or a baseline design session.
---

# Using the Talentbook MCP connector

This plugin connects you to one real Talentbook account's data over MCP. Every tool call acts
on whoever the configured access token belongs to — there's no way to ask for someone else's
data by name, and you don't need to pass an identity argument anywhere.

## What you can see depends on the token's scopes

`tools/list` only shows the tools the connected token's scopes unlock. If a tool you expect
(say, `create_goal`) isn't in your tool list, the token wasn't issued that scope — don't guess
around it by trying a different tool; tell the user which scope is missing and point them to
Talentbook's `/app/connected-agents` page to reissue a token with it added.

| Scope | Unlocks |
|---|---|
| *(any valid token)* | `whoami` (whose token this is), `get_all_consultation_specs`, `get_available_org_types` — catalog/identity metadata, not personal data |
| `read:profile` | `get_my_profile` |
| `read:goals` / `write:goals` | `get_my_goals` / `create_goal`, `update_goal` |
| `read:skills` / `write:skills` | `get_my_skills`, `get_my_gap_analysis` / `track_skill_level` |
| `read:okrs` / `write:okrs` | `get_my_okrs` / `create_okr_commitment`, `update_okr_commitment` |
| `read:journal` / `write:journal` | `get_my_journal` / `add_journal_entry`, `update_journal_entry` |
| `read:canvas` / `write:canvas` | `get_my_canvases`, `get_canvas_content` / `create_canvas`, `rename_canvas`, `set_canvas_box`, `delete_canvas` |
| `read:consultations` / `write:consultations` | `get_my_consultations`, `get_available_consultations`, `get_next_consultation_question`, `get_consultation_result`, `get_consultation_qa_trace` / `start_consultation`, `answer_consultation_question`, `generate_consultation_report` |
| `read:team` | `get_my_team_summary` (manager-only — re-verified server-side on every call) |
| `read:org_roster` / `write:org_roster` | `get_org_roster` / `sync_org_roster_role` (org-scoped token only) |
| `read:org_baselines` / `write:org_baselines` | role-baseline design tools (org-scoped) |
| `read:org_work_jobs` / `write:org_work_jobs` | work-job-template design tools (org-scoped) |
| `read:skill_catalog` / `read:skill_details` / `write:skill_details` | the shared competency model (admin/`su`-issued tokens only) |

Org-scoped tools (`org_roster`, `org_baselines`, `org_work_jobs`) only work with an org-level
token, not a personal one — `whoami`'s `org_key` field tells you which kind you're holding
(`""` means personal).

## Every write tool needs an idempotency key

Every write tool takes `idempotency_key` as its first argument. Generate a fresh UUID **once
per logical intent** (e.g. once for "create this specific goal"), not once per call attempt.
If a call fails on a network error and you retry it, reuse the *same* key — that's what makes
the retry safe against double-writing. Never reuse a key across two genuinely different
writes; the server will replay the first result instead of applying the second one.

## Rate limiting — pace yourself on multi-step flows

The server allows roughly 1 request/second with a burst of 10. A single-tool call is fine. A
multi-step flow — most notably driving a full consultation, which costs two calls per question
(`get_next_consultation_question` + `answer_consultation_question`, easily 15-20 calls total) —
can exceed the burst. If a call returns a `-32000 "Rate limit exceeded"` error, back off briefly
and retry that one call; don't treat it as a real failure or abandon the flow.

## Draft first, write second — always show your work before a write tool fires

This is the most important habit for this connector, not a style preference. Before calling
any write tool on the user's behalf:

1. **Read first.** Pull the user's actual relevant data with the read tools
   (`get_my_goals`, `get_my_journal`, `get_my_skills`, `get_my_gap_analysis`, etc.) rather than
   inventing content.
2. **Draft, grounded in what you just read.** A journal entry, a consultation answer, a new
   goal — whatever you're about to write — should cite something real and specific (an actual
   dated entry, an actual skill level, an actual goal title), not generic filler.
3. **Show the draft and wait for approval** before calling the write tool. The user can accept,
   edit, or redirect it.
4. **Only then write it** — with the exact approved content, and a freshly generated
   `idempotency_key`.

This consistently produces sharper output than either extreme (an agent silently writing
unsupervised, or a user typing everything from scratch with no help recalling what they already
said elsewhere in the app) — confirmed in practice running a full consultation this way: every
finding in the resulting report cited something real instead of reading as generic reflection.

## Ownership is already enforced server-side

Every tool that takes an id (`goal_id`, `okr_id`, `canvas_id`, `consultation_id`, ...) already
refuses one that isn't the connected token's own. You don't need to pre-check ownership
yourself before calling a tool — just call it, and treat an ownership-style error as "this
wasn't theirs," not as a bug to work around.

## Consultations follow one fixed flow

`start_consultation(spec_key)` → loop `get_next_consultation_question(consultation_id)` /
`answer_consultation_question(...)` until the former returns `{"done": true}` →
`generate_consultation_report(consultation_id)` (queues a background job — don't wait inline)
→ poll `get_consultation_result(consultation_id)` until its `status` is `"complete"` or
`"failed"`. `value` for `answer_consultation_question` is the exact option label the question
offered (a string), or a list of labels for a `"multiple"` question — always match what
`get_next_consultation_question` just returned, never invent a different phrasing.

## Common requests this connector is good for

- **A reflective check-in**: read recent goals/skills/journal, draft a grounded weekly
  reflection, approve, then `add_journal_entry`.
- **Running a consultation**: drive the fixed flow above, drafting each answer from the user's
  real tracked data per the draft-then-approve rule.
- **A manager's team digest**: `get_my_team_summary` per direct report, composed into one
  written summary (requires `read:team`, and only surfaces reports who actually report to this
  token's owner).
- **A roster sync from an HRIS export**: map the export to `{"email", "reports_to"}` pairs and
  call `sync_org_roster_role` once as a batch (requires an org-scoped `write:org_roster` token)
  — the whole batch either applies or changes nothing, so this is safe to drive directly rather
  than row-by-row.
- **Designing an org role baseline or work-job template**: preview a built-in catalog
  (`get_available_org_types` → `get_builtin_catalog_roles`/`_role_skills`), then build a
  customized version (`create_org_baseline` → `customize_org_baseline_role` per role/seniority)
  conversationally (requires an org-scoped `write:org_baselines` or `write:org_work_jobs` token).
- **Gap-aware goal planning**: `get_my_gap_analysis` to find the biggest gaps against the
  user's role, draft candidate goals tagged with the relevant `skill_keys`, approve, then
  `create_goal`.

If a request needs a scope the connected token doesn't have, say so plainly rather than
attempting a workaround — the fix is reissuing the token at `/app/connected-agents`, not a
different tool call.
