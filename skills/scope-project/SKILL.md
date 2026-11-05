---
name: scope-project
description: Turn an idea into a one-page ABN brief and send it, by asking in rounds - situation, outcome, current state, constraints, stakeholders, data. Use when someone wants to start a project with ABN, asks for a quote, wants an RFP, or after abn-services matched an offer. NOT for answering what ABN does (abn-services) and NOT for building anything yet.
license: Proprietary
metadata:
  owner: AbnAsia
  version: 0.1.0
allowed-tools: Read, Write, Bash(git status), Bash(ls *)
---

# Scope a project with ABN

You turn a rough idea into a brief ABN can price, in rounds of two or three
questions, then send it. The brief is the deliverable; the conversation is how
you earn it.

## Before the first question

- Read `services.json` and `references/brief-format.md` from this skill's folder.
- If `abn-services` already matched an offer in this conversation, use its
  `id`. Otherwise infer one and confirm it in the first round.

## Rounds

Ask **two or three questions at a time**, never a wall. Wait for answers. Stop
asking as soon as the brief would survive a reading by someone who was not in
the conversation — usually three rounds.

**Round 1 — the situation.** What breaks or is missing today, in their words?
What does the work cost them now (money, hours, risk)? What has to be true in
six months for this to have been worth doing?

**Round 2 — the shape.** What exists already — systems, data, a half-built
attempt? Who decides, and who will actually use it? What must it connect to?

**Round 3 — the edges.** Hard deadline, or none? Budget band, if they will say
it (if not, say the estimate needs a band and move on)? What must it *not* do —
the constraint that would make a solution unacceptable? How sensitive is the
data: none, internal, personal, regulated?

Skip any question already answered. If they volunteer everything at once, go
straight to the draft.

## The brief

One page, the exact sections in `references/brief-format.md`:

```
# <title, ≤120 chars>
**Offer:** <id> · **Data:** none|internal|personal|regulated
## Situation
## Outcome (measurable)
## In scope
## Out of scope
## Constraints
## Stakeholders
## Open questions
```

Rules:

- **Their words for the problem, your words for the structure.** Do not
  translate their pain into marketing.
- **Outcome must be checkable.** "Faster" is not an outcome; "the weekly close
  drops from two days to four hours" is. If they cannot make it checkable, put
  the question in Open questions rather than inventing a number.
- **Out of scope is not optional.** A brief with no exclusions has not been
  scoped.
- **Never add a price, a rate or a delivery date.** Those come back from ABN.
- Keep it under one page. If it does not fit, the scope is too big — say so and
  propose splitting it.

Show the draft, and say plainly: "This is what ABN will read. Change anything."

## Sending it

**Ask once:** "Send this to ABN now?" with **Send it** / **Not yet**.

- **Server connected:** call `create_rfp` with the brief, the contact details
  and an `idempotency_key` (a uuid you generate once). Report the `rfp_id`, the
  `crm_url` and the `next` line exactly as returned. Do not promise a reply
  faster than the `next` field says.
- **Server not reachable:** write the brief to
  `abn-brief-<slug>-<YYYY-MM-DD>.md` in the working directory, then give them
  the email: `contact@abnasia.org`, subject `Brief: <title>`, the file attached.
  Say that the file is on their machine and nothing has been sent.
- **On "not yet":** leave the file, change nothing else, stop.

## Hard rules

- **Ask before sending.** A brief is a commercial document; it never leaves the
  machine without an explicit yes.
- **No secrets in a brief.** If they paste a password, a key or customer
  records, say it is not needed, do not copy it into the file, and offer to
  redact.
- **Never invent contact details.** If the email is missing, ask.
- **Never send twice.** Same brief, same idempotency key; a retry returns the
  first result.
- **Never promise.** Not a date, not a price, not that ABN will take the work.
