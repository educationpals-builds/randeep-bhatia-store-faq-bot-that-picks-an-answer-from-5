# Store FAQ bot that picks an answer from the help center

**Skill type:** Audit advisor  
**Domain:** FAQ bot answer-matching for e-commerce help centers  
**Runtime:** Any assistant runtime that loads skill files

---

## Purpose

Audit whether a store FAQ bot correctly matches shopper questions to help-center answers—especially when product names appear in questions about different topics (refunds vs. shipping).

**Stakes:** Shoppers get the wrong policy and leave the cart

**Pass bar:** The answer matches the shopper's real ask — not a nearby FAQ about the same product

---

## Five-check walk

### Check 1: Unowned
**Score:** 4 (highest severity)  
**Question:** Does the bot latch onto a token (product name) without owning the shopper's actual intent?

**Measurement:** Count questions where the bot matched on product name but missed the action verb (return, cancel, refund). Flag if > 10% of sampled questions show product-name-only matching.

**Worked example:**  
Input: `refund for wrong size on the Trail Jacket, not a shipping question`  
Failure: Bot returns Trail Jacket shipping FAQ because "Trail Jacket" matched, ignoring "refund" and "not a shipping question."

---

### Check 2: Copies
**Score:** 2  
**Question:** Does the bot return duplicate or near-duplicate answers for distinct questions?

**Measurement:** Compute max cross-answer similarity (L2-normalized embeddings, dot product). Flag if any pair exceeds 0.92 similarity while serving different intent categories.

**Worked example:**  
Input: `how long do i have to return the Nova Buds after they ship`  
Check: Compare returned answer embedding against answers served for shipping-only questions. Flag if similarity > 0.92.

---

### Check 3: Room
**Score:** 1  
**Question:** Does the answer leave room for the shopper's specific situation, or does it force a generic response?

**Measurement:** Per-answer entropy versus uniform distribution over answer candidates. Flag if entropy < 0.3 (bot always picks same answer regardless of question variation).

**Worked example:**  
Input: `Nova Buds delivery says Friday — can i still cancel`  
Check: Does the bot distinguish "cancel before delivery" from "cancel after delivery"? Measure answer diversity across cancel-intent questions.

---

### Check 4: Stitch
**Score:** 2  
**Question:** Does the bot stitch together the right pieces when the question spans two topics?

**Measurement:** For multi-intent questions, measure mass-across-boundary—how much attention weight crosses from one intent token to another. Flag if < 15% of weight bridges the two intents.

**Worked example:**  
Input: `how long do i have to return the Nova Buds after they ship`  
Check: Question spans "return" (refund policy) and "after they ship" (delivery timing). Does the bot address both, or collapse to one?

---

### Check 5: Ablation
**Score:** 1  
**Question:** If you remove the product name, does the bot still route correctly?

**Measurement:** Ablation delta—run the question with product name zeroed out, compare answer. Flag if answer changes to a different policy category (refund → shipping).

**Worked example:**  
Input original: `refund for wrong size on the Trail Jacket, not a shipping question`  
Input ablated: `refund for wrong size on the [PRODUCT], not a shipping question`  
Check: If ablated version still returns refund policy, product-name dependency is low (good). If it flips to shipping, product-name latching is the failure mode.

---

## Severity story

**Deciding check:** Unowned (score 4)

The bot latches onto "Trail Jacket" and returns shipping information even when the shopper explicitly wrote "refund for wrong size on the Trail Jacket, not a shipping question." The shopper wanted refund policy. The bot ignored "refund" and "not a shipping question" because product-name matching fired first. The CX team gets a complaint; the shopper abandons the cart.

---

## Call

**Ship with conditions** — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."

---

## Tripwire

**Cadence:** Every week until launch, then monthly once it's live.  
**Drift threshold:** If the deciding check drifts more than one step from this ruling  
**Owners:** Rosalind and the document ops lead

---

## Output shape

When this skill runs, it returns:

```
SCORED FINDINGS
- Unowned: [score] — [finding]
- Copies: [score] — [finding]
- Room: [score] — [finding]
- Stitch: [score] — [finding]
- Ablation: [score] — [finding]

SEVERITY
[Deciding check name]: [severity story with specific failure]

CALL
[Ship / Ship with conditions / Hold] — [action and owner]

TRIPWIRE
[Cadence] | Drift threshold: [threshold] | Owners: [names]
```

---

## Sample asks

A stranger with their own FAQ bot can paste their situation:

> "Our product FAQ bot keeps answering warranty questions with setup instructions because both mention the model number. Here are three questions from this week's support queue:
> - how do I claim warranty on my SoundPro 500
> - SoundPro 500 won't pair, is that covered under warranty
> - return my SoundPro 500, it's defective"

The skill walks all five checks against their bot, scores each, identifies the deciding check, and returns findings with the measurement that would confirm each—plus a call and tripwire.

---

## Source

Specimen sentences from: Store help-desk chat logs

Usage reality: Short mobile questions with product names in the middle
