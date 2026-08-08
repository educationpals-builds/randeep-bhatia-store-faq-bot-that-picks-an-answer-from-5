# Store FAQ bot that picks an answer from the help center

An audit tool for store FAQ bots that confuse product-name matches with actual shopper intent—like answering a refund question with shipping times because both mention "Nova Buds."

## Verdict

**Ship with conditions** — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."

## Tripwire

Re-run Every week until launch, then monthly once it's live. If the deciding check drifts more than one step from this ruling → Rosalind and the document ops lead.

## The problem

Shoppers ask about refunds, but the FAQ bot answers with shipping times because it latched onto the product name. Fix that before the busy sale week.

**What breaks:** Shoppers get the wrong policy and leave the cart

**Pass bar:** The answer matches the shopper's real ask — not a nearby FAQ about the same product

## Worked example

These lines came from store help-desk chat logs:

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

**Usage reality:** Short mobile questions with product names in the middle

**Deciding check:** unowned (scored 4/5)

The "unowned" check surfaced the core failure: no part of the bot owns intent classification before product-name matching fires. When a shopper types "refund for wrong size on the Trail Jacket," the bot sees "Trail Jacket" and pulls shipping FAQs—nobody claimed the refund signal first.

## One-paste rebuild

Point this audit at your own FAQ bot setup:

1. Paste your failing inputs (the questions your bot misroutes)
2. Name where those lines came from and when
3. State your pass bar—what "correct" looks like
4. Walk the five checks and score each
5. Identify which check decides your call
6. Make the call: ship, hold, or ship with conditions
7. Set your tripwire: cadence, owner, drift threshold

The audit returns a scored finding for each check, a severity story grounded in your real inputs, and a standing rule with an alarm.

## Files

- `charter.md` — Full audit with findings, severity story, and the ship call
- `METHOD.md` — The five-check framework (PRISM)
- `VERIFY.md` — Stranger verification steps

<!-- educationpals-build-verified -->
