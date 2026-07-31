# Pillar 3 — Govern

## Concept

AI quality is rarely binary. Governance aggregates multiple evaluation signals into a measurable confidence score that supports risk-based release decisions.

**Principles:** P7 Risk-Based Confidence, P8 Human-in-the-Loop

## P7 — Risk-Based Confidence

Set configuration-driven, business-oriented thresholds (e.g., 98% for healthcare vs. 85% for an HR bot). The confidence score is a weighted aggregation of the evaluation signals produced by each validation layer.

### Confidence Score Calculation

```python
Confidence =
    (Deterministic Validation × weight)
  + (Semantic Evaluation × weight)
  + (LLM-as-a-Judge × weight)
```

Example:

| Signal | Score | Weight |
|--------|-------|--------|
| Deterministic Validation | 1.00 | 40% |
| Semantic Evaluation | 0.85 | 20% |
| LLM-as-a-Judge | 1.00 | 40% |

```
Confidence = (1.00 × 0.40) + (0.85 × 0.20) + (1.00 × 0.40) = 0.97 (97%)
```

The weighting strategy is intentionally configurable and should reflect the organization's risk tolerance. Regulated domains (healthcare, finance) may weight deterministic validation and groundedness higher; knowledge-intensive applications may emphasize semantic similarity. Calibrate weights using historical evaluation results, production telemetry, and expert feedback.

### Dynamic Risk Thresholds

| Use Case | Required Confidence |
|----------|---------------------|
| Healthcare | 98% |
| Financial | 95% |
| HR Assistant | 85% |
| Internal Tool | 80% |

## P8 — Human-in-the-Loop

Automatically route low-confidence or high-risk cases to human experts for review.

**Conflicting Evaluator Resolution** — If a deterministic check passes but the LLM Judge raises a hallucination flag, the system identifies the conflict and triggers HITL review.

### Implementation

```python
def governance_gate(evaluation_signals, business_risk_threshold):
    aggregate_score = calculate_confidence_score(evaluation_signals)

    if aggregate_score >= business_risk_threshold:
        return "RELEASE: Confidence satisfies business risk."

    return trigger_hitl_workflow(aggregate_score, evaluation_signals)
```

- `calculate_confidence_score()` — aggregate weighted evaluation signals into a normalized confidence score; configurable and calibrated
- `trigger_hitl_workflow()` — route low-confidence or conflicting evaluations to a human reviewer with supporting evidence: retrieved context, evaluator scores, tool execution traces, judge rationale

## Release Decision

- Score ≥ threshold → release
- Score < threshold → HITL, retry, escalation, or reject (ask the user which)

## Questions to Ask

- What is the business risk? (healthcare 98%, financial 95%, HR 85%, internal tool 80%)
- What are the weights for each evaluation signal?
- What should happen when confidence is below threshold? (HITL, retry, escalation, reject)
- How are conflicting evaluator results resolved?
