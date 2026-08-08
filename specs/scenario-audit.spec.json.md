{
  "spec_version": "1.0",
  "product": "Store FAQ bot that picks an answer from the help center",
  "description": "Review routine for auditing whether the FAQ bot matches shopper intent rather than latching onto product names",
  "stakes": "Shoppers get the wrong policy and leave the cart",
  "standard_line": "The answer matches the shopper's real ask — not a nearby FAQ about the same product",

  "change_triggers": [
    "New FAQ entries added to help center",
    "Product catalog update (new product names)",
    "Matching algorithm or embedding model change",
    "Reported mismatch rate exceeds 5% in any 24-hour window",
    "Pre-launch gate before busy sale week"
  ],

  "calendar_floor": {
    "pre_launch": "weekly",
    "post_launch": "monthly",
    "note": "Re-run every week until launch, then monthly once it's live"
  },

  "reviewer_roles": {
    "primary_owner": "CX lead Priya",
    "escalation_contacts": ["Rosalind", "document ops lead"],
    "responsibilities": {
      "CX lead Priya": "Adds and maintains dedicated refund/cancel word watch before any product-name matching runs",
      "Rosalind": "Receives drift alerts when deciding check moves more than one step",
      "document ops lead": "Receives drift alerts alongside Rosalind"
    }
  },

  "sampling_rule": {
    "source": "Store help-desk chat logs",
    "input_profile": "Short mobile questions with product names in the middle",
    "sample_size": "Minimum 20 questions per audit cycle",
    "selection_criteria": "Random sample weighted toward questions containing product names adjacent to policy keywords (refund, return, cancel, exchange)",
    "must_include_patterns": [
      "Questions mentioning product name + return/refund",
      "Questions mentioning product name + cancel/delivery",
      "Questions explicitly stating what they are NOT asking about"
    ]
  },

  "specimen_sentences": [
    "how long do i have to return the Nova Buds after they ship",
    "Nova Buds delivery says Friday — can i still cancel",
    "refund for wrong size on the Trail Jacket, not a shipping question"
  ],

  "checks": {
    "unowned": {
      "description": "Does the bot leave any part of the shopper's actual question unaddressed?",
      "rating": 4,
      "is_top_crack": true,
      "measurement": "mass_across_boundary",
      "threshold": "≤ 0.15 of question tokens should map to zero FAQ content",
      "fail_example": "Shopper asks about refund timing; bot answers only about shipping ETA"
    },
    "copies": {
      "description": "Does the bot duplicate information or give redundant FAQ matches?",
      "rating": 2,
      "measurement": "max_cross_similarity",
      "threshold": "Cosine similarity between top-2 FAQ matches must be < 0.85",
      "fail_example": "Bot returns both 'Shipping Policy' and 'Delivery Times' for a refund question"
    },
    "room": {
      "description": "Does the answer leave room for the shopper's actual intent?",
      "rating": 1,
      "measurement": "per_head_entropy",
      "threshold": "Entropy vs uniform distribution > 0.6 (answer should not over-commit to single FAQ)",
      "fail_example": "Bot locks onto 'Nova Buds' and ignores 'return' entirely"
    },
    "stitch": {
      "description": "Does the bot stitch together the right FAQ sections for multi-part questions?",
      "rating": 2,
      "measurement": "ablation_delta",
      "threshold": "Removing any single matched FAQ should change output by < 30%",
      "fail_example": "Answer collapses when product-name match is removed, even though policy match was correct"
    },
    "ablation": {
      "description": "If we remove the product-name signal, does the bot still find the right policy?",
      "rating": 1,
      "measurement": "ablation_delta_product_name",
      "threshold": "Answer should remain ≥ 70% similar when product name is masked",
      "fail_example": "Masking 'Trail Jacket' causes bot to return generic FAQ instead of refund policy"
    }
  },

  "measurements_reference": "specs/measurements.md",

  "per_check_measurements": {
    "mass_across_boundary": {
      "compute": "Count question tokens that map to zero FAQ content tokens; divide by total question tokens",
      "threshold": "≤ 0.15",
      "flags_when": "Exceeds 0.15 — shopper intent is being ignored"
    },
    "max_cross_similarity": {
      "compute": "L2-normalize flattened per-head maps, stack, matmul against transpose, read off-diagonal max",
      "threshold": "< 0.85",
      "flags_when": "≥ 0.85 — bot is returning near-duplicate FAQ matches"
    },
    "per_head_entropy": {
      "compute": "Calculate entropy of attention distribution per head; compare to uniform baseline",
      "threshold": "> 0.6 ratio to uniform",
      "flags_when": "≤ 0.6 — bot over-commits to single FAQ, ignoring question nuance"
    },
    "ablation_delta": {
      "compute": "Zero one matched FAQ before final answer; measure output change",
      "threshold": "< 0.30 change",
      "flags_when": "≥ 0.30 — answer depends too heavily on single FAQ match"
    },
    "ablation_delta_product_name": {
      "compute": "Mask product name in question; compare answer similarity to unmasked",
      "threshold": "≥ 0.70 similarity",
      "flags_when": "< 0.70 — bot latches onto product name instead of policy intent"
    }
  },

  "attach_or_it_doesnt_count_gate": {
    "required_attachments": [
      "Raw question from chat log with timestamp",
      "FAQ match(es) returned by bot",
      "Per-check measurement values",
      "Screenshot or log excerpt showing the mismatch (if flagged)"
    ],
    "validation": "Audit finding is invalid without all four attachments"
  },

  "ship_call": {
    "decision": "Ship with conditions",
    "condition": "CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against \"refund for wrong size on the Trail Jacket.\"",
    "test_sentence": "refund for wrong size on the Trail Jacket, not a shipping question",
    "owner": "CX lead Priya"
  },

  "tripwire": {
    "schedule": "Re-run Every week until launch, then monthly once it's live",
    "drift_threshold": "If the deciding check drifts more than one step from this ruling",
    "alert_recipients": ["Rosalind", "document ops lead"],
    "deciding_check": "unowned",
    "current_rating": 4
  },

  "cycle_verdict": "Hold my rating — I'll defend it"
}
