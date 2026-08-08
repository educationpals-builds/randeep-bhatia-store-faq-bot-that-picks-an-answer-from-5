# Store FAQ Bot That Picks an Answer from the Help Center

## Blueprint: Conversational Auditor Spec

### The Problem

Shoppers ask about refunds, but the FAQ bot answers with shipping times because it latched onto the product name. Fix that before the busy sale week.

**Stakes:** Shoppers get the wrong policy and leave the cart

---

## Pass Bar

> The answer matches the shopper's real ask — not a nearby FAQ about the same product

---

## Usage Reality

Short mobile questions with product names in the middle

---

## Specimen Sentences (from Store help-desk chat logs)

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

---

## Five-Check Audit Framework

### Check Ratings

| Check | Score | Description |
|-------|-------|-------------|
| Unowned | 4 | Intent tokens (refund, cancel, return) have no dedicated handler before product-name matching fires |
| Copies | 2 | Multiple FAQ entries share product names, creating retrieval collisions |
| Room | 1 | Embedding space clusters policy topics too close to shipping topics |
| Stitch | 2 | Query rewriting merges intent words with product names incorrectly |
| Ablation | 1 | Removing product-name matching changes output for intent-heavy queries |

### Deciding Check: Unowned (Score 4)

The "unowned" check scores highest because intent-signaling words like "refund," "cancel," and "return" have no dedicated processing path. The FAQ bot's retrieval fires on product names first, so when a shopper types "refund for wrong size on the Trail Jacket, not a shipping question," the bot latches onto "Trail Jacket" and returns shipping FAQs instead of the refund policy.

---

## Worked Example: The Failure Story

**Input:** `refund for wrong size on the Trail Jacket, not a shipping question`

**What happens:**
1. Bot tokenizes query, sees "Trail Jacket"
2. Product-name matching fires first
3. Retrieves "Trail Jacket shipping times" FAQ
4. Shopper sees shipping info when they explicitly asked about refunds
5. Shopper abandons cart or contacts support

**Who acts on it:** CX team handles the escalation; shopper may churn.

---

## Ship Call

Ship with conditions — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."

---

## Tripwire

Re-run Every week until launch, then monthly once it's live. If the deciding check drifts more than one step from this ruling → Rosalind and the document ops lead.

---

## Auditor Conversation Flow

When a stranger brings their own FAQ bot mismatch scenario, the auditor:

1. **Collects the setup** — What bot, what it's supposed to do, what breaks when it fails
2. **Walks all five checks** — Unowned, Copies, Room, Stitch, Ablation
3. **Scores each check** — 1 (fine) to 4 (critical)
4. **Identifies the deciding check** — The highest-scoring failure mode
5. **Proposes measurements** — Specific metrics that would confirm each finding
6. **Returns the ruling** — Ship, hold, or ship-with-conditions
7. **Sets the tripwire** — Cadence, threshold, and owner who wakes when it drifts

---

## Output Shape

Every audit run produces:

- **Scored findings** — Each check with its rating and evidence
- **Severity story** — The specific failure with a real input, wrong output, and affected party
- **Call** — Ship / hold / ship-with-conditions, with checkable actions and owners
- **Tripwire** — Review cadence and escalation path if the deciding check drifts

---

## Acceptance Criteria

- [ ] Reads a stranger's FAQ bot mismatch with all five checks
- [ ] Every finding names the measurement that would confirm it
- [ ] Each run ends with a ruling, the standing rule it became, and an alarm with trigger and owner
- [ ] The Store FAQ bot audit (Nova Buds / Trail Jacket examples) is visible as the worked example
