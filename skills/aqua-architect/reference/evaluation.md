# Evaluation

Evaluation is the systematic measurement of AI system quality against defined benchmarks and criteria.

---

## Evaluation Framework

### Evaluation Levels

```
System Level     → End-to-end quality of the complete system
Component Level  → Quality of individual components
Prompt Level     → Effectiveness of prompt templates
Model Level      → Base model capabilities and limitations
```

### Evaluation Dimensions

| Dimension      | What It Measures                              | Tools                    |
|----------------|-----------------------------------------------|--------------------------|
| Correctness    | Accuracy of outputs                           | Human eval, benchmarks   |
| Relevance      | Applicability of outputs to the query         | Semantic similarity      |
| Faithfulness   | Factual accuracy of generated content         | Citation checking        |
| Safety         | Absence of harmful content                    | Safety classifiers       |
| Consistency    | Deterministic behavior for same inputs        | Regression tests         |
| Latency        | Speed of response generation                  | Performance monitoring   |
| Cost           | Token usage and compute cost                  | Usage tracking           |

---

## Evaluation Methods

### 1. Automated Metrics

Objective, reproducible measurements.

**For Classification:**
- Accuracy, Precision, Recall, F1
- Confusion matrix analysis
- Per-class performance

**For Generation:**
- BLEU, ROUGE, METEOR (n-gram overlap)
- BERTScore (semantic similarity)
- Perplexity (fluency)

**For Retrieval:**
- Precision@K, Recall@K
- Mean Reciprocal Rank (MRR)
- Normalized Discounted Cumulative Gain (NDCG)

### 2. LLM-as-Judge

Use a strong LLM to evaluate outputs.

```python
judge_prompt = """
Rate the following response on a scale of 1-5:

Query: {query}
Response: {response}

Criteria:
1. Accuracy (1-5)
2. Relevance (1-5)
3. Completeness (1-5)
4. Clarity (1-5)

Provide scores and brief justification.
"""
```

**Best Practices:**
- Use a more capable model than the one being evaluated
- Provide clear rubrics and examples
- Use multiple judges and aggregate scores
- Validate judge scores against human judgment

### 3. Human Evaluation

Expert assessment of output quality.

**Methods:**
- Likert scale rating
- Side-by-side comparison
- Ranking tasks
- Free-form feedback

**When to Use:**
- Establishing ground truth
- Validating automated metrics
- Evaluating subjective quality
- High-stakes applications

### 4. A/B Testing

Compare system variants in production.

**Metrics:**
- User satisfaction scores
- Task completion rates
- Engagement metrics
- Error rates

---

## Evaluation Pipeline

### Pre-Deployment Evaluation

```
Code Change → Unit Tests → Integration Tests → Evaluation Suite → Approval → Deploy
```

### Continuous Evaluation

```
Production Traffic → Sampling → Evaluation → Dashboard → Alerting
```

### Evaluation Cadence

| Trigger                    | Evaluation Scope              |
|----------------------------|-------------------------------|
| Code change                | Component-level tests         |
| Prompt update              | Prompt evaluation suite       |
| Model change               | Full evaluation suite         |
| Weekly                     | Regression detection          |
| Monthly                    | Comprehensive benchmark       |

---

## Golden Dataset Evaluation

See [golden-dataset.md](golden-dataset.md) for dataset creation and maintenance.

### Running Evaluation

```bash
# Run full evaluation
aqua-eval --dataset golden-dataset-v2.json --output results.json

# Run specific category
aqua-eval --dataset golden-dataset-v2.json --category retrieval

# Compare versions
aqua-eval --compare results-v1.json results-v2.json
```

### Interpreting Results

**Score Report:**

```
Evaluation Results
==================
Dataset: golden-dataset-v2.json
Examples: 500

Overall Score: 0.84 (Target: 0.80) ✅

Breakdown:
  Correctness:  0.87 (target: 0.85) ✅
  Relevance:    0.82 (target: 0.80) ✅
  Faithfulness: 0.79 (target: 0.80) ❌
  Safety:       0.95 (target: 0.90) ✅

Latency:
  p50: 1200ms
  p95: 2800ms
  p99: 4500ms

Cost:
  Avg tokens/query: 1500
  Avg cost/query: $0.02
```

---

## Evaluation Anti-Patterns

1. **Metric Gaming** — Optimizing for metrics without improving actual quality
2. **Overfitting to Benchmarks** — Performance on benchmarks doesn't generalize
3. **Cherry-Picking** — Selecting only favorable evaluation results
4. **Ignoring Edge Cases** — Evaluating only on easy examples
5. **No Human Validation** — Relying entirely on automated metrics
6. **Stale Benchmarks** — Using outdated evaluation datasets
7. **Single Metric Focus** — Evaluating only one dimension of quality

---

## Evaluation Checklist

- [ ] Evaluation dataset covers all critical scenarios
- [ ] Multiple evaluation methods are used
- [ ] Results are compared against baselines
- [ ] Human evaluation validates automated metrics
- [ ] Evaluation runs automatically on every change
- [ ] Results are tracked over time
- [ ] Regressions trigger alerts
- [ ] Cost and latency are evaluated alongside quality
