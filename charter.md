# Store FAQ bot that picks an answer from the help center

## Audit Charter

This document records the full audit of the store FAQ bot — the system that picks an answer from the help center when shoppers ask questions. The audit determines whether the bot's checks actually split the work, or whether they collapse into a single pattern that misroutes queries.

---

## What breaks if the parts aren't really splitting the work

Shoppers get the wrong policy and leave the cart

---

## The pass bar

The answer matches the shopper's real ask — not a nearby FAQ about the same product

---

## Usage reality

Short mobile questions with product names in the middle

---

## Pasted inputs

Source: Store help-desk chat logs

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

---

## Check ratings

| Check | Score |
|-------|-------|
| Unowned | 4 |
| Copies | 2 |
| Room | 1 |
| Stitch | 2 |
| Ablation | 1 |

---

## Deciding check

**Unowned** — rated 4

The unowned check surfaces the core failure: when a shopper asks about refunds or cancellations, the bot latches onto the product name (Nova Buds, Trail Jacket) and returns shipping or delivery FAQs instead. No part of the current setup owns the distinction between "question about this product's return policy" and "question about this product's shipping status." The product name triggers shipping answers even when the shopper explicitly says "refund" or "return."

---

## Severity story

A shopper types "refund for wrong size on the Trail Jacket, not a shipping question" — explicitly stating they do not want shipping information. The bot sees "Trail Jacket" and returns the Trail Jacket shipping FAQ. The shopper, already frustrated by receiving the wrong size, now gets the wrong answer. They abandon the cart or escalate to a human agent during the busy sale week, when agent capacity is already stretched.

---

## The call

Ship with conditions — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."

---

## Pressure response

Hold my rating — I'll defend it

---

## The alarm

Re-run Every week until launch, then monthly once it's live. If the deciding check drifts more than one step from this ruling → Rosalind and the document ops lead.

---

## Summary

This audit found that the store FAQ bot's unowned check scores highest (4) — the bot has no dedicated handler for refund/cancel intent before product-name matching fires. The fix: Priya's word watch for refund/cancel terms runs first, tested against the explicit failure case. Weekly re-runs until launch, monthly after, with escalation to Rosalind and the document ops lead if the unowned check drifts.
