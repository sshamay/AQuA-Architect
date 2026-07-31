# AQuA Architecture

## The Problem

Legacy deterministic testing assumes a single expected output. AI applications are probabilistic: a correct-looking final answer can be produced by a broken internal process — the **"Perfect Outcome, Broken Process" trap**. An LLM can "fluke" a correct response by drawing on general training data, masking silent failures in context retrieval, orchestration, or agentic tool execution.

Because a correct final answer no longer proves a correct internal execution path, traditional black-box testing has become a "dangerous noise machine" that provides false security while ignoring underlying system drift.

## The Solution

**AQuA (Architecting Quality for AI Applications)** organizes the reliability lifecycle into five strategic pillars, anchored by the **Ten Principles of AI Evaluation Engineering**.

We must move beyond checking superficial outputs and start **architecting for evaluation** — from disjointed test suites toward a cohesive Evaluation Architecture.

## The Five Pillars

```
   ┌─────────────────────────────────────────────────┐
   │                                                 │
   ▼                                                 │
 Prevent ──► Detect ──► Govern ──► Observe ──► Learn │
   │          │          │          │          │    │
   └──────────┴──────────┴──────────┴──────────┘    │
                     │                               │
                     └────────────► Golden Dataset ──┘
   (loop: Learn feeds the Golden Dataset,
    which feeds Prevent's CI regression)
```

| Pillar | Role in the Architecture |
|--------|--------------------------|
| **Prevent** | Catching configuration and architectural defects (prompt syntax, tool schemas) before execution |
| **Detect** | The multi-tiered engine used at runtime to measure quality across structural, conceptual, and qualitative dimensions |
| **Govern** | Aggregating evaluation evidence into a business-oriented score to decide if a result is safe for release or requires manual review |
| **Observe** | Instrumenting the entire pipeline — retrieval, orchestration, and tools — to perform root cause analysis |
| **Learn** | Closing the loop by automatically converting production failures into new regression tests |

## The Ten Principles

| Pillar | Principle | Purpose |
|--------|-----------|---------|
| Prevent | P1. Shift-Left Validation | Catch configuration and architectural defects before execution |
| Prevent | P2. Golden Datasets | Maintain a version-controlled regression suite representing business scenarios and edge cases |
| Detect | P3. Progressive Evaluation | Orchestrate evaluators from cheapest to most expensive to optimize latency and token cost |
| Detect | P4. Deterministic First | Prioritize explainable rule-based validation (JSON Schema, regex) for structural contracts |
| Detect | P5. Semantic Evaluation | Use embeddings to measure conceptual meaning rather than exact wording |
| Detect | P6. LLM-as-a-Judge | Use secondary models to evaluate qualitative dimensions like groundedness and hallucination |
| Govern | P7. Risk-Based Confidence | Aggregate all evaluation evidence into a single business-oriented score |
| Govern | P8. Human-in-the-Loop | Escalate low-confidence or high-risk evaluations to humans |
| Observe | P9. Deep Observability | Instrument the entire pipeline to trace retrieval, orchestration, and tool execution |
| Learn | P10. Continuous Feedback Loop | Automatically convert production failures into new test cases |

## Why This Architecture

The pillars represent the strategic lifecycle (the "what" and "when"); the principles provide the operational rules (the "how"). This dual-layered approach validates not just the final output but the entire internal execution path, effectively solving the "Perfect Outcome, Broken Process" trap.

The framework is recursive: Prevent ensures structural soundness → Detect validates process and outcome → Govern translates signals into release decisions → Observe provides traceability → Learn converts production experience into new regression assets → which are validated again by Prevent in the next CI/CD run.

## Key Functional Relationships

- **Preventive Foundation (P1, P2):** Before an agent runs, Shift-Left Validation blocks builds with malformed schemas and Golden Datasets anchor regression.
- **Orchestrated Detection (P3-P6):** P4, P5, P6 define evaluator types; P3 dictates they run cheapest-first to minimize latency and cost.
- **Decision and Oversight (P7, P8):** P7 combines evaluator evidence into a confidence signal; P8 triggers only when the signal misses the business threshold.
- **Traceability for Root Cause (P9):** Deep Observability traces the *how* — verifying a correct answer wasn't a fluke caused by a broken internal process.
- **The Learning Engine (P10):** Takes the output of Observe and Govern and feeds it back into Prevent by evolving the Golden Dataset.

## The Question Shift

| Traditional | AQuA |
|-------------|------|
| "Did the model generate a good answer?" | "Did the application execute the correct workflow, use the correct data, make the correct business decision, and provide sufficient evidence to prove it?" |

As AI applications become increasingly autonomous, quality engineering must evolve alongside them. The future belongs to architectures that are not only intelligent, but also **observable, measurable, governable, and continuously improving**.
