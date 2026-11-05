---
name: publish-article
description: Publish an article to news.abnasia.org in every locale - write the English source, translate, validate, move, commit and deploy. Use when asked to publish, post or translate an article for ABN News. NOT for drafting marketing copy for other channels and NOT for editing the site's code.
license: Proprietary
metadata:
  owner: AbnAsia
  version: 0.1.0
allowed-tools: Read, Write, Edit, Bash(git *), Bash(ls *), Bash(wc *), Bash(mkdir *)
---

# Publish to ABN News

The site is a Next.js blog with contentlayer, one folder per locale, and a
`localeid` that ties the translations of one article together. Publishing is:
source in, nine locales out, validated, committed on `DEV`, merged to
`PRODUCTION`, deployed.

## The nine locales

| Locale | Folder | Language |
|---|---|---|
| `en` | `data/blog/posts` | English |
| `vi` | `data/blog/news` | Vietnamese |
| `fr` | `data/blog/articles` | French |
| `de` | `data/blog/germany` | German |
| `es` | `data/blog/es` | Spanish |
| `id` | `data/blog/indonesia` | Indonesian |
| `jp` | `data/blog/japan` | Japanese |
| `cn` | `data/blog/china` | Chinese |
| `th` | `data/blog/thailand` | Thai |

## Steps

1. **Write the English source** into `article.templates/article.maker.en.input.txt`
   using its field format: `[t]` title, `[d]` description, `[i]` images,
   `[pdf]`, `[v]`, then `[content] … [/content]`. Facts only, attributed, no
   adjectives the source did not use.
2. **Translate.** Prefer the repo's own script
   (`article.master.create.and.translate.py`). If its provider chain fails —
   invalid key, a hang, truncated bodies — write the translations yourself into
   `article.templates/` with the same frontmatter and the **same `localeid`**,
   then continue. Never ship a machine translation you have not length-checked.
3. **Validate every file** before moving it: frontmatter parses and carries
   `title`, `date`, `language`, `localeid`, `tags`, `draft: false`; the body is
   not a stub; the tail ends like prose, not mid-sentence; `contact@abnasia.org`
   survived. A truncated translation is skipped and reported, never published.
4. **Move** with `article.move.sh` so each locale lands in its folder.
5. **Commit on `DEV`** — one commit, the article's own message. Never commit
   `node_modules`, caches, rendered media or `__pycache__`.
6. **Publish**: PR from `DEV` to `PRODUCTION`, merge, then run the deploy on the
   server (`newsdeploy.sh`: lint, typecheck, build, container recreate).
7. **Verify live**: every locale URL returns 200, the title matches, and the
   `hreflang` alternates list all nine. Report the URLs, not a summary.

## Hard rules

- **One `localeid` for all nine.** It is how the language switcher finds them.
- **Never publish a half-written locale.** Skip and say so.
- **Never invent a fact, a number or a quote.** Attribute to the source in the
  body, not only in the frontmatter.
- **Never merge to `PRODUCTION` without a successful build** on the deploy host.
- **Verify, do not assume.** A 404 on one locale is a broken article.

## When the gateway is connected

`publish_article` on the `abn` server does steps 2, 3 and 7 for you and returns
`published`, `skipped` and `urls`. Use it for the translation and validation,
keep the git and deploy steps local, and report any locale it skipped.
