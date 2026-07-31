# Pillar 5 — Learn

## Concept

Quality in AI systems is not a static destination but a continuously evolving process. Learn is the continuous improvement engine of the AQuA framework. Rather than treating production failures as isolated incidents, every validated failure becomes an opportunity to strengthen future releases.

Learn closes the quality lifecycle by transforming production anomalies, evaluation failures, and HITL corrections into version-controlled regression assets. The expanded Golden Dataset is automatically verified during future CI/CD executions as prompts, orchestration logic, retrieval strategies, or foundation models evolve.

**Principle:** P10 Continuous Feedback Loop

## P10 — Continuous Feedback Loop

Continuously improve the AI application by converting validated production knowledge into version-controlled regression scenarios. This closes the loop between Observe and Prevent, ensuring every release benefits from lessons learned in production.

### Implementation

```python
def learning_layer_feedback_loop(production_anomaly, human_correction=None):
    # 1. Capture the production failure
    candidate_test_case = {
        "input_query": production_anomaly.query,
        "retrieved_context": production_anomaly.context,
        "failure_reason": production_anomaly.detection_signal,
        "application_version": "v1.4.2"
    }

    # 2. Attach verified ground truth when available
    if human_correction:
        candidate_test_case["expected_output"] = human_correction.text
        candidate_test_case["status"] = "READY_FOR_REVIEW"
    else:
        candidate_test_case["status"] = "PENDING_LABELING"

    # 3. Submit the candidate for review before promotion
    submit_candidate_test_case(candidate_test_case)
    return LearningStatus.submitted()
```

## Candidate Sources

Collect candidate regression scenarios from:

- Production failure analysis — identified through production telemetry, evaluation failures, low-confidence responses, or anomalous execution behavior
- HITL corrections — expert-reviewed responses become verified ground truth
- Edge case discovery — evaluator disagreements, repeated execution variance, prompt/orchestration changes, model upgrades, low-confidence predictions
- Regression evolution — previously resolved failures re-verified during changes

## Promotion Flow

```
Production Failure
    ↓
Candidate Regression Test
    ↓
Human Review (domain experts / trusted business knowledge)
    ↓
Golden Dataset (version-controlled)
    ↓
CI Regression
```

## Human Review & Ground Truth

Before becoming part of the Golden Dataset, candidate regression scenarios should be reviewed and validated by domain experts or trusted business knowledge to establish expected behavior for future evaluations.

**Never recommend automatically adding production failures directly into the Golden Dataset. Always recommend review before promotion.**

## Golden Dataset Evolution

Once approved, the regression scenario is versioned and incorporated into the Golden Dataset. The expanded regression suite executes automatically during future CI/CD pipelines, ensuring previously resolved failures remain fixed across prompt updates, orchestration changes, retrieval modifications, and model upgrades.

With the updated Golden Dataset committed to source control, the next CI/CD execution begins again at the Prevent pillar — completing the continuous quality lifecycle.

## Questions to Ask

- How will production failures improve future releases?
- What is the candidate regression test collection process?
- Who reviews candidates before promotion to the Golden Dataset?
- Is the expanded Golden Dataset executed in CI/CD?
- Are HITL corrections captured as future ground truth?
