# Brief format

One page. Every section is required except Open questions. No prices, no dates.

```markdown
# <Title: what is being built or fixed, ≤120 characters>

**Offer:** <offer id from services.json> · **Data:** none | internal | personal | regulated

## Situation

Two to four sentences, the client's own words for what breaks today. What it
costs them now — hours, money, risk, or a person doing something by hand that
should not be done by hand.

## Outcome

What has to be true when this works. Measurable where it can be: a number, a
count, a time, a state that can be checked. Where it cannot, say what would
count as proof and put the open question below.

- <outcome 1, checkable>
- <outcome 2, checkable>

## In scope

- <what ABN would deliver>
- <what ABN would deliver>

## Out of scope

- <what is explicitly not included, and why>
- <what is someone else's job>

## Constraints

- **Deadline:** <date or "none stated">
- **Budget:** <band if given, else "not stated — estimate needs a band">
- **Must use:** <existing systems, vendors, standards>
- **Must not:** <the constraint that would make a solution unacceptable>
- **Existing systems:** <what it has to connect to or replace>

## Stakeholders

| Role | Who | Decides what |
|---|---|---|
| Sponsor | <role> | budget, scope |
| User | <role> | adoption |
| Technical | <role> | architecture sign-off |

## Open questions

- <the thing nobody could answer yet>
```

## What makes a brief good

- A stranger can read it and know what to build and what not to.
- The outcome section has something checkable in it.
- Out of scope is not empty.
- It fits on one page. If it does not, the scope is two projects.

## What makes a brief bad

- Adjectives instead of numbers: "modern", "seamless", "world-class".
- A price or a delivery date written by the agent.
- No exclusions, so every argument later is a surprise.
- A solution described before the problem is.
