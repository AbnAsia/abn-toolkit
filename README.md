# ABN Asia — agent toolkit

Skills and a server that let an AI agent work with ABN Asia: match an idea to
what ABN actually does, turn it into a one-page brief, send it, review code
against ABN's security checklist, and publish to ABN News.

## Use it

Paste this into your agent:

> Set up ABN Asia for me by following INSTALL_FOR_AGENTS.md in
> https://github.com/AbnAsia/abn-toolkit

It installs five skills, and connects ABN's server if it is reachable. Then:

- **"What does ABN do?"** → the catalog, matched to your situation
- **"Scope a project with ABN"** → three rounds of questions, a one-page brief,
  and it asks before sending
- **"Review this repo for security"** → findings with file and line, and a fix
  for each
- **"Publish this to ABN News"** → nine locales, validated, deployed

## What is in here

| Path | What |
|---|---|
| `INSTALL_FOR_AGENTS.md` | The contract the agent follows. Start here. |
| `services.json` | The catalog: offers, who they are for, how engagements work. |
| `skills/abn-services` | Match a situation to an offer. Never quotes a price. |
| `skills/scope-project` | Idea → brief → submitted or emailed. |
| `skills/secure-review` | The ABN security checklist, run locally. |
| `skills/publish-article` | The news.abnasia.org pipeline, nine locales. |
| `skills/abn-setup` | Connects, checks, fixes, reports one line. |
| `docs/gateway-tools.md` | The server's tool contract (for ABN staff and integrators). |

## Principles

- **No price without scoping.** The catalog carries none, and the skills are
  forbidden to invent one.
- **Nothing leaves your machine** unless you ask for it. `secure-review` uploads
  no code. A brief is sent only after you say yes.
- **Every install is reversible.** The contract names every file it writes and
  the line that undoes it.

## For ABN staff

`docs/gateway-tools.md` is the agreement between these skills and the gateway at
`mcp.abnasia.org`. A skill that references a tool or field not in that file is a
bug in the skill. The gateway itself lives in the private
`abn.platform.gateway` repository.

Contact: contact@abnasia.org
