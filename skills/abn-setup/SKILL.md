---
name: abn-setup
description: Connect and check what ABN needs - the abn server, sign-in, account, catalog version, quotas, updates. Use when setting up ABN, when a tool fails, or when the catalog looks stale. NOT for matching an offer (abn-services) or writing a brief (scope-project).
license: Proprietary
metadata:
  owner: AbnAsia
  version: 0.1.0
allowed-tools: Bash(*/scripts/check-gateway) Bash(*/scripts/detect-host)
---

# Set up ABN

You connect the `abn` server if it is missing, check everything the other skills
need, fix what you can, and give the person the one step for each thing only
they can do. Keep it light: a ready person sees one line, not a report.

## 0. Updates

Run `scripts/check-gateway` once. It prints one line per check in the shape
`ok|fix|info <name> <message>`. Never stop for an `info` line.

If it says a newer toolkit version is out, give the update command once and
carry on — what is installed keeps working.

## 1. Is the server there?

Call `get_services`. Three outcomes:

- **It answers** → connected. Remember its `catalog_version` for step 4.
- **No `abn` tools at all** → the server is not added: go to step 2.
- **A tool fails with `auth_required`** → connected but not signed in: skip to
  step 4.

## 2. Add the server, only when it is missing

**Ask once:** "Connect ABN's server to <your agent>? It reads the live catalog
and sends a brief to ABN." with **Yes, connect it** and **Not now**.

On no: change nothing. Say the skills still work and briefs go by email.
Stop here.

On yes: add a remote MCP server named `abn` at `https://mcp.abnasia.org/mcp`.
Run `scripts/detect-host` for the agent and its config file. Check first that no
`abn` entry exists; if one does, use it and skip the write. Copy the config file
before editing it, then tell the person the file and the undo line.

Then: "Reload <your agent> and sign in to ABN when it asks, then tell me to go
on." Start again at step 1 next turn — trust only what you can see then.

## 3. Sign in

- **Claude Code:** "Run `/mcp`, pick `abn`, choose Authenticate."
- **Codex:** `codex mcp login abn` (a browser page opens; tell them first).
- **Cursor:** "Settings > MCP, sign in to `abn`."
- **opencode:** `opencode mcp auth abn`, then open the printed URL for them.
- **Gemini CLI:** "Sign in to `abn` from the MCP settings."

If it fails with `Integration not found`, clear the credential and retry **once**
(`opencode mcp logout abn`, or the equivalent). Twice means it is ABN-side: stop
and say so.

## 4. The checks

Do all of these before saying anything:

- **Account.** `get_account` → identity, `kind` (prospect, client, partner,
  staff), `entitlements`, `quotas`. An `account_ambiguous` error means the login
  belongs to several accounts: ask which one, pass `accountId` on every call.
- **Catalog.** Compare `catalog_version` with the version in `services.json`.
  Older catalog than the skills → tell them to update the skills. Newer → fine.
- **Quotas.** Only mention a quota that is close to its limit.
- **Local.** Are all five skills present? `npx -y skills list --global` and
  grep for the five names. Missing ones → give the install command.

## 5. Say the result

**Everything fine, one line:**

> All set: ABN connected, catalog 0.1.0, signed in as <email>, 5 skills.

**Offline, also one line:**

> All set (offline): 5 skills, catalog 0.1.0; briefs go by email until the
> gateway is reachable.

**Something to do:** one line per problem, only the problems, each with its one
step. No table, no list of what passed.

| Problem | Step |
|---|---|
| Not signed in | The sign-in command for their agent (step 3) |
| Server added, no tools | Reload the agent; tool lists are fixed at session start |
| Skills missing | `npx -y skills add AbnAsia/abn-toolkit --skill '*' --agent <id> --yes --global` |
| Catalog older than the skills | Update the skills, then reload |
| `entitlement_missing` | Read the error's `next` field — it names the step |
| Gateway unreachable | Nothing to fix locally; briefs go by email |

Run again whenever they ask; every check is safe to repeat.
