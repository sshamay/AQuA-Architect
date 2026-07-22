# Golden Dataset

A golden dataset is a curated collection of high-quality examples used for evaluating AI system performance.

---

## Purpose

- Establish baseline performance metrics
- Detect quality regressions in new versions
- Compare model and prompt variations
- Provide ground truth for automated evaluation

## Structure

### Dataset Schema

```json
{
  "id": "unique-example-id",
  "category": "classification|generation|retrieval|extraction",
  "difficulty": "easy|medium|hard",
  "input": {
    "query": "User input or prompt",
    "context": "Additional context if applicable",
    "metadata": {}
  },
  "expected_output": {
    "response": "Expected or reference response",
    "quality_score": 0.0,
    "key_elements": ["element1", "element2"],
    "constraints": ["constraint1"]
  },
  "evaluation_criteria": {
    "accuracy_threshold": 0.8,
    "relevance_threshold": 0.7,
    "required_elements": ["element1"]
  },
  "source": "human_created|human_validated|synthetic",
  "created_at": "2026-07-22T12:00:00Z",
  "last_validated": "2026-07-22T12:00:00Z",
  "version": 1
}
```

### Difficulty Levels

| Level   | Criteria                                            |
|---------|-----------------------------------------------------|
| Easy    | Standard inputs, clear expected outputs              |
| Medium  | Ambiguous inputs, partial context, nuanced outputs  |
| Hard    | Edge cases, adversarial inputs, complex reasoning    |

## Dataset Composition

### Recommended Distribution

| Category      | Easy | Medium | Hard | Total |
|---------------|------|--------|------|-------|
| Generation    | 30%  | 40%    | 30%  | 100%  |
| Classification| 40%  | 40%    | 20%  | 100%  |
| Retrieval     | 30%  | 50%    | 20%  | 100%  |
| Extraction    | 35%  | 45%    | 20%  | 100%  |

### Minimum Size Guidelines

| System Complexity | Minimum Examples |
|-------------------|------------------|
| Simple            | 50               |
| Moderate          | 200              |
| Complex           | 500              |
| Mission-critical  | 1000+            |

## Creation Process

### 1. Manual Creation

Human experts create examples based on real-world scenarios.

**Advantages:** High quality, realistic
**Disadvantages:** Time-consuming, expensive

### 2. Production Sampling

Extract examples from production traffic (anonymized).

**Advantages:** Realistic, representative
**Disadvantages:** May include noise, needs validation

### 3. Synthetic Generation

Use LLMs to generate candidate examples, validated by humans.

**Advantages:** Scalable, fast
**Disadvantages:** May lack realism, needs careful validation

### 4. Hybrid Approach

Combine manual curation with production sampling and synthetic augmentation.

**Recommended for most systems.**

## Validation

### Quality Checks

- [ ] All examples have correct expected outputs
- [ ] Difficulty ratings are accurate
- [ ] Evaluation criteria are appropriate
- [ ] No duplicates or near-duplicates
- [ ] Coverage of all critical paths
- [ ] No biased or problematic examples

### Validation Process

1. **Automated Validation** — Schema compliance, format checks
2. **Cross-Validation** — Multiple humans validate each example
3. **Model Validation** — Run current model to verify expected outputs are achievable
4. **Statistical Validation** — Check for distribution anomalies

## Maintenance

### Versioning

- Track dataset versions alongside model versions
- Maintain changelog for dataset modifications
- Archive deprecated datasets

### Refresh Cadence

| Dataset Type        | Refresh Frequency |
|---------------------|-------------------|
| Core benchmark      | Quarterly         |
| Regression test     | Monthly           |
| Edge case library   | As discovered     |
| Production-derived  | Monthly           |

### Quality Monitoring

Track dataset health over time:

- Example validity rate
- Model performance on dataset
- Dataset coverage of production scenarios
- Human agreement on expected outputs
