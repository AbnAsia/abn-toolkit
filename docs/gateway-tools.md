# ABN gateway — tool contract

The MCP server the skills call. This file is the agreement between the toolkit
(public) and the gateway (private): if a skill references a tool or a field
that is not here, the skill is wrong.

Endpoint: `https://mcp.abnasia.org/mcp` — MCP over HTTP, streamable transport.

## Auth, in two phases

**Phase 1 — API key (shipped first).** The account holder creates a key at
`https://one.abnasia.org/abn-gateway`, sends it as `X-Api-Key`. One paste, works
in every agent, no browser round trip. Keys are scoped, listed, revocable.

**Phase 2 — the gateway as its own authorization server.** OneID (Frappe
OAuth2) publishes no `registration_endpoint`, no `code_challenge_methods_supported`
and no `jwks_uri`, so an MCP client cannot register itself there. The gateway
therefore federates:

```
agent ──DCR──▶ gateway  (/.well-known/oauth-authorization-server, PKCE S256)
                  │
                  └── one pre-registered client ──▶ OneID authorize/token
```

- `POST /register` — dynamic client registration (RFC 7591). Public clients,
  `token_endpoint_auth_method: none`, redirect URIs restricted to loopback and
  the known agent schemes. One client per registration, reaped after 30 days
  unused.
- `GET /authorize` — stores the agent's `client_id`, `state` and PKCE challenge,
  redirects to OneID's authorize endpoint with the gateway's own client.
- `GET /callback` — exchanges the OneID code, reads the identity from
  `userinfo`, links it to the agent's client, issues the gateway's own tokens.
- `POST /token` — `authorization_code` with PKCE, and `refresh_token`.
- Tokens: JWT, HS256, 1 h access, 30 day refresh with rotation. Identity is
  re-checked against OneID `introspect` on refresh, cached 60 s — HS256 means
  nothing can be verified offline.
- `POST /revoke` — RFC 7009.
- `/.well-known/oauth-protected-resource` — points at the gateway as the AS.

Prospects may self-register in OneID, so an unknown email is a valid new
account, not an error.

## Rules that apply to every tool

- **Errors are prompts.** A bare `403` is a bug. Every failure returns
  `{ "error": { "code", "message", "next" } }` where `next` is an instruction
  the agent can act on, e.g. *"Publishing needs a client account. Call
  get_services to see what this account can do, or ask ABN at
  contact@abnasia.org."*
- **Idempotency.** `create_rfp` and `publish_article` accept
  `idempotency_key`; a repeat with the same key returns the first result.
- **Audit.** Every call logs account, tool, argument hash, result code,
  duration. Never argument bodies that carry personal data.
- **Quotas.** Per account and per tool. `create_rfp`: 5/day, 20/month.
  `publish_article`: 50/day. Reads: 600/hour.
- **No secrets out.** No tool returns a token, key or password except
  `create_api_key`, which returns the key once and never again.

## Tools

### `get_services` — free, anonymous

The catalog. Safe to call before signing in; this is what a stranger calls
first.

```jsonc
// input
{ "offer": "secure-software" }   // optional; omit for the whole catalog
// output
{
  "catalog_version": "0.1.0",
  "updated": "2026-11-05",
  "offers": [ { "id", "name", "summary", "for_who", "deliverables",
                "engagement", "good_fit_when", "not_a_fit_when", "next_step" } ],
  "how_abn_works": { "steps": [], "contracts": [], "payment", "nda" },
  "contact": "contact@abnasia.org"
}
```

### `get_account` — free, signed in

Who this is, what they may call, what is left.

```jsonc
// output
{
  "account_id": "…",
  "identity": { "email", "name", "organisation", "locale" },
  "kind": "prospect" | "client" | "partner" | "staff",
  "entitlements": ["rfp.create", "project.read", "publish.article", "keys.create"],
  "quotas": { "create_rfp": { "used_today": 0, "limit_today": 5 } },
  "gateway": { "version": "0.1.0", "catalog_version": "0.1.0" }
}
```

### `create_rfp` — prospects, clients

Turns a brief into a **Frappe CRM Lead** and returns its id.

```jsonc
// input
{
  "brief": {
    "title": "string, ≤120",
    "problem": "string — what is broken or missing today",
    "outcome": "string — what success looks like, measurably",
    "scope_hint": ["secure-software", "frappe-platform"],  // offer ids
    "constraints": { "deadline": "YYYY-MM-DD|null", "budget_band": "string|null",
                     "must_use": ["…"], "must_not": ["…"] },
    "stakeholders": [{ "role": "…", "decision": "…" }],
    "existing_systems": ["…"],
    "data_sensitivity": "none|internal|personal|regulated",
    "locale": "en"
  },
  "contact": { "email", "name", "organisation", "phone" },
  "idempotency_key": "uuid"
}
// output
{
  "rfp_id": "CRM-LEAD-000123",
  "status": "received",
  "crm_url": "https://one.abnasia.org/crm/lead/CRM-LEAD-000123",
  "next": "ABN replies within three business days. Nothing else is needed from you."
}
```

The lead is created with source `agent-gateway`, the brief attached as a
document, and the offer ids mapped to the CRM's service field. Duplicate
detection: same email + same title within 7 days returns the existing lead.

### `get_rfp` — owner only

```jsonc
// input  { "rfp_id": "CRM-LEAD-000123" }
// output { "rfp_id", "status": "received|scoping|proposal_sent|won|lost",
//          "updated", "next", "owner": { "name" } | null }
```

### `get_project_status` — clients

```jsonc
// input  { "project_id": "…" }   // omit for the list
// output {
//   "project_id", "name", "state": "discovery|build|handover|support",
//   "milestones": [ { "name", "state", "due", "delivered_on" } ],
//   "next", "blockers": [ "…" ]
// }
```

### `publish_article` — staff, `publish.article`

The news.abnasia.org pipeline: one English source, translated, validated and
published across locales.

```jsonc
// input
{
  "title": "string",
  "summary": "string ≤300",
  "body_markdown": "string",
  "tags": ["tech-week", "abn-news"],
  "locales": ["en", "vi", "fr", "de", "es", "id", "jp", "cn", "th"],
  "idempotency_key": "uuid"
}
// output
{
  "localeid": "en-…-4471",
  "published": ["en", "vi", "…"],
  "skipped": [ { "locale": "jp", "reason": "translation truncated" } ],
  "urls": { "en": "https://news.abnasia.org/en/blog/posts/…" }
}
```

A locale whose translation fails validation is skipped and reported, never
published half-written.

### `create_api_key` — clients, `keys.create`

```jsonc
// input  { "label": "acme-prod", "scopes": ["project.read"], "expires_in_days": 365 }
// output { "key": "abn_live_…", "key_id": "…", "scopes": [], "expires_at": "…" }
```

The key is returned once. `list_api_keys` and `revoke_api_key` exist for the
account page, not for agents.

## Error codes

| Code | HTTP | `next` says |
|---|---|---|
| `auth_required` | 401 | Sign in again: `… mcp auth abn` |
| `entitlement_missing` | 403 | Which entitlement is needed and how to ask for it |
| `quota_exceeded` | 429 | When it resets, and that the call was not made |
| `invalid_brief` | 422 | Which field, and what a good value looks like |
| `duplicate_rfp` | 200 | The existing `rfp_id` (not an error to the agent) |
| `crm_unavailable` | 503 | That the brief was saved locally and will be filed, with a retry time |
| `catalog_stale` | 200 | That the skills should be updated, with the version |
