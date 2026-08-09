---
name: yo
description: Use this skill to send your owner a Yo — a content-free push notification to their phone — when a task completes, you are blocked on user input, or an error needs human review. Also use it to register this agent with Yo the first time (pairing flow or YO_API_KEY). Trigger patterns include "yo me", "notify me when done", "ping me", "send me a push notification".
---

# Yo — push-notify your owner

Yo sends your owner a content-free push notification: "Yo 👉 from _your-name_". The sender identity plus the moment of the ping IS the message. There is no message body — the only optional payload is a `context_url` the notification tap-through opens.

- MCP server (Streamable HTTP): `https://getyo.dev/mcp`
- REST fallback: `POST https://getyo.dev/api/yo`

## Setup: credential discovery

1. **Check for an existing key first (Path B).** Look for a `YO_API_KEY` environment variable or platform secret. Keys look like `yo_<10>_<10>_<32>`. If present, skip registration entirely — use it as the Bearer token.
2. **No key and MCP is available? Self-register (Path A):**
   1. Call the `register_agent` MCP tool with a sensible `proposed_name` derived from your role/context (e.g. the repo name, or platform + project: "devin-home-repo", "cursor-yo-app"). Optionally pass `description` and `platform`.
   2. Present the returned `pairing_url` to the user **verbatim and complete, including the `#…` part** — that fragment is the secret that authorizes approval, so a truncated link cannot be approved. Tell the user to open the whole link (the bare `pairing_code` is only a reference, it approves nothing). Keep the returned `claim_token` secret — never show it to the user or write it anywhere; it is what authorizes you (and only you) to receive the API key and to cancel the pairing.
   3. Poll the `check_registration` MCP tool with the pairing code **and the `claim_token`**, following the returned `poll_hint` (every 5 s for the first 2 minutes, then every 30 s; codes expire after 15 minutes — the key pickup does too, so keep polling until you have the key).
   4. On `approved`, the response includes `api_key` **exactly once**. Store it immediately in your platform's native secret mechanism (e.g. a `YO_API_KEY` secret/env var) and tell the user where it is stored. If you lose it, you must re-register.
   5. The same response includes `paired_with`, the Yo account that approved. **Always tell the user which account you are now connected to**, e.g. "Connected to Yo account alice@example.com." If that is not their account, someone else approved your pairing: call `cancel_registration` with the pairing code and `claim_token` (this revokes the connection and the key), discard the key, and start over with a fresh `register_agent`. If you only notice later and `cancel_registration` returns `invalid`, call `revoke_self` (Bearer-authenticated with the API key, `confirm: true`) — it permanently revokes the key at any time.
3. **No key and interactive pairing is impossible** (stateless platform, no MCP): ask the user to open the Yo app (`https://getyo.dev/app`), tap **+ Add agent**, and give you the key as a `YO_API_KEY` secret/env var.

## Procedure: sending a yo

Call the `yo` MCP tool (authenticated with your API key as the Bearer token), or REST:

```bash
curl -X POST https://getyo.dev/api/yo \
  -H "Authorization: Bearer $YO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"context_url": "https://example.com/your/session"}'
```

**When to yo:** task complete; blocked on user input; error requiring human review. Explicitly **NOT** for routine progress updates.

**context_url:** include one when there is a natural link — your session URL, a PR URL, or a native deep link (e.g. `cursor://...`). Omit it otherwise.

## Gotchas

- **Show the whole pairing URL, never just the code.** Anyone who only sees the code cannot approve, bind or hijack the pairing — that only holds while you don't hand out the fragment separately.
- **Always confirm `paired_with` with the user, and cancel on a mismatch.** `cancel_registration` works both before and after approval — until the pairing expires, plus 15 minutes after you pick up the key. Discover a mismatch even later? `revoke_self` (authenticated with your API key) revokes the connection at any time.
- **Rate limit: 1 yo per 30 seconds per agent.** On a `rate_limited` error, do NOT retry in a loop — either wait the returned `retry_after_seconds` once, or drop the yo.
- A `revoked` error means the owner disconnected you; stop sending and tell the user.
- `ok: true` with `delivered_to: 0` means the yo was recorded but the owner has no push-enabled devices; relay the warning to the user once.
- Never write the API key into files committed to a repository; use the platform's secret mechanism.
