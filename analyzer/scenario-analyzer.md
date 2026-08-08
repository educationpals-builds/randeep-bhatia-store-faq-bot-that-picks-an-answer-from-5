# Store FAQ Bot Answer-Match Analyzer

Machine-readable analyzer for auditing whether the store FAQ bot picks answers that match the shopper's real ask.

## Specimen Under Analysis

**System:** Store FAQ bot that picks an answer from the help center  
**Failure mode:** Shoppers ask about refunds, but the FAQ bot answers with shipping times because it latched onto the product name  
**Stakes:** Shoppers get the wrong policy and leave the cart  
**Input reality:** Short mobile questions with product names in the middle  
**Source:** Store help-desk chat logs

---

## Analyzer Configuration

```yaml
analyzer:
  name: faq-bot-answer-match-analyzer
  version: 1.0.0
  domain: store-help-center-faq
  
  standard_line: "The answer matches the shopper's real ask — not a nearby FAQ about the same product"
  
  specimen_sentences:
    - "how long do i have to return the Nova Buds after they ship"
    - "Nova Buds delivery says Friday — can i still cancel"
    - "refund for wrong size on the Trail Jacket, not a shipping question"
  
  source: "Store help-desk chat logs"
```

---

## Check Findings (Scored)

```yaml
check_findings:
  unowned:
    score: 4
    status: critical
    description: "No component owns intent classification before product-name matching fires"
    
  copies:
    score: 2
    status: moderate
    description: "Product-name matching duplicates across FAQ retrieval and answer selection"
    
  room:
    score: 1
    status: low
    description: "Minimal headroom — retrieval saturates on product name alone"
    
  stitch:
    score: 2
    status: moderate
    description: "Intent signal lost when product-name match stitches to answer"
    
  ablation:
    score: 1
    status: low
    description: "Removing product-name priority shifts answer selection toward intent"
```

---

## Top Crack Identified

```yaml
top_crack:
  check: unowned
  score: 4
  severity: critical
  
  failure_story: |
    Shopper types "refund for wrong size on the Trail Jacket, not a shipping question"
    Bot latches onto "Trail Jacket" and returns shipping FAQ
    No component owned the refund/cancel intent before product-name matching ran
    Shopper sees wrong policy, abandons cart
```

---

## Analyzer Triggers

```yaml
triggers:
  change_trigger:
    - "FAQ content update in help center"
    - "Product catalog change affecting FAQ mapping"
    - "Bot retrieval logic modification"
    
  calendar_floor:
    pre_launch: "weekly"
    post_launch: "monthly"
    
  drift_threshold: 1
  drift_action: "Alert Rosalind and the document ops lead"
```

---

## Sampling Rule

```yaml
sampling:
  source: "Store help-desk chat logs"
  method: "Random sample from questions containing product names"
  minimum_per_cycle: 5
  must_include:
    - "At least one refund/return question"
    - "At least one cancel question"
    - "At least one question with explicit intent clarification"
```

---

## Ship Call

```yaml
ship_call:
  decision: "Ship with conditions"
  condition: "CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against \"refund for wrong size on the Trail Jacket.\""
  owner: "Priya (CX lead)"
```

---

## Tripwire Configuration

```yaml
tripwire:
  cadence:
    pre_launch: "Every week until launch"
    post_launch: "monthly once it's live"
  
  trigger: "If the deciding check drifts more than one step from this ruling"
  
  alert_recipients:
    - "Rosalind"
    - "document ops lead"
```

---

## Analyzer Output Schema

```yaml
output:
  per_check_scores:
    type: object
    properties:
      unowned: { type: integer, min: 1, max: 5 }
      copies: { type: integer, min: 1, max: 5 }
      room: { type: integer, min: 1, max: 5 }
      stitch: { type: integer, min: 1, max: 5 }
      ablation: { type: integer, min: 1, max: 5 }
  
  top_crack:
    type: string
    enum: [unowned, copies, room, stitch, ablation]
  
  severity_story:
    type: string
    description: "Specific failure with real sentence, wrong output, and who acts"
  
  call:
    type: string
    enum: [ship, ship_with_conditions, hold, reject]
  
  tripwire:
    cadence: string
    owner: string
    threshold: string
```

---

## Integration Points

- **Spec reference:** `specs/scenario-audit.spec.json`
- **Measurements reference:** `specs/measurements.md`
- **Review cycle template:** `runs/review-cycle-01.md`
- **Skill file:** `skills/audit-advisor.skill.md`
