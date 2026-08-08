# Verification: Store FAQ bot that picks an answer from the help center

This document confirms that a stranger can run the audit tool against the seeded specimen and receive the deciding-check finding with a numeric measurement demand.

## Stranger verification steps

1. **Open the tool in /play mode**  
   Load the Store FAQ bot audit with the seeded specimen already embedded.

2. **Confirm the specimen is visible**  
   The tool should display the worked example:
   - **Specimen:** Store FAQ bot that picks an answer from the help center
   - **Stakes:** Shoppers get the wrong policy and leave the cart
   - **Standard:** The answer matches the shopper's real ask — not a nearby FAQ about the same product

3. **Run the five checks**  
   Walk through each check. The tool must surface findings for:
   - Unowned (rated 4)
   - Copies (rated 2)
   - Room (rated 1)
   - Stitch (rated 2)
   - Ablation (rated 1)

4. **Verify the deciding-check finding surfaces**  
   The tool must identify **unowned** as the deciding check (top crack) and present it prominently.

5. **Confirm numeric measurement is demanded**  
   For the unowned finding, the tool must ask for a specific numeric measurement — not a vague description. Example prompt the tool should issue:
   > "How many of the last 50 refund/cancel questions were routed to shipping answers instead?"

   A stranger cannot complete the audit without providing a number.

## Test inputs from the seeded specimen

Use these real lines from store help-desk chat logs to verify the tool handles the specimen domain:

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

The tool should recognize that these questions contain product names ("Nova Buds", "Trail Jacket") alongside intent words ("return", "cancel", "refund") and surface the unowned finding: no dedicated owner catches refund/cancel intent before product-name matching runs.

## Expected output confirmation

After a stranger completes the run, the tool must return:

- **Deciding check:** unowned
- **Call:** Ship with conditions — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."
- **Tripwire:** Re-run Every week until launch, then monthly once it's live. If the deciding check drifts more than one step from this ruling → Rosalind and the document ops lead.

If the tool does not surface the unowned finding or does not demand a numeric measurement for it, verification fails.
