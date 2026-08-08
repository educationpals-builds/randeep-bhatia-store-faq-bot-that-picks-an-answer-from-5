# Store FAQ Bot Audit Measurements

Per-check measurements for auditing whether the FAQ bot correctly matches shopper questions to help center answers — not nearby FAQs about the same product.

---

## 1. Unowned Mass (Check: Unowned)

**What it measures:** How much of the shopper's question the bot ignores when selecting an answer.

**How to compute:**
1. Tokenize the shopper question (e.g., "refund for wrong size on the Trail Jacket, not a shipping question")
2. Identify which tokens the bot's retrieval actually weighted when selecting the FAQ
3. Calculate: `unowned_mass = (tokens_ignored / total_tokens) × 100`

**Threshold:** Flag if unowned_mass > 30%

**Example from specimen:**
- Input: "refund for wrong size on the Trail Jacket, not a shipping question"
- If bot latches onto "Trail Jacket" and ignores "refund," "wrong size," and "not a shipping question" → unowned_mass ≈ 70%
- **Flags: YES**

---

## 2. Cross-FAQ Similarity (Check: Copies)

**What it measures:** Whether the bot is confusing two different FAQ topics because they share product names.

**How to compute:**
1. For each candidate FAQ answer, extract the retrieval embedding
2. L2-normalize each embedding (flatten if multi-dimensional)
3. Stack normalized embeddings into matrix A
4. Compute similarity matrix: `S = A @ A.T`
5. Read off-diagonal values: `max_cross_similarity = max(S[i,j]) for i ≠ j`

**Threshold:** Flag if max_cross_similarity > 0.85

**Example from specimen:**
- "Nova Buds return policy" FAQ vs. "Nova Buds shipping times" FAQ
- If both embeddings are nearly identical because "Nova Buds" dominates → similarity ≈ 0.92
- **Flags: YES** (bot can't distinguish refund from shipping for same product)

---

## 3. Intent Entropy (Check: Room)

**What it measures:** Whether the bot's intent classification is spread too thin across multiple categories or concentrated on the correct one.

**How to compute:**
1. Get the bot's intent probability distribution for the question
2. Calculate entropy: `H = -Σ p(i) × log₂(p(i))`
3. Calculate uniform entropy for same number of intents: `H_uniform = log₂(n_intents)`
4. Compute ratio: `entropy_ratio = H / H_uniform`

**Threshold:** Flag if entropy_ratio > 0.7 (too uncertain)

**Example from specimen:**
- Input: "Nova Buds delivery says Friday — can i still cancel"
- If bot assigns: shipping=0.45, cancellation=0.35, returns=0.20 → entropy_ratio ≈ 0.82
- **Flags: YES** (bot isn't confident which intent applies)

---

## 4. Stitch Coherence (Check: Stitch)

**What it measures:** Whether the bot's answer actually addresses the question type, not just the product mentioned.

**How to compute:**
1. Extract the question type from the shopper query (return, cancel, refund, shipping, etc.)
2. Extract the answer type from the selected FAQ
3. Calculate: `stitch_match = 1 if question_type == answer_type else 0`

**Threshold:** Flag if stitch_match = 0

**Example from specimen:**
- Input: "how long do i have to return the Nova Buds after they ship"
- Question type: return
- If bot selects FAQ about shipping times → answer type: shipping
- **Flags: YES** (return ≠ shipping)

---

## 5. Ablation Delta (Check: Ablation)

**What it measures:** What happens when you remove the product name from the query — does the bot still pick the right FAQ?

**How to compute:**
1. Run original query through bot, record selected FAQ
2. Remove product name from query (e.g., "how long do i have to return the [PRODUCT] after they ship")
3. Run ablated query through bot, record selected FAQ
4. Calculate: `ablation_delta = 1 if original_faq ≠ ablated_faq else 0`

**Threshold:** Flag if ablation_delta = 1 AND ablated answer is more accurate

**Example from specimen:**
- Original: "refund for wrong size on the Trail Jacket, not a shipping question" → bot picks shipping FAQ
- Ablated: "refund for wrong size on the [PRODUCT], not a shipping question" → bot picks refund FAQ
- **Flags: YES** (product name was causing the mismatch)

---

## Summary Table

| Check | Measurement | Compute | Threshold |
|-------|-------------|---------|-----------|
| Unowned | unowned_mass | ignored tokens / total tokens | > 30% |
| Copies | max_cross_similarity | L2-norm, stack, matmul transpose, read off-diagonal | > 0.85 |
| Room | entropy_ratio | intent entropy / uniform entropy | > 0.7 |
| Stitch | stitch_match | question_type vs answer_type | = 0 |
| Ablation | ablation_delta | original vs product-name-removed result | = 1 (and ablated is better) |

---

## Pass Bar

The answer matches the shopper's real ask — not a nearby FAQ about the same product

Any measurement crossing its threshold contributes to the audit finding. The deciding check for this bot is **Unowned** (rated 4) — the bot ignores the actual question words when a product name is present.
