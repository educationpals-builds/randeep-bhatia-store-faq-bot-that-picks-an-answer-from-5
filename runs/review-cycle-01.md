# Review Cycle 01 — Store FAQ bot that picks an answer from the help center

**Cycle date:** Reference cycle  
**Reviewer:** Rosalind + document ops lead  
**Source:** Store help-desk chat logs  
**Cadence:** Every week until launch, then monthly once it's live

---

## Packet under review

Three shopper questions pulled from Store help-desk chat logs:

| Line # | Shopper question |
|--------|------------------|
| 1 | how long do i have to return the Nova Buds after they ship |
| 2 | Nova Buds delivery says Friday — can i still cancel |
| 3 | refund for wrong size on the Trail Jacket, not a shipping question |

**Usage reality:** Short mobile questions with product names in the middle

**Pass bar committed:** The answer matches the shopper's real ask — not a nearby FAQ about the same product

---

## Per-check measurements produced

### Check 1: Unowned (Score: 4)

**Measurement:** Mass-across-boundary  
**Threshold:** ≤ 0.15 of total attention mass should cross from question-intent tokens to product-name tokens  
**Observed:** 0.41 — FAQ bot attention mass flows heavily from "return" and "refund" tokens toward "Nova Buds" and "Trail Jacket" tokens  
**Result:** FAIL — product-name tokens capture intent that belongs to action words

### Check 2: Copies (Score: 2)

**Measurement:** Max cross-head similarity (L2-normalized flattened per-head maps, stack, matmul against transpose, read off-diagonal)  
**Threshold:** Off-diagonal similarity ≤ 0.70  
**Observed:** 0.58 — moderate redundancy between heads attending to product names  
**Result:** PASS — heads are not fully duplicating each other

### Check 3: Room (Score: 1)

**Measurement:** Per-head entropy versus uniform distribution  
**Threshold:** Entropy ratio ≥ 0.60 of uniform  
**Observed:** 0.72 — attention is reasonably distributed  
**Result:** PASS — heads have room to specialize

### Check 4: Stitch (Score: 2)

**Measurement:** Coherence of attention flow from question tokens to answer selection  
**Threshold:** Stitch continuity ≥ 0.65  
**Observed:** 0.54 — some fragmentation when product name appears mid-question  
**Result:** MARGINAL — attention path breaks when product name interrupts intent

### Check 5: Ablation (Score: 1)

**Measurement:** Ablation delta (zero one head before concat, measure output change)  
**Threshold:** Delta ≤ 0.25 for any single head  
**Observed:** 0.19 — no single head dominates the answer selection  
**Result:** PASS — removing one head does not collapse the system

---

## Caught-or-missed summary

| Line # | Shopper question | Bot behavior | Caught by check |
|--------|------------------|--------------|-----------------|
| 1 | how long do i have to return the Nova Buds after they ship | Returned shipping ETA for Nova Buds instead of return policy | **Caught: Unowned** — "Nova Buds" captured attention from "return" |
| 2 | Nova Buds delivery says Friday — can i still cancel | Returned delivery tracking info instead of cancellation policy | **Caught: Unowned** — "Nova Buds" + "Friday" pulled attention from "cancel" |
| 3 | refund for wrong size on the Trail Jacket, not a shipping question | Returned Trail Jacket shipping FAQ despite explicit "not a shipping question" | **Caught: Unowned** — "Trail Jacket" overrode "refund" and ignored negation |

**Lines caught:** 3 of 3  
**Deciding check:** Unowned (score 4) — product-name tokens consistently steal attention from action-intent tokens

---

## Severity story

Line 3 shows the failure at its worst: the shopper explicitly wrote "refund for wrong size on the Trail Jacket, not a shipping question" — and the bot still returned shipping information. The product name "Trail Jacket" captured so much attention mass that even the explicit negation ("not a shipping question") was ignored. This shopper gets the wrong policy and leaves the cart.

---

## Call

Ship with conditions — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."

---

## Tripwire

Re-run Every week until launch, then monthly once it's live. If the deciding check drifts more than one step from this ruling → Rosalind and the document ops lead.

---

## Time cost

| Phase | Duration |
|-------|----------|
| Packet assembly | 5 min |
| Five-check measurement run | 12 min |
| Per-line caught/missed annotation | 8 min |
| Severity story + call | 5 min |
| **Total cycle time** | **30 min** |

---

## Verdict

**Hold my rating — I'll defend it**

The unowned check at score 4 is the crack. Until Priya's refund/cancel word watch is in place and tested, the FAQ bot will keep latching onto product names and returning shipping answers when shoppers ask about refunds or cancellations.
