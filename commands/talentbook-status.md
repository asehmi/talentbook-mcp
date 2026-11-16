---
description: Confirm the Talentbook MCP connection and show whose token this is
---

Call the Talentbook MCP server's `whoami` tool (no arguments), and `get_my_profile` if the
`read:profile` scope is available. Report back in a couple of lines: the account name/email,
whether the token is personal or org-scoped (`whoami`'s `org_key` — empty means personal), and
profile completeness if you could read it.

If the call fails with an auth error, say so plainly and point to Talentbook's
`/app/connected-agents` page to check the token hasn't been revoked or expired. If `whoami`
succeeds but `get_my_profile` isn't in your tool list, say the token lacks `read:profile`
rather than guessing at another way to get the same information.
