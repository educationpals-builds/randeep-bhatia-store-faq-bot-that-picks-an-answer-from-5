# Method: The PRISM Framework for Splitting Work

When a setup like the Store FAQ bot that picks an answer from the help center fails to route questions correctly, the root cause is often that the work isn't actually split—it's collapsed. This method walks five checks to audit whether your setup's parts are genuinely dividing labor or just pretending to.

---

## The Five Checks

### P — Partition the Space

Does each component own a distinct region of the input space, or do they overlap and fight?

For the FAQ bot: Does the refund-handling path own refund questions exclusively, or does product-name matching grab them first?

### R — Run in Parallel

Can the parts operate independently, or does one block another?

For the FAQ bot: Can intent detection and product lookup run at the same time, or does product-name matching preempt intent classification?

### I — Individuate the Pattern

Does each part recognize its own pattern without bleeding into a neighbor's territory?

For the FAQ bot: When a shopper asks "refund for wrong size on the Trail Jacket," does the refund detector see "refund" before the product matcher latches onto "Trail Jacket"?

### S — Stitch the Spectra

When parts produce outputs, do they combine cleanly or collide?

For the FAQ bot: If both product info and policy info are retrieved, which one wins? Is there a clear merge rule?

### M — Map What Each Head Sees

Can you trace exactly what input each component actually processed?

For the FAQ bot: When shipping times appear instead of refund policy, can you see that the product matcher saw "Nova Buds" and ignored "return"?

---

## The Anti-Pattern: Collapse to Monochrome

When checks fail, the setup has collapsed to monochrome—one signal dominates and drowns out the rest.

**Symptoms of collapse:**
- One matcher (like product-name lookup) fires on every query
- Intent signals get overwritten by entity signals
- The "winning" path has no gate that checks whether it should have won

**In the FAQ bot case:** The product name "Nova Buds" or "Trail Jacket" triggers product-info retrieval even when the shopper's actual ask is about returns, cancellations, or refunds. The refund/cancel intent is never evaluated because the product matcher already claimed the query.

---

## Using the Checks

Score each check 1–5:
- **1** = The part clearly owns its space, no overlap
- **5** = Total collapse; one signal dominates everything

The check with the highest score is your deciding check—the crack that needs fixing first.

When the deciding check is **unowned** (no part owns the refund/cancel space), the fix is to add a dedicated owner for that space before other matchers run.
