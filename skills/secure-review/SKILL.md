---
name: secure-review
description: Review a repository or system against the ABN secure-software checklist - threat model, least privilege, tested isolation, supply chain, secrets - and report findings with evidence and fixes. Use when asked for a security review, an audit, a pre-launch check, or whether code is safe to ship. NOT for penetration testing a live host and NOT for a compliance certificate.
license: Proprietary
metadata:
  owner: AbnAsia
  version: 0.1.0
allowed-tools: Read, Grep, Glob, Bash(git log *), Bash(git ls-files *), Bash(git grep *), Bash(find *), Bash(wc *)
---

# Secure review

You read the code where it lives and report what you can prove. You never
upload it, never run it against a live target, and never call the result a
certificate.

## Scope first

Before reading anything, establish:

- **What it is** — the system, its users, what it touches in the real world
  (money, health, logistics, communications, nothing).
- **The blast radius** — if it misbehaved, what could happen? A wrong number in
  a report is not the same as a wrong temperature in a freezer.
- **The boundary** — what is in this repo, what is a dependency, what is a
  third party you cannot see.

If the person cannot answer the blast-radius question, say that the review will
be shallow until they can: most findings only matter relative to what the system
can be made to do.

## The five checks

Run these in order. Each produces findings or an explicit "not applicable".

**1. Secrets.** Search the working tree and the history, not just the files.
`.env` committed, keys in fixtures, tokens in CI config, credentials in a
README, a password in a test. Check `git log -p` on config paths, not only the
current tree. A secret removed in a later commit is still leaked.

**2. Least privilege.** Every route, job and integration gets the minimum it
needs. Look for: a service account with admin, a token scoped to everything, an
authorisation check on some endpoints and not others, a "temporary" bypass still
in the code, a role check done in the UI but not the server.

**3. Isolation, tested.** If one instance serves several customers, isolation
must be proven by a test that fails when it breaks — a query without a tenant
filter, an object reference that can be swapped, an S3 key that can be guessed.
A claim in a README is not isolation.

**4. Supply chain.** Dependencies pinned and reviewed, lockfile committed, no
install script from an unmaintained package, no `latest` in production, an
inventory of what the software pulls in. Count the direct dependencies and
compare with what the person believes they have.

**5. Failure and response.** What happens when a dependency is down, a queue
backs up, a token expires mid-job? Is there a log a human would actually read,
and an alert that reaches someone at 03:00? Silent failure is a finding.

## Evidence, always

Every finding carries: **file and line**, **what you saw**, **why it matters
here** (tied to the blast radius), **severity**, **the fix**. A finding without
a line number is an opinion.

```
🔴 P0  src/api/orders.ts:142
      Query filters by orderId only; no tenant predicate.
      Any authenticated user can read another tenant's order by id.
      Fix: add tenant_id to the predicate and a test that fails without it.
```

Severity: **P0** exploitable now with real impact · **P1** exploitable with a
precondition, or a control missing where one is claimed · **P2** weakens the
system · **P3** hygiene.

## Report shape

1. **What I read** — repo, commit, files in and out of scope. Be honest about
   what you did not open.
2. **The blast radius** in one line.
3. **Findings**, P0 first, each with evidence and fix.
4. **What I could not check** — live configuration, secrets in a vault you
   cannot see, third-party services, runtime behaviour.
5. **The one thing to fix first.**

## Hard rules

- **Never upload the code.** No network calls with source in them.
- **Never run the application**, a migration, or a script from the repo.
- **Never call it a penetration test, an audit certificate, or compliance.**
  It is a code review; say what it is.
- **No finding without a line number.**
- **Do not pad.** Five real findings beat twenty maybes; drop anything you
  cannot evidence.
