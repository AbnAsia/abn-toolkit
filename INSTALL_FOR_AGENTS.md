# Set up ABN Asia (instructions for agents)

A person asked you to set up ABN Asia: the skills and the server that let them
find the right ABN service, turn an idea into a scoped brief, and send it to
ABN without leaving the conversation. Complete every step yourself with your own
tools. A step marked **Ask the person** needs them — a browser sign-in, an API
key, a restart. Ask in one short line, wait until they say it is done, then go
on. Tell them what to do and what they will see; do not explain the internals.

What gets installed:

- **Five skills.** `abn-services` (what ABN does, and which offer fits an idea),
  `scope-project` (idea → one-page brief → submitted to ABN), `secure-review`
  (the ABN security checklist run against a repo or system), `publish-article`
  (the news.abnasia.org multi-language pipeline) and `abn-setup` (checks
  everything and fixes what it can).
- **One server, `abn`.** At `https://mcp.abnasia.org/mcp` it reads the live
  catalog, files a brief as a CRM lead, reports project status, publishes
  articles and mints product API keys. `abn-services` and `scope-project` work
  without it — the server is what submits.

## 0. Before you start

**Repository.** Everything installs from the public repo `AbnAsia/abn-toolkit`;
no GitHub account is needed. Check this machine can reach it:

```
git ls-remote https://github.com/AbnAsia/abn-toolkit.git HEAD
```

If that fails, stop and tell the person: "This machine can't reach
github.com/AbnAsia/abn-toolkit. Check the network or proxy, then ask me again."

**Gateway.** Check the server answers:

```
curl -sS -m 10 -o /dev/null -w '%{http_code}\n' https://mcp.abnasia.org/healthz
```

`200` means the server is up. Anything else — no DNS, a timeout, `000` — means
it is not live yet on this network. **Do not stop.** The skills work offline:
install them (step 2), skip steps 3 and 4, and tell the person in one line that
briefs will be sent by email until the gateway is reachable. Everything else in
this file still applies.

