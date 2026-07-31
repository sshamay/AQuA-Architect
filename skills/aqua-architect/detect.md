# Pillar 2 — Detect

## Concept

In a probabilistic system, a correct-looking response does not necessarily indicate correct system behavior. An LLM can "fluke" a correct answer through general training data even when underlying retrieval or tool-calling logic fails. Detect solves the **"Perfect Outcome, Broken Process" trap** with a multi-tiered evaluation strategy that validates the internal execution path alongside the final result.

**Principles:** P3 Progressive Evaluation, P4 Deterministic First, P5 Semantic Evaluation, P6 LLM-as-a-Judge

## P3 — Progressive Evaluation

Orchestrate evaluators from cheapest to most expensive, escalating only when additional confidence is required. Implement a fail-fast pipeline: a response must pass a zero-cost structural check before being sent to an expensive LLM-as-a-Judge. This is the primary driver for reducing latency and token costs in production evaluation.

## P4 — Deterministic First

Prioritize explainable, rule-based validation for structural contracts and business logic:

- **Structural Contract Compliance** — Verify the model reliably outputs the strict JSON or regex-compliant formats required by downstream code
- **Tool-Call Verification** — Verify the selected tool, parameter values, execution order, and returned status before evaluating the final response

## P5 — Semantic Evaluation

Use embeddings to measure conceptual meaning and proximity rather than exact wording. Compare a generated answer against the Golden Anchor reference using cosine similarity. This is a supporting signal (not a correctness guarantee) when a deterministic "exact match" is too pedantic for natural language.

## P6 — LLM-as-a-Judge

Use a secondary, highly capable model to audit qualitative dimensions against a structured rubric:

- Groundedness — claims supported by retrieved evidence
- Hallucination detection — plausible but factually incorrect statements
- Completeness, Relevance, Instruction Following, Reasoning Quality

### Implementation

```python
def detect_layer_orchestration(response_payload, reference_data):
    # 1. Deterministic First (P4): zero-cost structural check
    if not is_valid_json_contract(response_payload, reference_data.schema):
        return EvaluationResult.fail("Structural Contract Violation")

    # 2. Semantic Evaluation (P5): conceptual proximity
    similarity = measure_semantic_similarity(response_payload.text, reference_data.golden_answer)
    if similarity < 0.85:
        # 3. Progressive Evaluation (P3): escalate to judge
        return run_qualitative_audit(response_payload, reference_data)

    return EvaluationResult.pass_with_confidence(similarity)


def run_qualitative_audit(payload, reference):
    judge_critique = llm_judge.evaluate(
        content=payload.text,
        context=payload.retrieved_context,
        rubric="groundedness_and_completeness"
    )
    return judge_critique
```

- `is_valid_json_contract()` — JSON Schema, Pydantic, or custom business rule validators
- `measure_semantic_similarity()` — embedding models (OpenAI Embeddings, Sentence Transformers) + cosine similarity
- `run_qualitative_audit()` — secondary LLM with structured evaluation rubric

## Evaluation Layers (cheapest first)

1. Deterministic — JSON Schema, regex, business rules, output format, tool parameters, retrieval validation
2. Semantic — embedding similarity against the Golden Anchor
3. LLM-as-a-Judge — groundedness, completeness, hallucination, instruction following, answer quality

## Progressive Evaluation Rule

Always fail-fast. Example: if JSON is invalid → stop. Do not execute the LLM Judge.

## Questions to Ask

- Is there structural contract validation?
- Is tool-call verification implemented (tool, params, order, status)?
- Is semantic similarity used against a reference answer?
- Is an LLM judge used only after cheaper layers fail?
- Is the evaluation pipeline ordered cheapest-first?