**Already installed?** Skip to [step 5](#5-verify) only when all five skills are
listed (`abn-services`, `scope-project`, `secure-review`, `publish-article`,
`abn-setup`) **and** you can list an `abn` tool and `get_services` answers.
Servers added by hand without the skills still need step 2.

## 1. Which agent are you?

Run this. It reads only environment variables, writes nothing, and prints the
agent plus the config file it owns:

```
bash <<'EOF'
v() { local n="$1"; printf '%s' "${!n-}"; }
agent=unknown
if [ -n "$(v CODEX_THREAD_ID)$(v CODEX_CI)" ]; then agent=codex
elif [ -n "$(v CURSOR_AGENT)" ]; then agent=cursor
elif [ -n "$(v GEMINI_CLI)" ]; then agent=gemini-cli
elif [ -n "$(v OPENCODE)" ]; then agent=opencode
elif [ -n "$(v CLAUDE_CODE_CHILD_SESSION)$(v CLAUDECODE)" ]; then agent=claude-code
fi
case "$agent" in
  claude-code*) cfg="$HOME/.claude.json" ;;
  codex)        cfg="$HOME/.codex/config.toml" ;;
  cursor*)      cfg="$HOME/.cursor/mcp.json" ;;
  gemini-cli)   cfg="$HOME/.gemini/settings.json" ;;
  opencode)     cfg="$HOME/.config/opencode/opencode.json" ;;
  *)            cfg="" ;;
esac
printf '%s\n' "$agent"
[ -n "$cfg" ] && printf 'config: %s\n' "$cfg"
EOF
```

| It prints | You are | Install skills as |
|---|---|---|
| `claude-code…` | Claude Code | `claude-code` |
| `codex` | Codex | `codex` |
| `cursor…` | Cursor | `cursor` |
| `gemini-cli` | Gemini CLI | `gemini-cli` |
| `opencode` | opencode | `opencode` |
| `unknown` | judge for yourself: Windsurf `windsurf`, Cline `cline`, Copilot `github-copilot`, Antigravity `antigravity-cli` |

## 2. Install the skills

Needs Node.js 22.20 or newer (`node --version`). Use the id from the table:

```
npx -y skills add AbnAsia/abn-toolkit --skill '*' --agent <agent-id> --yes --global
```

Check it worked:

```
npx -y skills list --global | grep -E 'abn-services|scope-project|secure-review|publish-article|abn-setup'
```

Five names must appear. This route installs no server — step 3 does that.

## 3. Connect the ABN gateway

Only when step 0 found the gateway up.

**Ask the person:** "Connect ABN's server to <your agent>? It reads the live
service catalog and sends a brief to ABN. You sign in on one.abnasia.org
afterwards." Options: **Yes, connect it** / **Not now**.

On no: change nothing. Say that the skills still work and briefs will be sent
by email, and stop here.

On yes: add a remote MCP server named `abn` at `https://mcp.abnasia.org/mcp`.
First check no `abn` entry exists already; if one does, use it and skip the
write. Then tell the person which file changed and the undo line.

| Agent | Command | File | Undo |
|---|---|---|---|
| Claude Code | `claude mcp add --transport http abn https://mcp.abnasia.org/mcp --scope user` | `~/.claude.json` | `claude mcp remove abn --scope user` |
| Codex | `codex mcp add abn --url https://mcp.abnasia.org/mcp` | `~/.codex/config.toml` | `codex mcp remove abn` |
| Cursor | add `"abn": { "url": "https://mcp.abnasia.org/mcp" }` under `mcpServers` in the file above | `~/.cursor/mcp.json` | remove that entry |
| opencode | add `"abn": { "type": "remote", "url": "https://mcp.abnasia.org/mcp", "enabled": true }` under `mcp` | `~/.config/opencode/opencode.json` | remove that entry |
| Gemini CLI | add `"abn": { "httpUrl": "https://mcp.abnasia.org/mcp" }` under `mcpServers` | `~/.gemini/settings.json` | remove that entry |

**Before editing any JSON file, copy it** (`cp file file.bak-$(date +%s)`) and
tell the person the backup path. Keep every other entry. Leave valid JSON.

## 4. Sign in

The gateway signs in through OneID. Prospects may create an account there; an
unknown email is a new account, not an error.

- **Claude Code:** **Ask the person:** "Run `/mcp`, pick `abn`, choose
  Authenticate. Tell me when you're done."
- **Codex:** **Ask the person:** "A sign-in page will open in your browser. Sign
  in or create an account there, then tell me." Then run
  `codex mcp login abn`.
- **Cursor:** **Ask the person:** "Open Settings > MCP and sign in to `abn`."
- **opencode:** run `opencode mcp auth abn`. It prints a URL — open it for them
  and say: "Sign in on the page that just opened."
- **Gemini CLI:** **Ask the person:** "Sign in to `abn` from the MCP settings."

If the agent's sign-in flow fails with `Integration not found`, clear the stored
credential and retry **once**: `opencode mcp logout abn` (or the equivalent),
then repeat the command above. If it fails twice, stop and tell the person it is
an ABN-side problem, not their machine.

**API key instead of a browser.** If the person prefers a key, they create one
at `https://one.abnasia.org/abn-gateway` and give it to you; add it as a header
on the same entry — `"headers": { "X-Api-Key": "<key>" }` — never paste it into
chat, never echo it, and say where you wrote it.

## 5. Verify

Run the setup skill: `/abn:abn-setup` in Claude Code with the plugin, otherwise
ask yourself to "set up ABN", which runs `abn-setup`.

It checks the server, the sign-in, the account, the catalog version, quotas and
updates, fixes what it can, and ends in **one line**:

- `All set: ABN connected, catalog 0.1.0, 5 skills.` → done.
- `All set (offline): 5 skills, catalog 0.1.0; briefs go by email.` → the
  gateway is unreachable; this is a working state, not a failure.
- Anything else is one line per problem, each with the one step to take.

Without the setup skill, call `get_services` (free, no auth). A catalog version
in the answer means the server is reachable.

## 6. Finish

Tell the person, in one or two lines:

> ABN is ready. Say "find me an ABN service for …" and I'll match your idea to
> what they actually do — or "scope a project with ABN" and you'll get a
> one-page brief I can send them.

Then stop. Do not start a scoping conversation until they ask.

## What this writes, and what it never does

Writes, and only these:

- five skill folders under your agent's skills directory
- one server entry in the config file named in step 3 (backed up first)

Never:

- runs a remote script through a shell (`curl … | sh` is not used anywhere here)
- sends the person's code, files or data anywhere — the only outbound calls are
  the repository check, the health check, and the tools the person asks for
- asks for a password, and never puts a key in a log, a command line or chat
- changes a permission prompt, disables a sandbox, or widens access to make
  something work

## Data boundary

May leave the machine when the person asks for it: the brief they wrote, their
contact details, the catalog query.

Must never leave the machine: source code, credentials, customer records,
database contents, or anything the person marked confidential. `secure-review`
reads code locally and reports findings; it uploads nothing.

## If something goes wrong

| What you see | Do |
|---|---|
| `git ls-remote` fails | No GitHub access. Check the network or proxy; nothing was installed. |
| Gateway health check fails | Not live on this network. Install the skills, skip 3-4, briefs go by email. |
| `npx skills` says Node is too old | Install Node 22.20+ and retry, or use a plugin route if the agent has one. |
| Skills listed, no `abn` tools | The agent has not reloaded: restart it, or reload config. Then re-check. |
| `auth_required` on a tool | Sign in again as in step 4. |
| `Integration not found` twice | ABN-side. Stop; tell the person, do not loop. |
| `entitlement_missing` | The account cannot use that tool. Read the `next` field — it names the step. |
| Tools work in a new session but not this one | Expected: tool lists are fixed when a session starts. |
| Catalog version older than the skills expect | Run `abn-setup`; it names the update command. |

## Remove ABN

```
npx -y skills remove abn-services scope-project secure-review publish-article abn-setup --global --yes
```

Then remove the `abn` server entry from the config file in step 3 — or restore
the backup that step made. For Claude Code: `claude mcp remove abn --scope user`.
